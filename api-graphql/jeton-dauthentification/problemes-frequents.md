# Problèmes fréquents

## Je n'arrive pas à requêter l'API avec mon jeton (token)

Si l'API vous répond :

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

Vous devez vérifier que :&#x20;

* vous envoyez bien le token (sans espace à l'intérieur du jeton, probablement avec des `==` à la fin)
* le token est envoyé par un header et respecte la syntaxe `--header 'Authorization: Bearer <votre_token>'`
* le format demandé est de type `--header 'Content-Type: application/json'`
* vérifiez les guillements, apostrophes, les espaces, majuscules et miniscules
* le token a accès à la démarche (vous pouvez uniquement requêter les données des démarches de votre compte administrateur, et pour lesquelles le jeton a été autorisé)
* si vous avez perdu votre jeton, vous pouvez le révoquer et en créer un nouveau depuis la page _Voir mon profil_ sur demarches-simplifiees.fr

Consultez également [notre documentation sur les erreurs](../gestion-des-erreurs.md) si vous obtenez une autre erreur.
