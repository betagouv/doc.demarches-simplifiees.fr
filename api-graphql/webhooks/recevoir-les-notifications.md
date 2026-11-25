---
description: Format des notifications, vérification de la signature et garanties de livraison
---

# Recevoir les notifications

## Format

Chaque notification est une requête `POST` au format JSON. Elle contient un **lot** de 1 à 100 évènements, dans l'ordre où ils se sont produits.

```http
POST /webhooks/demarche-numerique HTTP/1.1
Content-Type: application/json
webhook-id: msg_12_987650_987655
webhook-timestamp: 1795511807
webhook-signature: v1,K5oZfzN95Z9UVu1EsfQmfVNQhnkZ2pj9o9NDN/H/pI4=

{
  "webhook_id": "V2ViaG9vay0xMg==",
  "demarche_number": 12345,
  "events": [
    {
      "sequence": 987650,
      "type": "dossier_depose",
      "dossier_number": 4567890,
      "timestamp": "2026-11-24T10:15:02+01:00"
    },
    {
      "sequence": 987655,
      "type": "dossier_accepte",
      "dossier_number": 4567812,
      "timestamp": "2026-11-24T10:16:40+01:00"
    }
  ]
}
```

### En-têtes

| En-tête             | Contenu                                                                                                  |
| ------------------- | -------------------------------------------------------------------------------------------------------- |
| `webhook-id`        | Identifiant de la livraison. Il reste le même quand un lot est renvoyé.                                  |
| `webhook-timestamp` | Date d'envoi, en secondes depuis le 1er janvier 1970 (timestamp Unix).                                   |
| `webhook-signature` | Signature de la requête. Pendant un [renouvellement du secret](gerer-ses-webhooks.md#renouveler-le-secret), elle en contient deux, séparées par une espace. |

### Corps

| Champ                     | Description                                                                                      |
| ------------------------- | ------------------------------------------------------------------------------------------------ |
| `webhook_id`              | Identifiant du webhook, celui renvoyé par `webhookCreer`.                                        |
| `demarche_number`         | Numéro de la démarche.                                                                           |
| `events[].sequence`       | Numéro d'ordre de l'évènement : toujours croissant, mais pas consécutif. Sert à écarter les doublons. |
| `events[].type`           | Type d'évènement (voir [la liste](README.md#evenements)).                                        |
| `events[].dossier_number` | Numéro du dossier concerné.                                                                      |
| `events[].timestamp`      | Date de l'évènement, au format ISO 8601.                                                         |

## Vérifier la signature <a href="#verifier-la-signature" id="verifier-la-signature"></a>

La vérification est obligatoire : n'importe qui peut envoyer une requête à votre URL, et seule la signature prouve que la notification vient bien de **demarche.numerique.gouv.fr** et non d'un tiers qui se ferait passer pour nous. Rejetez toute requête dont la signature est invalide.

Les notifications suivent la spécification ouverte [Standard Webhooks](https://www.standardwebhooks.com). Ses [bibliothèques officielles](https://github.com/standard-webhooks/standard-webhooks/tree/main/libraries) (JavaScript, Python, PHP, Ruby, Go, Java, C#, Rust, Elixir) vérifient la signature et rejettent les requêtes de plus de 5 minutes, ce qui empêche de rejouer une ancienne notification. Elles acceptent le secret tel que vous l'avez reçu, préfixe `whsec_` compris.

{% hint style="warning" %}
Vérifiez la signature sur le **corps brut** de la requête, octet pour octet. Un JSON décodé puis réencodé par votre framework ne correspond plus à la signature.
{% endhint %}

{% tabs %}
{% tab title="Node.js" %}
```javascript
// npm install express standardwebhooks
import express from "express";
import { Webhook } from "standardwebhooks";

const webhook = new Webhook(process.env.WEBHOOK_SECRET);
const app = express();

app.post(
  "/webhooks/demarche-numerique",
  express.raw({ type: "application/json" }), // garde le corps brut
  (req, res) => {
    let payload;
    try {
      payload = webhook.verify(req.body, req.headers);
    } catch {
      return res.sendStatus(401);
    }

    // … traiter payload.events
    res.sendStatus(204);
  },
);

app.listen(3000);
```
{% endtab %}

{% tab title="Python" %}
```python
# pip install flask standardwebhooks
import os

from flask import Flask, request
from standardwebhooks.webhooks import Webhook, WebhookVerificationError

webhook = Webhook(os.environ["WEBHOOK_SECRET"])
app = Flask(__name__)


@app.post("/webhooks/demarche-numerique")
def recevoir():
    try:
        payload = webhook.verify(request.get_data(), request.headers)
    except WebhookVerificationError:
        return "", 401

    # … traiter payload["events"]
    return "", 204
```
{% endtab %}

{% tab title="Ruby (Rails)" %}
```ruby
# Gemfile : gem "standardwebhooks"
class DemarcheNumeriqueWebhooksController < ActionController::API
  WEBHOOK = StandardWebhooks::Webhook.new(ENV.fetch("WEBHOOK_SECRET"))

  def create
    payload = WEBHOOK.verify(request.raw_post, request.headers)

    # … traiter payload["events"]
    head :no_content
  rescue StandardWebhooks::WebhookVerificationError
    head :unauthorized
  end
end
```
{% endtab %}
{% endtabs %}

<details>

<summary>Vérifier la signature sans bibliothèque</summary>

1. Construisez le contenu signé : `{webhook-id}.{webhook-timestamp}.{corps brut}`.
2. Retirez le préfixe `whsec_` du secret et décodez le reste en base64 : c'est la clé.
3. Calculez `v1,` suivi de `base64(HMAC-SHA256(clé, contenu signé))`.
4. Acceptez la requête si ce résultat est égal à l'une des signatures de l'en-tête `webhook-signature` (séparées par des espaces), avec une comparaison en temps constant, et si `webhook-timestamp` date de moins de 5 minutes.

</details>

## Répondre

* Répondez par un code **`2xx`** dans les **15 secondes**, avec un corps vide ou court : il n'est pas lu.
* Tout autre code, un délai dépassé ou une erreur réseau compte comme un échec. Les redirections (`3xx`) ne sont pas suivies.
* Si votre traitement est long, enregistrez d'abord les évènements (base de données, file de messages), répondez aussitôt, puis traitez-les en arrière-plan.

Deux réponses particulières sont prises en compte :

* `Retry-After` (en secondes, ou une date HTTP) repousse la prochaine tentative, de 24 heures au plus. Il ne peut pas l'avancer.
* `410 Gone` désactive le webhook immédiatement. Réservez-la au cas où vous ne voulez plus rien recevoir.

## Garanties de livraison

### Ordre

Les évènements d'un webhook sont livrés dans l'ordre, un lot à la fois : le lot suivant ne part qu'une fois le précédent accepté.

### Doublons <a href="#doublons" id="doublons"></a>

Un même lot peut vous parvenir plusieurs fois, par exemple si votre réponse s'est perdue. Ignorez les évènements dont la `sequence` a déjà été traitée : l'ordre étant garanti, il suffit de mémoriser la plus grande `sequence` traitée.

### Nouvelles tentatives

En cas d'échec, le même lot est renvoyé sous le même `webhook-id`, au bout de 5 secondes, puis 5 minutes, 30 minutes, 2, 5, 10, 14, 20 et 24 heures (à 10 % près). Les évènements suivants attendent leur tour.

### Désactivation automatique <a href="#desactivation-automatique" id="desactivation-automatique"></a>

Après 10 échecs consécutifs, soit environ 3 jours, ou dès une réponse `410 Gone`, le webhook est désactivé et les administrateurs de la démarche sont prévenus par email. Le champ `lastError` du webhook indique la dernière erreur rencontrée.

Une fois le problème corrigé, [réactivez le webhook](gerer-ses-webhooks.md#desactiver-et-reactiver) : les évènements en attente sont livrés dans l'ordre.

### Conservation

Les évènements sont conservés 7 jours. Au-delà, ceux qui n'ont pas pu être livrés sont perdus : resynchronisez alors les dossiers modifiés depuis la dernière notification reçue, en [paginant avec `updatedSince`](../cas-dusages-exemple-dimplementation/pagination-synchroniser-une-demarche-a-forte-volumetrie-synchronisation.md).

## Bonnes pratiques

* **Relisez le dossier via l'API.** La notification signale un changement ; la query [`dossier`](../les-queries/getdossier.md) donne l'état actuel, qui a pu évoluer depuis.
* **Ne lisez chaque dossier qu'une fois par lot.** Plusieurs évènements d'un même lot peuvent concerner le même dossier.
* **Prévoyez les dossiers supprimés.** Un dossier supprimé n'est plus lisible via l'API (erreur `not_found`) : traitez `dossier_supprime` sans appeler l'API, et attendez-vous à ce `not_found` pour un dossier supprimé entre l'évènement et votre lecture.
* **Autorisez l'adresse IP de votre serveur.** Le serveur qui reçoit les notifications et appelle l'API doit avoir une [adresse IP autorisée](../jeton-dauthentification/README.md) par votre jeton.
* **Protégez le secret.** Stockez-le comme un mot de passe, hors du code source. En cas de fuite, [renouvelez-le](gerer-ses-webhooks.md#renouveler-le-secret) sans période de transition.
