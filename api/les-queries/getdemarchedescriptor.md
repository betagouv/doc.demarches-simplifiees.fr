---
description: Récupérer les versions des formulaires par démarche
---

# getDemarcheDescriptor

Lors de son cycle de vie, un administrateur d'une démarche peut publier une nouvelle version du formulaire (ex, ajout d'un champ au formulaire).&#x20;

Les nouveaux dossiers crées par les usagers sont donc sur cette nouvelle version du formulaire. Cependant les anciens dossiers déjà traités (accépté/refusé ou classé sans suite) eux ne changent pas de version.

La correspondance entre une démarche, se versions, chacuns des dossiers, et chacuns des champs peut se faire en comparant la valeur de `data.demarche.activeRevision.champDescriptors[x].id` / avec la valeur de `data.demarche.dossiers.champs[x].id`

## Query pour demander le descriptif d'une démarche

Vous pouvez aussi récupérer la description d'une démarche via le endpoint getDemarche

```graphql

  query getDemarcheDescriptor(
    $demarche: FindDemarcheInput!
    $includeRevision: Boolean = false
    $includeService: Boolean = false
  ) {
    demarcheDescriptor(demarche: $demarche) {
      ...DemarcheDescriptorFragment
    }
  }

  fragment ServiceFragment on Service {
    nom
    siret
    organisme
    typeOrganisme
  }


  fragment DemarcheDescriptorFragment on DemarcheDescriptor {
    id
    number
    title
    description
    state
    declarative
    dateCreation
    datePublication
    dateDerniereModification
    dateDepublication
    dateFermeture
    notice { url }
    deliberation { url }
    demarcheURL
    cadreJuridiqueURL
    service @include(if: $includeService) {
      ...ServiceFragment
    }
    revision @include(if: $includeRevision) {
      ...RevisionFragment
    }
  }

  fragment RevisionFragment on Revision {
    id
    datePublication
    champDescriptors {
      ...ChampDescriptorFragment
      ... on RepetitionChampDescriptor {
        champDescriptors {
          ...ChampDescriptorFragment
        }
      }
    }
    annotationDescriptors {
      ...ChampDescriptorFragment
      ... on RepetitionChampDescriptor {
        champDescriptors {
          ...ChampDescriptorFragment
        }
      }
    }
  }

  fragment ChampDescriptorFragment on ChampDescriptor {
    __typename
    id
    label
    description
    required
    ... on DropDownListChampDescriptor {
      options
      otherOption
    }
    ... on MultipleDropDownListChampDescriptor {
      options
    }
    ... on LinkedDropDownListChampDescriptor {
      options
    }
    ... on PieceJustificativeChampDescriptor {
      fileTemplate {
        ...FileFragment
      }
    }
    ... on ExplicationChampDescriptor {
      collapsibleExplanationEnabled
      collapsibleExplanationText
    }
  }

  fragment FileFragment on File {
    __typename
    filename
    contentType
    checksum
    byteSize: byteSizeBigInt
    url
    createdAt
  }

```

### Variables pour récupérer les déscripteurs d'une démarche :

```graphql
{
  "query": <query>,
  "operationName": "getDemarche",
  "variables": {
    "includeRevision": true, 
    "demarche": {"number": 83173
  }
}
```
