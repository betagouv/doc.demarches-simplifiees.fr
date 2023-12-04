# Problèmes fréquents

## J'arrive pas à requeter l'API avec mon token

Si notre API vous répond :

{% code overflow="wrap" %}
```json
{
  "data": null,
  "errors": [
    {
      "message": "An object of type Demarche was hidden due to permissions",
      "locations": [
        {
          "line": 33,
          "column": 5
        }
      ],
      "path": [
        "demarche"
      ],
      "extensions": {
        "code": "unauthorized"
      }
    }
  ]
}
```
{% endcode %}

Vous devez vérifier :&#x20;

* que vous envoyez bien le token
* que le format envoyé est bien dans le header et respecte la synthax (--header 'Authorization: Bearer \<votre\_token>' )
* que le format demandé est bien de type (--header 'Content-Type: application/json')
* que le token à accès à la démarche (vous pouvez uniquement requeter les données que vous maitrisez)
