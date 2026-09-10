# Premiers pas : tester l'API avec un client GraphQL

Avant d'écrire la moindre ligne de code, le plus simple pour découvrir l'API est d'utiliser un client GraphQL sur votre poste. Ces outils permettent d'explorer le schéma, d'écrire des requêtes avec autocomplétion et de lire les réponses, exactement comme le faisait l'ancien éditeur en ligne.

{% hint style="warning" %}
L'éditeur de requêtes en ligne (`/graphql`) a été retiré pour des raisons de sécurité : il authentifiait les requêtes avec la session du navigateur. Toute requête à l'API passe désormais obligatoirement par un [jeton d'authentification](jeton-dauthentification/).
{% endhint %}

## Ce dont vous avez besoin

* un compte administrateur sur **demarche.numerique.gouv.fr** ;
* une **démarche de test** publiée (un administrateur ne peut requêter que ses propres démarches) ;
* un [jeton d'authentification](jeton-dauthentification/) autorisé sur cette démarche.

{% hint style="info" %}
Pour explorer l'API, créez un jeton **en lecture seule**, restreint à votre démarche de test. Si le jeton est en mode « détection automatique » d'adresse IP, la première adresse qui l'utilise devient la seule autorisée : faites votre premier appel depuis le poste sur lequel vous allez travailler.
{% endhint %}

## Quel client choisir ?

Tous ces outils sont gratuits pour cet usage et fonctionnent sur Windows, macOS et Linux.

| Client                                             | Points forts                                                                                                           |
| -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| [Altair GraphQL Client](https://altairgraphql.dev) | Dédié à GraphQL, très léger. Existe en application de bureau et en extension de navigateur. Le bon choix pour débuter. |
| [Bruno](https://www.usebruno.com)                  | Libre, fonctionne hors ligne, enregistre les collections dans des fichiers texte faciles à versionner.                 |
| [Insomnia](https://insomnia.rest)                  | Client HTTP complet avec un mode GraphQL. Un compte est demandé au premier lancement.                                  |
| [Postman](https://www.postman.com)                 | Très répandu dans les équipes qui l'utilisent déjà pour d'autres API. Un compte est demandé.                           |

Si vous préférez la ligne de commande, la page [jeton d'authentification](jeton-dauthentification/) contient un exemple complet avec `curl`.

## Configurer le client

La configuration est la même quel que soit l'outil :

1. **Adresse** : `https://demarche.numerique.gouv.fr/api/v2/graphql`, en méthode **POST**.
2.  **En-têtes** :

    | Nom             | Valeur                 |
    | --------------- | ---------------------- |
    | `Authorization` | `Bearer <votre_jeton>` |
    | `Content-Type`  | `application/json`     |
3. **Schéma** : une fois l'en-tête `Authorization` renseigné, demandez au client de charger le schéma (bouton « Reload docs », « Fetch schema » ou équivalent). Il effectue une requête d'introspection, et vous disposez alors de l'autocomplétion et de la documentation de chaque champ directement dans l'éditeur.

{% hint style="warning" %}
Le chargement du schéma échoue avec une erreur `forbidden` si l'en-tête `Authorization` n'est pas encore renseigné : l'introspection nécessite un jeton.
{% endhint %}

## Première requête

Collez la requête suivante dans l'éditeur, puis les variables dans le panneau dédié (souvent nommé « Variables » ou « Query variables »), en remplaçant `12345` par le numéro de votre démarche de test :

```graphql
query getDemarche($demarcheNumber: Int!) {
  demarche(number: $demarcheNumber) {
    id
    title
    dossiers(first: 10) {
      nodes {
        id
        number
        state
      }
    }
  }
}
```

```json
{ "demarcheNumber": 12345 }
```

La réponse contient le titre de votre démarche et ses dix premiers dossiers. Si vous obtenez une erreur, consultez [Problèmes fréquents](jeton-dauthentification/problemes-frequents.md).

{% hint style="danger" %}
Ces clients **ne sont pas une sandbox** : les requêtes sont exécutées sur les données de production. Si vous testez des mutations, faites-le uniquement sur une démarche de test, jamais sur une démarche contenant de vrais dossiers.
{% endhint %}

{% hint style="info" %}
Pour construire des requêtes plus complètes et interpréter les réponses, consultez la [documentation complète du schéma de l'API](https://demarche.numerique.gouv.fr/graphql/schema/).
{% endhint %}
