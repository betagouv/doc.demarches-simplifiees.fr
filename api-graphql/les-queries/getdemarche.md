---
description: Récupérer les dossiers par démarche
---

# getDemarche

Le cas d'usage classique de ce endpoint est de lister tous les dossiers d'une démarche (par exemple, si vous souhaitez faire des statistiques).



{% hint style="info" %}
Là encore, réferrez-vous [au schéma de l'API](https://www.demarches-simplifiees.fr/graphql/schema/index.html) qui documente à quoi correspond chaque attribut.
{% endhint %}

## Query pour récupérer les informations d'une démarche :&#x20;

<pre class="language-graphql"><code class="lang-graphql"><strong>query getDemarche(
</strong>  $demarcheNumber: Int!
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
  $pendingDeletedOrder: Order
  $pendingDeletedFirst: Int
  $pendingDeletedAfter: String
  $pendingDeletedSince: ISO8601DateTime
  $includeGroupeInstructeurs: Boolean = false
  $includeDossiers: Boolean = false
  $includeDeletedDossiers: Boolean = false
  $includePendingDeletedDossiers: Boolean = false
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
  demarche(number: $demarcheNumber) {
    id
    number
    title
    state
    declarative
    dateCreation
    dateFermeture
    activeRevision @include(if: $includeRevision) {
      ...RevisionFragment
    }
    groupeInstructeurs @include(if: $includeGroupeInstructeurs) {
      ...GroupeInstructeurFragment
    }
    service @include(if: $includeService) {
      ...ServiceFragment
    }
    labels @include(if: $includeLabels){
      ...LabelFragment
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
    pendingDeletedDossiers(
      order: $pendingDeletedOrder
      first: $pendingDeletedFirst
      after: $pendingDeletedAfter
      deletedSince: $pendingDeletedSince
    ) @include(if: $includePendingDeletedDossiers) {
      pageInfo {
        ...PageInfoFragment
      }
      nodes {
        ...DeletedDossierFragment
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

fragment ServiceFragment on Service {
  nom
  siret
  organisme
  typeOrganisme
}

fragment GroupeInstructeurFragment on GroupeInstructeur {
  id
  number
  label
  instructeurs @include(if: $includeInstructeurs) {
    id
    email
  }
}

fragment DossierFragment on Dossier {
  id
  number
  archived
  state
  dateDerniereModification
  dateDepot
  datePassageEnConstruction
  datePassageEnInstruction
  dateTraitement
  dateExpiration
  dateSuppressionParUsager
  motivation
  motivationAttachment {
    ...FileFragment
  }
  attestation {
    ...FileFragment
  }
  pdf {
    url
  }
  usager {
    email
  }
  groupeInstructeur {
    ...GroupeInstructeurFragment
  }
  demandeur {
    __typename
    ... on PersonnePhysique {
      civilite
      nom
      prenom
    }
    ... on PersonneMoraleIncomplete { siret }
    ...PersonneMoraleFragment
  }
  demarche {
    revision {
      id
    }
  }
  instructeurs @include(if: $includeInstructeurs) {
    id
    email
  }
  traitements @include(if: $includeTraitements) {
    state
    emailAgentTraitant
    dateTraitement
    motivation
  }
  champs @include(if: $includeChamps) {
    ...ChampFragment
    ...RootChampFragment
  }
  annotations @include(if: $includeAnotations) {
    ...ChampFragment
    ...RootChampFragment
  }
  avis @include(if: $includeAvis) {
    ...AvisFragment
  }
  messages @include(if: $includeMessages) {
    ...MessageFragment
  }
  labels @include(if: $includeLabels) {
    ...LabelFragment
  }
}

fragment DeletedDossierFragment on DeletedDossier {
  id
  number
  dateSupression
  state
  reason
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

fragment AvisFragment on Avis {
  id
  question
  reponse
  dateQuestion
  dateReponse
  claimant {
    email
  }
  expert {
    email
  }
  attachments {
    ...FileFragment
  }
}

<strong>fragment MessageFragment on Message {
</strong>  id
  email
  body
  createdAt
  attachments {
    ...FileFragment
  }
}

fragment LabelFragment on Label {
  id
  name
  color
}

fragment GeoAreaFragment on GeoArea {
  id
  source
  description
  geometry @include(if: $includeGeometry) {
    type
    coordinates
  }
  ... on ParcelleCadastrale {
    commune
    numero
    section
    prefixe
    surface
  }
}

fragment RootChampFragment on Champ {
  ... on RepetitionChamp {
    rows {
      champs {
        ...ChampFragment
      }
    }
  }
  ... on CarteChamp {
    geoAreas {
      ...GeoAreaFragment
    }
  }
  ... on DossierLinkChamp {
    dossier {
      id
      number
      state
    }
  }
}

fragment ChampFragment on Champ {
  id
  __typename
  label
  stringValue
  ... on DateChamp {
    date
  }
  ... on DatetimeChamp {
    datetime
  }
  ... on CheckboxChamp {
    checked: value
  }
  ... on DecimalNumberChamp {
    decimalNumber: value
  }
  ... on IntegerNumberChamp {
    integerNumber: value
  }
  ... on CiviliteChamp {
    civilite: value
  }
  ... on LinkedDropDownListChamp {
    primaryValue
    secondaryValue
  }
  ... on MultipleDropDownListChamp {
    values
  }
  ... on PieceJustificativeChamp {
    files {
      ...FileFragment
    }
  }
  ... on AddressChamp {
    address {
      ...AddressFragment
    }
  }
  ... on CommuneChamp {
    commune {
      name
      code
    }
    departement {
      name
      code
    }
  }
  ... on DepartementChamp {
    departement {
      name
      code
    }
  }
  ... on RegionChamp {
    region {
      name
      code
    }
  }
  ... on PaysChamp {
    pays {
      name
      code
    }
  }
  ... on SiretChamp {
    etablissement {
      ...PersonneMoraleFragment
    }
  }
}

fragment PersonneMoraleFragment on PersonneMorale {
  siret
  siegeSocial
  naf
  libelleNaf
  address {
    ...AddressFragment
  }
  entreprise {
    siren
    capitalSocial
    numeroTvaIntracommunautaire
    formeJuridique
    formeJuridiqueCode
    nomCommercial
    raisonSociale
    siretSiegeSocial
    codeEffectifEntreprise
    dateCreation
    nom
    prenom
    attestationFiscaleAttachment {
      ...FileFragment
    }
    attestationSocialeAttachment {
      ...FileFragment
    }
  }
  association {
    rna
    titre
    objet
    dateCreation
    dateDeclaration
    datePublication
  }
}

fragment FileFragment on File {
  filename
  contentType
  checksum
  byteSize: byteSizeBigInt
  url
}

fragment AddressFragment on Address {
  label
  type
  streetAddress
  streetNumber
  streetName
  postalCode
  cityName
  cityCode
  departmentName
  departmentCode
  regionName
  regionCode
}

fragment PageInfoFragment on PageInfo {
  hasPreviousPage
  hasNextPage
  endCursor
}
</code></pre>

Une fois cette query en place, vous pouvez utilisez differentes variantes pour executer une requete et en sortir les résultats suivants :

### Variables pour récupérer tous les dossiers d'une démarche

```graphql
{
  "query": <query>,
  "operationName": "getDemarche",
  "variables": {
    "demarcheNumber": 1234,
    "includeDossiers": true,
    "includeChamps": true
  }
}
```

### Variables pour récupérer tous les dossiers en instruction d'une démarche&#x20;

```graphql
{
  "query": <query>,
  "operationName": "getDemarche",
  "variables": {
    "demarcheNumber": 1234,
    "state": "en_instruction",
    "includeDossiers": true,
    "includeChamps": true
  }
}
```

### Variables pour récupérer tous les dossiers en attente de suppression définitive

```graphql
{
  "query": <query>,
  "operationName": "getDemarche",
  "variables": {
    "demarcheNumber": 1234,
    "includePendingDeletedDossiers": true
  }
}

```

### Variables pour récupérer la liste des dossiers définitivement supprimés

```graphql
{
  "query": <query>,
  "operationName": "getDemarche",
  "variables": {
    "demarcheNumber": 1234,
    "includeDeletedDossiers": true
  }
}
```
