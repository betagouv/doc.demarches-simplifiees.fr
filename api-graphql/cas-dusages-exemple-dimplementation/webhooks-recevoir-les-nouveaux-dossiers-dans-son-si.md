---
description: >-
  Exemple de cas d'usage : recevoir chaque dossier dans votre SI quelques
  secondes après son dépôt, sans interroger l'API en boucle.
---

# Webhooks – Recevoir les nouveaux dossiers dans son SI

Objectif : à chaque dépôt ou modification d'un dossier, votre SI récupère sa version à jour. L'exemple est donné en Node.js, Python et Ruby (Rails) ; le principe est le même dans tout autre langage.

{% hint style="info" %}
Cet exemple part d'un webhook déjà créé (voir le [démarrage rapide](../webhooks/README.md#demarrage-rapide)), abonné aux évènements `dossier_depose` et `dossier_modifie`. L'application lit son secret dans la variable d'environnement `WEBHOOK_SECRET`, et votre jeton d'API dans `API_TOKEN`.
{% endhint %}

## 1. Recevoir les notifications et lire les dossiers

Pour chaque lot reçu, l'application :

1. [vérifie la signature](../webhooks/recevoir-les-notifications.md#verifier-la-signature) ;
2. écarte les [évènements déjà traités](../webhooks/recevoir-les-notifications.md#doublons) ;
3. lit une seule fois chaque dossier concerné avec la query `dossier` de l'API GraphQL, puis l'enregistre dans le SI ;
4. répond `204`, ou `500` en cas d'erreur pour que le lot soit renvoyé plus tard.

{% tabs %}
{% tab title="Node.js" %}
{% code title="serveur.mjs" %}
```javascript
// npm install express standardwebhooks
import express from "express";
import { Webhook } from "standardwebhooks";

const API_URL = "https://demarche.numerique.gouv.fr/api/v2/graphql";
const webhook = new Webhook(process.env.WEBHOOK_SECRET);

const DOSSIER_QUERY = `
  query getDossier($number: Int!) {
    dossier(number: $number) {
      number
      state
      dateDepot
      dateDerniereModification
      usager { email }
      champs { id label stringValue }
    }
  }
`;

// Renvoie null si le dossier a été supprimé depuis l'évènement
async function lireDossier(number) {
  const response = await fetch(API_URL, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${process.env.API_TOKEN}`,
    },
    body: JSON.stringify({ query: DOSSIER_QUERY, variables: { number } }),
  });
  const { data, errors } = await response.json();

  if (errors?.some((error) => error.extensions?.code === "not_found")) return null;
  if (errors) throw new Error(errors[0].message);
  return data.dossier;
}

// À remplacer par l'enregistrement dans votre SI (base de données, gestion documentaire…)
async function enregistrerDossier(dossier) {
  console.log(`Dossier ${dossier.number} : ${dossier.state}`);
}

// Dernière sequence traitée, par webhook. À conserver dans votre SI : en mémoire, elle est perdue au redémarrage
const dernieresSequences = new Map();

const app = express();

app.post(
  "/webhooks/demarche-numerique",
  express.raw({ type: "application/json" }), // la signature porte sur le corps brut
  async (req, res) => {
    let payload;
    try {
      payload = webhook.verify(req.body, req.headers);
    } catch {
      return res.sendStatus(401);
    }

    const derniereSequence = dernieresSequences.get(payload.webhook_id) ?? 0;
    const events = payload.events.filter((event) => event.sequence > derniereSequence);
    const numeros = new Set(events.map((event) => event.dossier_number));

    try {
      for (const numero of numeros) {
        const dossier = await lireDossier(numero);
        if (dossier) await enregistrerDossier(dossier);
      }
    } catch (error) {
      console.error(error);
      return res.sendStatus(500);
    }

    if (events.length > 0) dernieresSequences.set(payload.webhook_id, events.at(-1).sequence);
    res.sendStatus(204);
  },
);

app.listen(3000);
```
{% endcode %}
{% endtab %}

{% tab title="Python" %}
{% code title="serveur.py" %}
```python
# pip install flask requests standardwebhooks
import os

import requests
from flask import Flask, request
from standardwebhooks.webhooks import Webhook, WebhookVerificationError

API_URL = "https://demarche.numerique.gouv.fr/api/v2/graphql"
webhook = Webhook(os.environ["WEBHOOK_SECRET"])
app = Flask(__name__)

DOSSIER_QUERY = """
  query getDossier($number: Int!) {
    dossier(number: $number) {
      number
      state
      dateDepot
      dateDerniereModification
      usager { email }
      champs { id label stringValue }
    }
  }
"""


# Renvoie None si le dossier a été supprimé depuis l'évènement
def lire_dossier(number):
    response = requests.post(
        API_URL,
        json={"query": DOSSIER_QUERY, "variables": {"number": number}},
        headers={"Authorization": f"Bearer {os.environ['API_TOKEN']}"},
        timeout=10,
    )
    body = response.json()
    errors = body.get("errors") or []

    if any(error.get("extensions", {}).get("code") == "not_found" for error in errors):
        return None
    if errors:
        raise RuntimeError(errors[0]["message"])
    return body["data"]["dossier"]


# À remplacer par l'enregistrement dans votre SI (base de données, gestion documentaire…)
def enregistrer_dossier(dossier):
    print(f"Dossier {dossier['number']} : {dossier['state']}")


# Dernière sequence traitée, par webhook. À conserver dans votre SI : en mémoire, elle est perdue au redémarrage
dernieres_sequences = {}


@app.post("/webhooks/demarche-numerique")
def recevoir():
    try:
        payload = webhook.verify(request.get_data(), request.headers)
    except WebhookVerificationError:
        return "", 401

    derniere_sequence = dernieres_sequences.get(payload["webhook_id"], 0)
    events = [event for event in payload["events"] if event["sequence"] > derniere_sequence]
    numeros = dict.fromkeys(event["dossier_number"] for event in events)

    try:
        for numero in numeros:
            dossier = lire_dossier(numero)
            if dossier:
                enregistrer_dossier(dossier)
    except Exception:
        app.logger.exception("Lot non traité")
        return "", 500

    if events:
        dernieres_sequences[payload["webhook_id"]] = events[-1]["sequence"]
    return "", 204
```
{% endcode %}
{% endtab %}

{% tab title="Ruby (Rails)" %}
{% code title="app/controllers/demarche_numerique_webhooks_controller.rb" %}
```ruby
# Gemfile : gem "standardwebhooks"
# config/routes.rb : post "webhooks/demarche-numerique", to: "demarche_numerique_webhooks#create"
# Dernière sequence traitée, par webhook :
#   bin/rails generate model CurseurWebhook webhook_id:string:uniq derniere_sequence:bigint
class DemarcheNumeriqueWebhooksController < ActionController::API
  API_URL = URI("https://demarche.numerique.gouv.fr/api/v2/graphql")
  WEBHOOK = StandardWebhooks::Webhook.new(ENV.fetch("WEBHOOK_SECRET"))

  DOSSIER_QUERY = <<~GRAPHQL
    query getDossier($number: Int!) {
      dossier(number: $number) {
        number
        state
        dateDepot
        dateDerniereModification
        usager { email }
        champs { id label stringValue }
      }
    }
  GRAPHQL

  # Une exception non rattrapée répond 500 : le lot sera renvoyé plus tard
  def create
    payload = WEBHOOK.verify(request.raw_post, request.headers)

    curseur = CurseurWebhook.create_with(derniere_sequence: 0)
      .find_or_create_by!(webhook_id: payload["webhook_id"])
    events = payload["events"].select { |event| event["sequence"] > curseur.derniere_sequence }

    events.map { |event| event["dossier_number"] }.uniq.each do |numero|
      dossier = lire_dossier(numero)
      enregistrer_dossier(dossier) if dossier
    end

    curseur.update!(derniere_sequence: events.last["sequence"]) if events.any?
    head :no_content
  rescue StandardWebhooks::WebhookVerificationError
    head :unauthorized
  end

  private

  # Renvoie nil si le dossier a été supprimé depuis l'évènement
  def lire_dossier(numero)
    response = Net::HTTP.post(
      API_URL,
      { query: DOSSIER_QUERY, variables: { number: numero } }.to_json,
      "Content-Type" => "application/json",
      "Authorization" => "Bearer #{ENV.fetch("API_TOKEN")}"
    )
    body = JSON.parse(response.body)
    errors = body["errors"] || []

    return nil if errors.any? { |error| error.dig("extensions", "code") == "not_found" }
    raise errors.first["message"] if errors.any?

    body["data"]["dossier"]
  end

  # À remplacer par l'enregistrement dans votre SI (base de données, gestion documentaire…)
  def enregistrer_dossier(dossier)
    Rails.logger.info("Dossier #{dossier["number"]} : #{dossier["state"]}")
  end
end
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
Le traitement doit tenir en 15 secondes. Cet exemple lit les dossiers avant de répondre, ce qui convient tant que les lots restent petits. Or un lot peut contenir jusqu'à 100 évènements, par exemple après une interruption de votre application. En production, enregistrez les évènements dans une file, répondez aussitôt, puis lisez les dossiers en arrière-plan.
{% endhint %}

Dans votre SI, enregistrez le dossier en remplaçant la copie qui porte le même `number`, ou en la créant si elle n'existe pas encore : un lot reçu deux fois ne crée alors aucun doublon.

L'exemple répond `500` sur toute erreur de l'API, ce qui fait renvoyer le lot. Une erreur qui se reproduirait à chaque tentative bloquerait tous les lots suivants : voir les [bonnes pratiques](../webhooks/recevoir-les-notifications.md#bonnes-pratiques).

## 2. Importer les dossiers existants

Le webhook ne notifie que les évènements postérieurs à sa création. Pour reprendre les dossiers déjà déposés, parcourez une fois la démarche en suivant l'exemple [Récupérer tous les dossiers d'une démarche](pagination-recuperer-tous-les-dossiers-dune-demarche.md).

## Aller plus loin

* **Suivre aussi les décisions** : ajoutez `dossier_accepte`, `dossier_refuse`, `dossier_sans_suite` et `dossier_repasse_en_instruction` au webhook ([modifier un webhook](../webhooks/gerer-ses-webhooks.md#modifier-un-webhook)). Le code ne change pas : il relit le dossier, dont l'état (`state`) reflète la décision.
* **Répercuter les suppressions** : abonnez le webhook à `dossier_supprime`. Un dossier supprimé n'est plus lisible via l'API : pour ces évènements, supprimez ou archivez votre copie sans appeler l'API, selon vos propres règles de conservation.
