# Gestion des Erreurs

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

