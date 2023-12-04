# Envoyer un message

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
