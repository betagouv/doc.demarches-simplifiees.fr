---
description: Récupérer les versions du formulaire par démarche
---

# getDemarcheDescriptor

Lors de son cycle de vie, un administrateur d'une démarche peut publier une nouvelle version du formulaire (ex, ajout d'un champ au formulaire).&#x20;

Les dossiers crées a partir de ce moment auront donc la derniere version du formulaire de la démarche.&#x20;

Cependant les anciens dossiers (accépté/refusé ou classé sans suite) eux ne changent pas de version (ex: on ne vas demandé à un usager de mettre à jour son dossier si celui ci a été accepté).

getDemarcheDescriptor (et toutes les notions de champDescriptor) permet de mapper un champs à son descripteur de champ de manière fiable. En somme :&#x20;

* Toutes les versions d'une démarche (au travers des objets `Revision` : [https://www.demarches-simplifiees.fr/graphql/schema/index.html#definition-Revision](https://www.demarches-simplifiees.fr/graphql/schema/index.html#definition-Revision)).
* Une `Revision` porte le schema de donnée des dossiers déposés lorsque la démarche était sur cette version. Vous trouverez donc la liste des champs et des annotations sur la révision cf:  [https://www.demarches-simplifiees.fr/graphql/schema/index.html#definition-ChampDescriptor](https://www.demarches-simplifiees.fr/graphql/schema/index.html#definition-ChampDescriptor)
* Les champs et annotations des dossiers de la démarche  `(data.demarche.dossiers.champs[].champDescriptorId)` correspondent à l'une des Revision de la démarches `data.demarche.revisions[].champDescriptors[].id`&#x20;

## Query pour demander le descriptif d'une démarche

Vous pouvez aussi récupérer la description d'une démarche via le endpoint getDemarcheDescriptor

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
  "operationName": "getDemarcheDescriptor",
  "variables": {
    "includeRevision": true, 
    {"demarche": { "number":64482 }, "includeRevision": true }
  }
}
```
