---
description: Récupérer les dossiers par groupe d'instructeur
---

# getGroupeInstructeur

Le cas d'usage classique pour ce endpoint est le cas d'une démarche a forte volumétrie ayant des groupes d'instructeurs

## Query pour récupérer les dossiers d'un groupe d'instructeur

<pre class="language-graphql"><code class="lang-graphql"><strong>query getGroupeInstructeur(
</strong>  $groupeInstructeurNumber: Int!
  $state: DossierState
  $order: Order
  $first: Int
  $after: String
  $archived: Boolean
  $revision: ID
  $createdSince: ISO8601DateTime
  $updatedSince: ISO8601DateTime
  $deletedOrder: Order
  $deletedFirst: Int
  $deletedAfter: String
  $deletedSince: ISO8601DateTime
  $includeDossiers: Boolean = false
  $includeDeletedDossiers: Boolean = false
  $includeChamps: Boolean = true
  $includeAnotations: Boolean = true
  $includeTraitements: Boolean = true
  $includeInstructeurs: Boolean = true
  $includeAvis: Boolean = false
  $includeMessages: Boolean = false
  $includeGeometry: Boolean = false
) {
  groupeInstructeur(number: $groupeInstructeurNumber) {
    id
    number
    label
    instructeurs @include(if: $includeInstructeurs) {
      id
      email
    }
    dossiers(
      state: $state
      order: $order
      first: $first
      after: $after
      archived: $archived
      createdSince: $createdSince
      updatedSince: $updatedSince
      revision: $revision
    ) @include(if: $includeDossiers) {
      pageInfo {
        ...PageInfoFragment
      }
      nodes {
        ...DossierFragment
      }
    }
    deletedDossiers(
      order: $deletedOrder
      first: $deletedFirst
      after: $deletedAfter
      deletedSince: $deletedSince
    ) @include(if: $includeDeletedDossiers) {
      pageInfo {
        ...PageInfoFragment
      }
      nodes {
        ...DeletedDossierFragment
      }
    }
  }
}
</code></pre>

### Variables pour récupérer tous les dossiers du groupe d'instructeur

```graphql
# Tout les dossiers d’un groupe instructeur
{
  "query": <query>,
  "operationName": "getGroupeInstructeur",
  "variables": {
    "groupeInstructeurNumber": 1234,
    "includeDossiers": true,
    "includeChamps": true
  }
}
```
