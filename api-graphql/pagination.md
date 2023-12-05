---
description: Basée sur un système de curseur
---

# Pagination

La pagination sur l’API GraphQL se fait par « curseur ». Nous vous renvoyons à la définition faite par Microsoft d'un curseur : [https://learn.microsoft.com/fr-fr/office/client-developer/access/desktop-database-reference/what-is-a-cursor](https://learn.microsoft.com/fr-fr/office/client-developer/access/desktop-database-reference/what-is-a-cursor)

> Le curseur est ainsi nommé car il indique la position active dans le jeu de résultats, à l'instar du curseur de l'écran de votre ordinateur.

Concrètement, pour récupérer la prochaine page il faut passer à l’API le « curseur » de la fin de la page précédente.

## Application concrète&#x20;

### Première requete&#x20;

Voici un exemple. On commence par faire une query pour récupérer les 100 premiers dossiers :

```graphql
query getDemarche($demarcheNumber: Int!, $after: String) {
  demarche(number: $demarcheNumber) {
    dossiers(first: 100, after: $after) {
      pageInfo {
        endCursor
        hasNextPage
      }
      nodes {
        id
        number
      }
    }
  }
}
```

```graphql
{
  "query": <query>,
  "operationName": "getDemarche",
  "variables": {
    "demarcheNumber": 1234
  }
}
```

Dans le résultat obtenu il faut lire la valeur du « curseur » dans le champ `demarche.dossiers.pageInfo.endCursor` .&#x20;

### Deuxième requête, et toutes autres :

On peut passer alors ce « curseur » comme argument `after` dans la prochaine query. Et ainsi de suite jusqu’à ce que le champ `demarche.dossiers.pageInfo.hasNextPage` soit égal à `false`.

```graphql
query getDemarche($demarcheNumber: Int!, $after: String) {
  demarche(number: $demarcheNumber) {
    dossiers(first: 100, after: $after) {
      pageInfo {
        endCursor
        hasNextPage
      }
      nodes {
        id
        number
      }
    }
  }
}
```

```graphql
{
  "query": <query>,
  "operationName": "getDemarche",
  "variables": {
    "demarcheNumber": 1234,
    "after" "abc"
  }
}
```

Pour un exemple d'implementation, rendez-vous sur :&#x20;

{% content-ref url="cas-dusages-exemple-dimplementation/recuperer-tous-les-dossiers-pagination.md" %}
[recuperer-tous-les-dossiers-pagination.md](cas-dusages-exemple-dimplementation/recuperer-tous-les-dossiers-pagination.md)
{% endcontent-ref %}
