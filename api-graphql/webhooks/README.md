---
description: Être notifié en temps réel des évènements sur les dossiers d'une démarche
---

# Webhooks

Un webhook permet à **demarche.numerique.gouv.fr** de prévenir votre application dès qu'un évènement survient sur un dossier : dépôt, passage en instruction, décision, nouveau message… Vous n'avez plus besoin d'interroger l'API à intervalles réguliers pour détecter les changements.

## Comment ça marche

1. Vous déclarez, par une requête à l'API GraphQL, l'adresse (URL) de votre application et les évènements qui vous intéressent.
2. À chaque évènement, **demarche.numerique.gouv.fr** envoie une requête `POST` signée à cette adresse, en quelques secondes.
3. Votre application vérifie la signature, puis récupère le dossier avec la query [`dossier`](../les-queries/getdossier.md) de l'API GraphQL pour lire son contenu à jour.

Les notifications sont volontairement **légères** : elles indiquent quel dossier a changé et pourquoi, sans aucune donnée du dossier. Les données des usagers ne transitent que par l'API, protégée par votre jeton.

## Prérequis

* Un [jeton d'authentification](../jeton-dauthentification/README.md) avec un accès **en écriture** à la démarche, pour créer et gérer les webhooks.
* Une URL en **HTTPS**, joignable depuis internet. Elle peut être soumise à une validation manuelle par l'équipe de **demarche.numerique.gouv.fr**.

## Les évènements <a href="#evenements" id="evenements"></a>

Vous choisissez à la création du webhook les évènements qui vous intéressent ; vous pourrez [modifier cette liste](gerer-ses-webhooks.md#modifier-un-webhook) par la suite.

### Cycle de vie du dossier

| Évènement                         | Quand                                                                                                         |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `dossier_depose`                  | L'usager dépose son dossier.                                                                                  |
| `dossier_modifie`                 | Le dossier en construction est modifié : l'usager dépose des modifications ou corrige son identité, ou un instructeur modifie ses champs. |
| `dossier_en_instruction`          | Le dossier passe en instruction, manuellement ou automatiquement.                                             |
| `dossier_accepte`                 | Le dossier est accepté, manuellement ou automatiquement.                                                      |
| `dossier_refuse`                  | Le dossier est refusé, manuellement ou automatiquement.                                                       |
| `dossier_sans_suite`              | Le dossier est classé sans suite.                                                                             |
| `dossier_repasse_en_construction` | Le dossier repasse en construction, par exemple à la suite d'une demande de correction.                       |
| `dossier_repasse_en_instruction`  | Une décision est annulée : le dossier revient en instruction.                                                 |
| `dossier_supprime`                | Le dossier n'est plus accessible à l'administration : supprimé par un instructeur, supprimé par l'usager alors qu'il était en construction, ou arrivé au terme de sa durée de conservation. |
| `dossier_restaure`                | Un dossier supprimé redevient accessible à l'administration : restauration, ou allongement de la durée de conservation. |

### Instruction

| Évènement                   | Quand                                                       |
| --------------------------- | ----------------------------------------------------------- |
| `correction_demandee`       | Un instructeur demande une correction à l'usager.           |
| `groupe_instructeur_change` | Le dossier change de groupe instructeur (routage ou réaffectation). |
| `avis_cree`                 | Un avis est demandé à un expert.                            |
| `avis_repondu`              | L'expert rend son avis (première réponse seulement).        |
| `dossier_label_ajoute`      | Un label est ajouté au dossier.                             |
| `dossier_label_supprime`    | Un label est retiré du dossier.                             |

### Messagerie

| Évènement      | Quand                                                                                                |
| -------------- | ---------------------------------------------------------------------------------------------------- |
| `message_cree` | Un message est posté dans la messagerie du dossier. Les messages signés par la plateforme (accusé de dépôt, notification de décision…) ne sont pas notifiés. |

{% hint style="info" %}
* Aucun évènement n'est émis pour un dossier **en brouillon**, que l'API n'expose pas.
* Une même action peut produire plusieurs évènements, toujours livrés dans l'ordre. Sur une démarche déclarative, par exemple, `dossier_depose` est suivi de `dossier_en_instruction` ou `dossier_accepte`.
{% endhint %}

### Idées d'intégration

| Besoin                                                     | Évènements à suivre                                                                          |
| ---------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Verser chaque nouveau dossier dans votre SI                | `dossier_depose`, `dossier_modifie`                                                          |
| Suivre les décisions (tableau de bord, archivage…)         | `dossier_accepte`, `dossier_refuse`, `dossier_sans_suite`, `dossier_repasse_en_instruction`  |
| Traiter les échanges avec l'usager dans un outil de support | `message_cree`, `correction_demandee`, `dossier_modifie`                                    |
| Déclencher une étape quand un expert a répondu             | `avis_repondu`                                                                               |
| Répercuter l'organisation de l'instruction                 | `groupe_instructeur_change`, `dossier_label_ajoute`, `dossier_label_supprime`                |
| Tenir à jour votre copie quand un dossier est supprimé     | `dossier_supprime`, `dossier_restaure`                                                       |

## Démarrage rapide <a href="#demarrage-rapide" id="demarrage-rapide"></a>

### 1. Créer le webhook

Le webhook se crée avec la mutation `webhookCreer` de l'API GraphQL, envoyée comme toute autre requête à `https://demarche.numerique.gouv.fr/api/v2/graphql` avec votre jeton. Dans cet exemple, il s'abonne au dépôt, à l'acceptation et au refus des dossiers : remplacez ces `eventTypes` par les [évènements](#evenements) qui vous intéressent.

```graphql
mutation {
  webhookCreer(input: {
    demarche: { number: 12345 }
    url: "https://mon-si.exemple.gouv.fr/webhooks/demarche-numerique"
    eventTypes: [dossier_depose, dossier_accepte, dossier_refuse]
    label: "Synchronisation SI"
  }) {
    webhook { id enabled }
    secret
    errors { message }
  }
}
```

{% hint style="warning" %}
Le `secret` n'est affiché qu'une fois, dans la réponse à cette mutation. Conservez-le comme un mot de passe : il sert à vérifier que les notifications proviennent bien de **demarche.numerique.gouv.fr**. En cas de perte, [générez-en un nouveau](gerer-ses-webhooks.md#renouveler-le-secret).
{% endhint %}

### 2. Recevoir les notifications

Votre application reçoit des requêtes de cette forme :

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

[Vérifiez la signature](recevoir-les-notifications.md#verifier-la-signature), puis répondez par un code `2xx` dans les 15 secondes. Le détail est sur la page [Recevoir les notifications](recevoir-les-notifications.md).

{% hint style="warning" %}
**La vérification de la signature est obligatoire.** N'importe qui peut envoyer une requête à votre URL : seule la signature, calculée avec votre secret, prouve que la notification vient bien de **demarche.numerique.gouv.fr** et non d'un tiers qui se ferait passer pour nous. Rejetez toute requête dont la signature est invalide.
{% endhint %}

### 3. Récupérer les dossiers

Pour chaque évènement, lisez le dossier `dossier_number` avec la query [`dossier`](../les-queries/getdossier.md) de l'API GraphQL. L'[exemple complet](../cas-dusages-exemple-dimplementation/webhooks-recevoir-les-nouveaux-dossiers-dans-son-si.md) montre une application qui reçoit les notifications et lit les dossiers, en Node.js, Python et Ruby (Rails).

## Limites

* 50 webhooks au plus par démarche.
* Un webhook ne reçoit que les évènements postérieurs à sa création. Pour importer les dossiers existants, [parcourez la démarche](../cas-dusages-exemple-dimplementation/pagination-recuperer-tous-les-dossiers-dune-demarche.md) une première fois.
* Les évènements sont conservés 7 jours : passé ce délai, ceux qui n'ont pas pu être livrés sont perdus.

## Pour aller plus loin

* [Recevoir les notifications](recevoir-les-notifications.md) : format, vérification de la signature, nouvelles tentatives.
* [Gérer ses webhooks](gerer-ses-webhooks.md) : lister, modifier, désactiver, renouveler le secret, supprimer.
* [Exemple complet](../cas-dusages-exemple-dimplementation/webhooks-recevoir-les-nouveaux-dossiers-dans-son-si.md) : recevoir les nouveaux dossiers dans son SI.
