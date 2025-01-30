---
description: Récupérer un dossier et ses dépendances
---

# getDossier

Le cas d'usage classique de ce endpoint est d'afficher les informations complète d'un dossier sur une page pour visualiser le dossier dans son entièreté.

## Query pour récupérer les informations d'un dossier unitairement

```graphql
query getDossier(
  $dossierNumber: Int!
  $includeRevision: Boolean = false
  $includeService: Boolean = false
  $includeChamps: Boolean = true
  $includeAnotations: Boolean = true
  $includeTraitements: Boolean = true
  $includeInstructeurs: Boolean = true
  $includeAvis: Boolean = false
  $includeMessages: Boolean = false
  $includeLabels: Boolean = false
  $includeGeometry: Boolean = false
) {
  dossier(number: $dossierNumber) {
    ...DossierFragment
    demarche {
      ...DemarcheDescriptorFragment
    }
  }
}
```

### Variables pour récupérer un dossier

<pre class="language-graphql"><code class="lang-graphql"><strong># Un dossier par son numero
</strong>{
  "query": &#x3C;query>,
  "operationName": "getDossier",
  "variables": {
    "dossierNumber": 1234,
    "includeChamps": true,
    "includeAnotations": true,
    "includeAvis": true
  }
}
</code></pre>
