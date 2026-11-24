---
description: >-
  Exemple de cas d'usage : recevoir chaque dossier dans votre SI quelques
  secondes après son dépôt, sans interroger l'API en boucle.
---

# Webhooks – Recevoir les nouveaux dossiers dans son SI

Objectif : à chaque dépôt ou modification d'un dossier, votre SI récupère sa version à jour. L'exemple utilise Node.js (version 18 ou plus) ; le principe est le même dans tout autre langage.

{% hint style="info" %}
Commencez par lire la présentation des [webhooks](../webhooks/README.md).
{% endhint %}

## 1. Créer le webhook

```shell
curl 'https://demarche.numerique.gouv.fr/api/v2/graphql' \
  --header 'Authorization: Bearer <votre_token>' \
  --header 'Content-Type: application/json' \
  --data '{
    "query": "mutation($input: WebhookCreerInput!) { webhookCreer(input: $input) { webhook { id } secret errors { message } } }",
    "variables": {
      "input": {
        "demarche": { "number": 12345 },
        "url": "https://mon-si.exemple.fr/webhooks/demarche-numerique",
        "eventTypes": ["dossier_depose", "dossier_modifie"],
        "label": "Import SI"
      }
    }
  }'
```

Conservez le `secret` de la réponse : l'application le lit dans la variable d'environnement `WEBHOOK_SECRET`.

## 2. Recevoir les notifications et lire les dossiers

Pour chaque lot reçu, l'application :

1. vérifie la signature ;
2. écarte les évènements déjà traités ;
3. lit une seule fois chaque dossier concerné, puis l'enregistre dans le SI ;
4. répond `204`, ou `500` en cas d'erreur pour que le lot soit renvoyé plus tard.

```shell
npm install express standardwebhooks
WEBHOOK_SECRET="whsec_…" API_TOKEN="votre_token" node serveur.mjs
```

{% code title="serveur.mjs" %}
```javascript
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

// À remplacer par l'enregistrement dans votre SI (base de données, GED…)
async function enregistrerDossier(dossier) {
  console.log(`Dossier ${dossier.number} : ${dossier.state}`);
}

// À conserver dans votre SI : en mémoire, elle est perdue au redémarrage
let derniereSequence = 0;

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

    if (events.length > 0) derniereSequence = events.at(-1).sequence;
    res.sendStatus(204);
  },
);

app.listen(3000);
```
{% endcode %}

{% hint style="warning" %}
Le traitement doit tenir en 15 secondes. Cet exemple lit les dossiers avant de répondre, ce qui convient tant que les lots restent petits. Pour des volumes importants, enregistrez les évènements dans une file, répondez aussitôt, puis lisez les dossiers en arrière-plan.
{% endhint %}

Relire un dossier et remplacer sa copie donne le même résultat quel que soit le nombre de fois où l'opération est répétée : un lot reçu deux fois ne crée pas de doublon dans votre SI.

## 3. Importer les dossiers existants

Le webhook ne notifie que les évènements postérieurs à sa création. Pour reprendre les dossiers déjà déposés, parcourez une fois la démarche en suivant l'exemple [Récupérer tous les dossiers d'une démarche](pagination-recuperer-tous-les-dossiers-dune-demarche.md).

## Variantes

### Suivre aussi les décisions

Ajoutez les évènements de décision au webhook. Le code ne change pas : il relit le dossier, dont l'état (`state`) reflète la décision.

```graphql
mutation {
  webhookModifier(input: {
    webhook: "<id du webhook>"
    eventTypes: [dossier_depose, dossier_modifie, dossier_accepte, dossier_refuse, dossier_sans_suite, dossier_repasse_en_instruction]
  }) {
    webhook { eventTypes }
    errors { message }
  }
}
```

### Répercuter les suppressions

Abonnez le webhook à `dossier_supprime` : un dossier supprimé n'est plus lisible via l'API. Pour ces évènements, supprimez ou archivez votre copie sans appeler l'API, selon vos propres règles de conservation.
