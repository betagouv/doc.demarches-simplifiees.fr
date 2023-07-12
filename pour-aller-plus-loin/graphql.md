# API GraphQL

{% hint style="info" %}
Pour être informé des évolutions de l'API, vous pouvez vous [inscrire à notre newsletter dédiée](https://7b97debb.sibforms.com/serve/MUIEAOpYNmRDuQF5Ib0fqC2aNgJW1j\_5EkA8AUkO6UBxCpofKKXPFNSiCInma2JfRsXGchOns5RZ0UWYVbC-8utoVRMT1wK\_1JRH7o0tbP-DQ07YtTsmkotgdSobF2n4MemAagXAO1mhB1vx3vqxijaSUYKnu-kg6O-1\_l7WcTNLdgHoQM1EfkJjaVpc3TPvaOYgj6Uk35dqi86-)
{% endhint %}

L'API GraphQL de demarches-simplifiees.fr permet de **consulter** :

* les informations d'une démarche,
* la liste et les détails des dossiers d'une démarche,
* les détails d'un dossier spécifique.

Elle permet également de **modifier** certaines informations d'un dossier :

* Envoyer un message à l'usager d'un dossier ;
* Changer l'état d'un dossier (accepté, refusé, etc.)

### Présentation de l'API

{% embed url="https://vimeo.com/333749814" %}
3 minutes pour tout savoir de notre API
{% endembed %}

### Aperçu général de GraphQL

Cette API suit le paradigme GraphQL. Pour plus d'informations sur le fonctionnement de GraphQL en général, vous pouvez consulter les liens suivants :

* [Introduction aux concepts et raisons d'être de GraphQL](https://blog.octo.com/graphql-et-pourquoi-faire/) (en français)
* [Documentation officielle de la spécification GraphQL](https://graphql.org) (en anglais)

### Accéder à l’API GraphQL de demarches-simplifiees.fr

#### Point d'entrée de l’API

L’API est accessible à l’adresse **https://www.demarches-simplifiees.fr/api/v2/graphql** (cette adresse n’est pas visitable dans un navigateur). Elle renvoie des données au format JSON, à travers un transport HTTPS.

Pour construire une requête et interpréter les réponses, consultez la [**documentation complète du schéma de l’API**](https://www.demarches-simplifiees.fr/graphql/schema/).

#### Éditeur de requêtes en ligne

Une fois authentifié en tant qu’administrateur disposant d’un token, vous pouvez également accéder à l’**éditeur de requêtes en ligne** (attention, ne confondez pas cette adresse avec celle de l’endpoint, qui ressemble) :

[https://www.demarches-simplifiees.fr/graphql](https://www.demarches-simplifiees.fr/graphql)

### Authentification

Tous les appels sont authentifiés et doivent donc fournir un jeton valide qui est accessible dans la partie profil de l’administrateur. Ce jeton doit être fourni dans l’en-tête HTTP `Authorization` de la requête.

&#x20;`Authorization: Bearer token=valeur_du_jeton`.

{% swagger baseUrl="https://www.demarches-simplifiees.fr" path="/api/v2/graphql" method="post" summary="GraphQL" %}
{% swagger-description %}
Le point d’entrée de l’API GraphQL.
{% endswagger-description %}

{% swagger-parameter in="header" name="Content-Type" type="string" %}
application/json
{% endswagger-parameter %}

{% swagger-parameter in="header" name="Authorization" type="string" %}
Le jeton de l’administrateur
{% endswagger-parameter %}

{% swagger-parameter in="body" name="query" type="string" %}
La requête GraphQL
{% endswagger-parameter %}

{% swagger-parameter in="body" name="variables" type="object" %}
Les variables de la requête
{% endswagger-parameter %}

{% swagger-response status="200" description="Reponse GraphQL" %}
```
{
  "data": {
    "dossier": {
      "id": "RG9zc2llci0xMDMyODgy",
      "number": "1032882"
    }
  },
  "errors": [
    {
      "message": "Field 'dosier' doesn't exist on type 'Query'",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "path": [
        "query getDossier",
        "dosier"
      ],
      "extensions": {
        "code": "undefinedField",
        "typeName": "Query",
        "fieldName": "dosier"
      }
    }
  ]
}
```
{% endswagger-response %}
{% endswagger %}

### Exemple de requête

#### Pour tester que tout fonctionne bien

Pour tester l’API, le plus simple est d’effectuer une requête [curl](https://fr.wikipedia.org/wiki/CURL) telle que ci dessous. Le principe est le même avec un autre client HTTP : remplacez **votre\_token** et **votre\_numero\_de\_demarche** par les valeurs souhaitez, et n’oubliez pas de préciser le **content-type :**

```graphql
curl 'https://www.demarches-simplifiees.fr/api/v2/graphql' \
    --header 'Authorization: Bearer <votre_token>' \
    --header 'Content-Type: application/json' \
    --data '{ "query": "query getDemarche($demarcheNumber: Int!) { demarche(number: $demarcheNumber) { id dossiers { nodes { id demandeur { ... on PersonnePhysique { civilite nom prenom } ... on PersonneMorale { siret } } } } } }", "variables": { "demarcheNumber": <votre_numero_de_demarche> } }'
```

Vous devriez alors obtenir des informations en sortie. S’il y a des dossiers dans votre démarche, cette requête vous donne les noms des demandeurs. Si la démarche s’adresse à des entreprises, vous aurez le numéro SIRET des demandeurs.

### Récupérer les informations d’une démarche, et les dossiers associés

Voici une requête plus lisible, mais plus complexe, qui va chercher beaucoup d’informations sur les dossiers

#### Queries

```graphql
query getDemarche(
  $demarcheNumber: Int!
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

query getGroupeInstructeur(
  $groupeInstructeurNumber: Int!
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
  $includeGeometry: Boolean = false
) {
  dossier(number: $dossierNumber) {
    ...DossierFragment
    demarche {
      ...DemarcheDescriptorFragment
    }
  }
}

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
  notice {
    url
  }
  deliberation {
    url
  }
  demarcheUrl
  cadreJuridiqueUrl
  service @include(if: $includeService) {
    ...ServiceFragment
  }
  revision @include(if: $includeRevision) {
    ...RevisionFragment
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

fragment MessageFragment on Message {
  id
  email
  body
  createdAt
  attachments {
    ...FileFragment
  }
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

```

#### Variables

<pre class="language-graphql"><code class="lang-graphql"># Tout les dossiers d’une démarche
{
  "query": &#x3C;query>,
  "operationName": "getDemarche",
  "variables": {
    "demarcheNumber": 1234,
    "includeDossiers" true,
    "includeChamps": true
  }
}

# Tout les dossiers "en_instruction" d’une démarche
{
  "query": &#x3C;query>,
  "operationName": "getDemarche",
  "variables": {
    "demarcheNumber": 1234,
    "state": "en_instruction",
    "includeDossiers": true,
    "includeChamps": true
  }
}

<strong># Tout les dossiers en attente de suppression d’une démarche
</strong>{
  "query": &#x3C;query>,
  "operationName": "getDemarche",
  "variables": {
    "demarcheNumber": 1234,
    "includePendingDeletedDossiers": true
  }
}

# Tout les dossiers supprimés d’une démarche
{
  "query": &#x3C;query>,
  "operationName": "getDemarche",
  "variables": {
    "demarcheNumber": 1234,
    "includeDeletedDossiers": true
  }
}

# Tout les dossiers d’un groupe instructeur
{
  "query": &#x3C;query>,
  "operationName": "getGroupeInstructeur",
  "variables": {
    "groupeInstructeurNumber": 1234,
    "includeDossiers": true,
    "includeChamps": true
  }
}

# Un dossier par son numero
{
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

#### Pagination

La pagination sur l’API GraphQL se fait par « curseur ». Ça veut dire que pour récupérer la prochaine page il faut passer à l’API le « curseur » de la fin de la page précédente.

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

Dans le résultat obtenu il faut lire la valeur du « curseur » dans le champ `demarche.dossiers.pageInfo.endCursor` . On peut passer alors ce « curseur » comme argument `after` dans la prochaine query. Et ainsi de suite jusqu’à ce que le champ `demarche.dossiers.pageInfo.hasNextPage` soit égal à `false`.

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

### Modifier l’état ou les informations d’un dossier

Certaines opérations sur dossier sont accessibles via l’API :

* Affecter un dossier à un groupe instructeur
* Changer l’état du dossier
* Envoyer un message à l’usager qui a déposé le dossier
* Ajouter des annotations privées sur un dossier
* Créer ou modifier les « groupes instructeurs »

{% hint style="warning" %}
Les identifiants utilisés dans les variables `input` des mutations sont les identifiants « techniques » que vous devez récupérer dans les réponses des requêtes précédentes. En particulier, l’identifiant du dossier est différent du « numéro de dossier »
{% endhint %}

```graphql
query getDossierIdByNumber($dossierNumber: Int!) {
  dossier(number: $dossierNumber) {
    id
  }
}

{
  "query": <query>,
  "operationName": "getDossierIdByNumber",
  "variables": {
    "dossierNumber": 1234
  }
}
```

{% hint style="info" %}
Si un identifiant d’instructeur est demandé, il s’agit d’identifier l’instructeur qui sera enregistré dans le système comme responsable de l’opération. Pour récupérer la liste des instructeurs avec leurs identifiants, vous avez plusieurs possibilités.
{% endhint %}

Vous pouvez récupérer tous les instructeurs qui suivent un dossier :

```graphql
query getInstructeursForDossier($dossierNumber: Int!) {
  dossier(number: $dossierNumber) {
    instructeurs {
      id
      email
    }
  }
}

{
  "query": <query>,
  "operationName": "getInstructeursForDossier",
  "variables": {
    "dossierNumber": 1234
  }
}
```

Vous pouvez récupérer tous les instructeurs d’un « groupe instructeur » :

```graphql
query getInstructeursForGroupeInstructeur($groupeInstructeurNumber: Int!) {
  groupeInstructeur(number: $groupeInstructeurNumber) {
    instructeurs {
      id
      email
    }
  }
}

{
  "query": <query>,
  "operationName": "getInstructeursForGroupeInstructeur",
  "variables": {
    "groupeInstructeurNumber": 1234
  }
}
```

#### Queries <a href="#mutations-queries" id="mutations-queries"></a>

```graphql
mutation dossierArchiver($input: DossierArchiverInput!) {
  dossierArchiver(input: $input) {
    dossier {
      id
    }
    errors {
      message
    }
  }
}

mutation dossierPasserEnInstruction($input: DossierPasserEnInstructionInput!) {
  dossierPasserEnInstruction(input: $input) {
    dossier {
      id
    }
    errors {
      message
    }
  }
}

mutation dossierAccepter($input: DossierAccepterInput!) {
  dossierAccepter(input: $input) {
    dossier {
      id
    }
    errors {
      message
    }
  }
}

mutation dossierRefuser($input: DossierRefuserInput!) {
  dossierRefuser(input: $input) {
    dossier {
      id
    }
    errors {
      message
    }
  }
}

mutation dossierClasserSansSuite($input: DossierClasserSansSuiteInput!) {
  dossierClasserSansSuite(input: $input) {
    dossier {
      id
    }
    errors {
      message
    }
  }
}

```

#### Variables <a href="#mutations-variables" id="mutations-variables"></a>

```graphql
# Archiver un dossier
{
  "query": <query>,
  "operationName": "dossierArchiver",
  "variables": {
    "input": {
      "dossierId": "UHJvY4VkdXKlLTI5NTgw",
      "instructeurId": "OPJvtN7kdXKlLTI4NTlf"
    }
  }
}

# Passer un dossier en instruction
{
  "query": <query>,
  "operationName": "dossierPasserEnInstruction",
  "variables": {
    "input": {
      "dossierId": "UHJvY4VkdXKlLTI5NTgw",
      "instructeurId": "OPJvtN7kdXKlLTI4NTlf"
    }
  }
}

# Accepter un dossier
{
  "query": <query>,
  "operationName": "dossierAccepter",
  "variables": {
    "input": {
      "dossierId": "UHJvY4VkdXKlLTI5NTgw",
      "instructeurId": "OPJvtN7kdXKlLTI4NTlf",
      "motivation": "J’accepte ce dossier"
    }
  }
}

# Refuser un dossier
{
  "query": <query>,
  "operationName": "dossierRefuser",
  "variables": {
    "input": {
      "dossierId": "UHJvY4VkdXKlLTI5NTgw",
      "instructeurId": "OPJvtN7kdXKlLTI4NTlf",
      "motivation": "Je refuse ce dossier"
    }
  }
}

# Classer un dossier sans suite
{
  "query": <query>,
  "operationName": "dossierClasserSansSuite",
  "variables": {
    "input": {
      "dossierId": "UHJvY4VkdXKlLTI5NTgw",
      "instructeurId": "OPJvtN7kdXKlLTI4NTlf",
      "motivation": "Je classe ce dossier sans suite"
    }
  }
}
```

{% hint style="info" %}
Pour plus d’informations sur le format des requêtes et des réponses, consultez la [documentation complète du schéma de l’API](https://demarches-simplifiees-graphql.netlify.com/).
{% endhint %}

### Envoyer un message

Vous pouvez envoyer des messages au dépositaire du dossier. Une notification par email sera envoyée et le message apparaitra dans l’historique des messages.

#### Queries

```graphql
mutation dossierEnvoyerMessage($input: DossierEnvoyerMessageInput!) {
  dossierEnvoyerMessage(input: $input) {
    message {
      id
    }
    errors {
      message
    }
  }
}
```

#### Variables

```graphql
{
  "query": <query>,
  "operationName": "dossierEnvoyerMessage",
  "variables": {
    "input": {
      "dossierId": "UHJvY4VkdXKlLTI5NTgw",
      "instructeurId": "OPJvtN7kdXKlLTI4NTlf",
      "body": "Bonjour !"
    }
  }
}
```

### Gestion des erreurs

La gestion des erreurs est une partie essentielle de toute interaction avec une API. Dans notre API GraphQL, nous utilisons deux mécanismes principaux pour gérer les erreurs : les erreurs au niveau racine et les erreurs des mutations.

#### Erreurs au niveau racine

Les erreurs au niveau racine sont renvoyées lorsque la requête GraphQL ne peut pas être traitée en raison d’une erreur. Les codes d’erreur courants incluent :

* `not_found` : l’objet demandé n’a pas été trouvé.
* `invalid_null` : un champ est nul alors qu’il est déclaré comme non-nul. Dans la très grande majorité des cas, c’est un bug dans démarches-simplifiées.fr.
* `unauthorized` : l’utilisateur n’est pas autorisé à exécuter l’action demandée.
* `bad_request` : la requête était mal formée.
* `graphql_parse_error` : la requête GraphQL n’a pas pu être analysée correctement.
* `internal_server_error` : une erreur interne du serveur s’est produite lors de la tentative de traitement de la requête.
* `timeout` : la requête a dépassé le temps d'exécution maximal de 10 secondes par nœud.

Ces codes d’erreur se trouvent dans l’objet `extensions` des erreurs globales.

{% hint style="info" %}
Il est important de noter que l’erreur `timeout` peut apparaître même lorsque certaines données sont renvoyées. C’est parce que le délai d’exécution de 10 secondes est défini par nœud. Ainsi, certains nœuds peuvent retourner des données tandis que d’autres peuvent dépasser le délai imparti. De même, une erreur `invalid_null` peut apparaître avec une réponse incomplète.
{% endhint %}

{% hint style="danger" %}
Dans les faits, à moins de vraiment savoir ce que vous faites, on conseille de traiter toute présence d'erreurs à la racine comme fatale.
{% endhint %}

```json
{
  "data": null,
  "errors": [
    {
      "message": "Demarche not found",
      "extensions": { "code": "not_found" }
    }
  ]
}
```

#### Erreurs des mutations

Dans le cas des mutations, nous utilisons une approche "d’erreur en tant que données". Chaque mutation expose un champ `erreurs` qui est rempli en cas d’erreur et est nul autrement. Les erreurs de mutation n’ont pas de code. Si des erreurs sont présentes, le champ contenant la réponse sera nul.

Par exemple, prenons la mutation `dossierAccepter`. Si une erreur se produit lors de l’exécution de cette mutation, la réponse peut ressembler à :

```json
{
  "data": {
    "dossierAccepter": {
      "errors": [
        {
          "message": "Les informations du SIRET du dossier ne sont pas complètes. Veuillez réessayer plus tard."
        }
      ],
      "dossier": null
    }
  }
}
```

#### Avertissements (warnings)

Certaines mutations exposent également un champ `warnings`. Les avertissements peuvent être présents simultanément avec une réponse valide. Par exemple, une réponse à une mutation pourrait ressembler à :

```json
{
  "data": {
    "groupeInstructeurCreer": {
      "warnings": [
        {
          "message": "testyahoo.fr n’est pas une adresse email valide"
        }
      ],
      "groupeInstructeur": {
        "id": "c9d97ddb-7532-4791-9f83-0fdebb92b7eb",
      }
    }
  }
}
```

Ici, bien qu'un avertissement ait été renvoyé, la mutation a réussi et des données valides ont été renvoyées.



{% hint style="info" %}
Pour être informé des évolutions de l’API, vous pouvez vous [inscrire à notre newsletter dédiée](https://7b97debb.sibforms.com/serve/MUIEAOpYNmRDuQF5Ib0fqC2aNgJW1j\_5EkA8AUkO6UBxCpofKKXPFNSiCInma2JfRsXGchOns5RZ0UWYVbC-8utoVRMT1wK\_1JRH7o0tbP-DQ07YtTsmkotgdSobF2n4MemAagXAO1mhB1vx3vqxijaSUYKnu-kg6O-1\_l7WcTNLdgHoQM1EfkJjaVpc3TPvaOYgj6Uk35dqi86-)
{% endhint %}
