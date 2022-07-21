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

Pour construire une requête et interpréter les réponses, consultez la [**documentation complète du schéma de l’API**](https://demarches-simplifiees-graphql.netlify.com/).

#### Éditeur de requêtes en ligne

Une fois authentifié en tant qu’administrateur disposant d’un token, vous pouvez également accéder à l’**éditeur de requêtes en ligne** (attention, ne confondez pas cette adresse avec celle de l’endpoint, qui ressemble) :

[https://www.demarches-simplifiees.fr/graphql](https://www.demarches-simplifiees.fr/graphql)

### **Vous êtes un organisme public, un intégrateur, une SSII ou un éditeur, et vous souhaitez tester l’API Graph QL**

Vous devez effectuer une démarche auprès des équipes DS et pour cela remplir une demande à l’adresse suivante.

{% embed url="https://www.demarches-simplifiees.fr/commencer/demande-d-adhesion-a-ds-pour-api" %}

Votre demande sera examinée, et vous recevrez un token de test, ainsi qu'un accès à un formulaire de test qui vous permettra d’effectuer des saisies de dossiers puis de tester les API.

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
  $after: String
) {
  demarche(number: $demarcheNumber) {
    id
    number
    title
    publishedRevision {
      ...RevisionFragment
    }
    groupeInstructeurs {
      id
      number
      label
      instructeurs {
        id
        email
      }
    }
    dossiers(state: $state, order: $order, after: $after) {
      nodes {
        ...DossierFragment
      }
    }
  }
}

query getGroupeInstructeur(
  $groupeInstructeurNumber: Int!
  $state: DossierState
  $order: Order
  $after: String
) {
  groupeInstructeur(number: $groupeInstructeurNumber) {
    id
    number
    label
    instructeurs {
      id
      email
    }
    dossiers(state: $state, order: $order, after: $after) {
      nodes {
        ...DossierFragment
      }
    }
  }
}

query getDossier($dossierNumber: Int!) {
  dossier(number: $dossierNumber) {
    ...DossierFragment
    demarche {
      ...DemarcheDescriptorFragment
    }
  }
}

query getDeletedDossiers($demarcheNumber: Int!, $order: Order, $after: String) {
  demarche(number: $demarcheNumber) {
    deletedDossiers(order: $order, after: $after) {
      nodes {
        ...DeletedDossierFragment
      }
    }
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
  instructeurs {
    email
  }
  groupeInstructeur {
    id
    number
    label
  }
  revision {
    ...RevisionFragment
  }
  traitements {
    state
    emailAgentTraitant
    dateTraitement
    motivation
  }
  champs {
    ...ChampFragment
    ...RootChampFragment
  }
  annotations {
    ...ChampFragment
    ...RootChampFragment
  }
  avis {
    ...AvisFragment
  }
  messages {
    ...MessageFragment
  }
  demandeur {
    ... on PersonnePhysique {
      civilite
      nom
      prenom
      dateDeNaissance
    }
    ...PersonneMoraleFragment
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
  champDescriptors {
    ...ChampDescriptorFragment
    champDescriptors {
      ...ChampDescriptorFragment
    }
  }
  annotationDescriptors {
    ...ChampDescriptorFragment
    champDescriptors {
      ...ChampDescriptorFragment
    }
  }
}

fragment ChampDescriptorFragment on ChampDescriptor {
  id
  type
  label
  description
  required
  options
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
  attachment {
    ...FileFragment
  }
}

fragment MessageFragment on Message {
  id
  email
  body
  createdAt
  attachment {
    ...FileFragment
  }
}

fragment GeoAreaFragment on GeoArea {
  id
  source
  description
  geometry {
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
    champs {
      ...ChampFragment
    }
  }
  ... on SiretChamp {
    etablissement {
      ...PersonneMoraleFragment
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
      state
      usager {
        email
      }
    }
  }
}

fragment ChampFragment on Champ {
  id
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
    file {
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
  byteSizeBigInt
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

```

#### Variables

```graphql
# Tout les dossiers d’une démarche
{
  "query": <query>,
  "operationName": "getDemarche",
  "variables": {
    "demarcheNumber": 1234
  }
}

# Tout les dossiers "en_instruction" d’une démarche
{
  "query": <query>,
  "operationName": "getDemarche",
  "variables": {
    "demarcheNumber": 1234,
    "state": "en_instruction"
  }
}

# Tout les dossiers supprimés d’une démarche
{
  "query": <query>,
  "operationName": "getDeletedDossiers",
  "variables": {
    "demarcheNumber": 1234
  }
}

# Tout les dossiers d’un groupe instructeur
{
  "query": <query>,
  "operationName": "getGroupeInstructeur",
  "variables": {
    "groupeInstructeurNumber": 1234
  }
}

# Un dossier par son numero
{
  "query": <query>,
  "operationName": "getDossier",
  "variables": {
    "dossierNumber": 1234
  }
}
```

#### Pagination

La pagination sur l’API GraphQL se fait par "curseur". Ça veut dire que pour récupérer la prochaine page il faut passer à l’API le "curseur" de la fin de la page précédente.

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

Dans le résultat obtenu il faut lire la valeur du "curseur" dans le champ `demarche.dossiers.pageInfo.endCursor` . On peut passer alors ce "curseur" comme argument `after` dans la prochaine query. Et ainsi de suite jusqu’à ce que le champ `demarche.dossiers.pageInfo.hasNextPage` soit égale à `false`.

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

**Attention !** Les identifiants utilisés dans les variables `input` des mutations sont les identifiants « technique » que vous devez récupérer dans les réponses des requêtes précédentes. En particulier, l'identifiant du dossier est différent du « numéro de dossier » :

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

Si un identifiant d’instructeur est demandé, il s’agit d’identifier l’instructeur qui sera enregistré dans le système comme responsable de l’opération. Pour récupérer la liste des instructeurs avec leurs identifiants,  vous avez plusieurs possibilités.

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

Vous pouvez récupérer tous les instructeurs d'un « groupe instructeur » :

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

Pour plus d’informations sur le format des requêtes et des réponses, consultez la [documentation complète du schéma de l’API](https://demarches-simplifiees-graphql.netlify.com/).



{% hint style="info" %}
Pour être informé des évolutions de l’API, vous pouvez vous [inscrire à notre newsletter dédiée](https://7b97debb.sibforms.com/serve/MUIEAOpYNmRDuQF5Ib0fqC2aNgJW1j\_5EkA8AUkO6UBxCpofKKXPFNSiCInma2JfRsXGchOns5RZ0UWYVbC-8utoVRMT1wK\_1JRH7o0tbP-DQ07YtTsmkotgdSobF2n4MemAagXAO1mhB1vx3vqxijaSUYKnu-kg6O-1\_l7WcTNLdgHoQM1EfkJjaVpc3TPvaOYgj6Uk35dqi86-)
{% endhint %}

