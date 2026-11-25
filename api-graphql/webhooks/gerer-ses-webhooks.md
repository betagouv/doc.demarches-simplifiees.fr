---
description: Lister, modifier, désactiver, renouveler le secret ou supprimer un webhook via l'API
---

# Gérer ses webhooks

Les webhooks se gèrent via l'API GraphQL. Les mutations demandent un jeton avec un accès **en écriture** à la démarche ; un jeton en lecture suffit pour les consulter.

Un webhook est désigné par son `id`, renvoyé à sa création (par exemple `V2ViaG9vay0xMg==`).

## Lister les webhooks d'une démarche

```graphql
query {
  webhooks(demarche: { number: 12345 }) {
    id
    label
    url
    eventTypes
    enabled
    lastSuccessAt
    lastError
    autoDisabledAt
  }
}
```

Pour un seul webhook, utilisez `webhook(id: "V2ViaG9vay0xMg==")`, qui renvoie les mêmes champs.

| Champ                     | Description                                                                         |
| ------------------------- | ----------------------------------------------------------------------------------- |
| `enabled`                 | Le webhook est actif.                                                               |
| `lastSuccessAt`           | Date de la dernière livraison réussie.                                              |
| `lastError`               | Dernière erreur de livraison (code HTTP, délai dépassé…).                           |
| `autoDisabledAt`          | Date de la [désactivation automatique](recevoir-les-notifications.md#desactivation-automatique), le cas échéant. |
| `previousSecretExpiresAt` | Pendant un renouvellement du secret, date d'expiration de l'ancien.                 |

## Créer un webhook

Voir le [démarrage rapide](README.md#demarrage-rapide). La mutation `webhookCreer` prend la démarche, l'`url`, la liste `eventTypes` et un `label` facultatif. Elle renvoie le `secret`, qui ne sera plus jamais affiché.

## Modifier un webhook <a href="#modifier-un-webhook" id="modifier-un-webhook"></a>

```graphql
mutation {
  webhookModifier(input: {
    webhook: "V2ViaG9vay0xMg=="
    eventTypes: [dossier_depose, dossier_accepte, dossier_refuse, message_cree]
  }) {
    webhook { id eventTypes }
    errors { message }
  }
}
```

* Seuls les arguments fournis (`url`, `eventTypes`, `label`) sont modifiés.
* `eventTypes` remplace la liste entière.
* Un évènement ajouté à la liste n'est notifié qu'à partir de la modification.
* Changer l'`url` remet à zéro le compteur d'échecs.

{% hint style="info" %}
Pour changer d'URL sans aucune interruption, créez un second webhook vers la nouvelle URL, vérifiez qu'il reçoit bien les notifications, puis supprimez l'ancien. Les deux portent les mêmes numéros de `sequence` : votre [dédoublonnage](recevoir-les-notifications.md#doublons) absorbe la période où ils coexistent.
{% endhint %}

## Désactiver et réactiver <a href="#desactiver-et-reactiver" id="desactiver-et-reactiver"></a>

`webhookDesactiver` suspend les livraisons, par exemple pendant une maintenance de votre application. Les évènements continuent d'être enregistrés (7 jours au plus).

`webhookActiver` reprend les livraisons : les évènements en attente sont livrés dans l'ordre. C'est aussi la mutation à utiliser après une désactivation automatique, une fois le problème corrigé.

```graphql
mutation {
  webhookActiver(input: { webhook: "V2ViaG9vay0xMg==" }) {
    webhook { enabled lastError }
    errors { message }
  }
}
```

{% hint style="info" %}
Sur un webhook actif en attente d'une nouvelle tentative, `webhookActiver` relance la livraison immédiatement. Utile quand vous venez de rétablir votre application.
{% endhint %}

## Renouveler le secret <a href="#renouveler-le-secret" id="renouveler-le-secret"></a>

```graphql
mutation {
  webhookRenouvelerSecret(input: {
    webhook: "V2ViaG9vay0xMg=="
    delaiExpirationAncienSecret: 24
  }) {
    secret
    webhook { previousSecretExpiresAt }
    errors { message }
  }
}
```

Le renouvellement se fait sans interruption :

1. La mutation renvoie le nouveau `secret`.
2. Jusqu'à `previousSecretExpiresAt`, chaque notification porte deux signatures, l'une avec l'ancien secret et l'autre avec le nouveau. Votre application les accepte avec l'un ou l'autre.
3. Déployez le nouveau secret avant cette échéance. L'ancien expire ensuite de lui-même.

`delaiExpirationAncienSecret` fixe la durée de cette transition, en heures : 24 par défaut, 168 (7 jours) au plus.

{% hint style="warning" %}
Attendez la fin de la transition avant de renouveler à nouveau : seul le dernier secret remplacé est conservé, le plus ancien cesse aussitôt de signer.
{% endhint %}

{% hint style="danger" %}
En cas de fuite du secret, renouvelez-le avec `delaiExpirationAncienSecret: 0` : l'ancien secret cesse immédiatement de signer les notifications.
{% endhint %}

## Supprimer un webhook

```graphql
mutation {
  webhookSupprimer(input: { webhook: "V2ViaG9vay0xMg==" }) {
    id
    errors { message }
  }
}
```

La suppression est définitive : les évènements pas encore livrés à ce webhook ne le seront pas.

## Erreurs fréquentes

| Message                                                                 | Cause                                                                                   |
| ----------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Le jeton utilisé est configuré seulement en lecture                     | Les mutations demandent un jeton avec un accès en écriture.                             |
| Le nombre maximum de webhooks (50) est atteint pour cette démarche      | Supprimez un webhook inutilisé avant d'en créer un autre.                               |

Les autres erreurs sont décrites sur la page [Gestion des erreurs](../gestion-des-erreurs.md).
