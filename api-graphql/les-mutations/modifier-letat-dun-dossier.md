# Modifier l'état d'un dossier

## Modifier l’état ou les informations d’un dossier

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
