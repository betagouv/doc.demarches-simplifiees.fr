---
description: >-
  Grâce à l'API de préremplissage, il est possible de préremplir un dossier pour
  une démarche donnée.
---

# API de préremplissage

Vous disposez de données sur vos usager·ères et vous souhaitez les utiliser pour **préremplir** un dossier ?

## Présentation

Demarches-simplifiees.fr propose une **API de préremplissage.** Pour une démarche donnée, elle permet de préremplir un dossier, avec les données dont vous disposez déjà.

Le dossier est créé en **brouillon** et vous pouvez y diriger votre usager·ère afin qu'iel s'authentifie, poursuive son remplissage, et le dépose auprès de l'administration.

Cette API est REST et ne nécessite pas d'authentification.

## Clés et valeurs

Le principe du préremplissage d'un dossier est le suivant : vous fournissez à l'API un couple clé / valeur pour chaque champ de la démarche à préremplir.

La **clé** est un identifiant en base 64. Elle identifie le champ de façon unique. Elle est stable et ne change pas dans le temps. Elle ressemble à une chaîne de cette forme : "champ\_Q2hhbXAtMTx0MjM2OX==".

La **valeur** est la donnée à renseigner dans le champ du formulaire. Selon le type de champ, cette valeur peut être contrainte. Par exemple, il est impossible de fournir la valeur "toto" pour un champ de type entier.

## Préremplissage en GET (par URL)

L'API de préremplissage met à votre disposition une URL, que vous **pouvez communiquer à votre usager·ère**. Cette URL crée un dossier prérempli en brouillon. Elle permet à l'usager·ère de s'authentifier, puis d'accéder au dossier afin de poursuivre son remplissage et enfin de le déposer.

L'URL prend la forme suivante :&#x20;

{% code overflow="wrap" %}
```
https://www.demarches-simplifiees.fr/commencer/<nom-demarche>?<cle1>=<valeur1>&<cle2>=<valeur2>
```
{% endcode %}

Comme il s'agit d'une URL, celle-ci est **limitée en longueur** (à 2000 caractères environ, selon les navigateurs). Le préremplissage par URL convient donc uniquement si vous avez peu de champs à préremplir, et / ou qu'il s'agit de champs dont la valeur est courte. Dans le cas contraire, il est préférable d'effectuer un [préremplissage en POST](api-de-preremplissage.md#preremplissage-en-post).

## Préremplissage en POST

L'API de préremplissage permet la création d'un dossier prérempli avec un appel en POST. Cette approche n'impose pas de limite sur la quantité ou la longueur des champs à préremplir.

### Requête

La requête doit être adressé en POST à l'URL `/api/public/v1/demarches/<id-demarche>/dossiers`. Elle prend donc en paramètre de route l'ID de la démarche.

Il est ensuite nécessaire d'indiquer :&#x20;

* le content type `application/json` dans les headers de la requête
* la liste de couples clé / valeur, au format JSON, dans le body de la requête

Une requête complète prend donc la forme suivante :&#x20;

```shell
curl --request POST 'https://demarches-simplifiees.fr/api/public/v1/demarches/<id>/dossiers' \
     --header 'Content-Type: application/json' \
     --data '{"cle1": "valeur1", "cle2": "valeur2"}'
```

### Réponse

L'API répond en JSON. La réponse contient l'URL du dossier vers laquelle **diriger l'usager·ère**. Iel peut alors s'authentifier, poursuivre le remplissage et déposer le dossier auprès de l'administration.

La réponse prend la forme suivante :&#x20;

```json
{
  "dossier_url": "https://demarches-simplifiees.fr/commencer/<nom-demarche>?prefill_token=<token de préremplissage>",
  "dossier_id": "<ID du dossier en base 64>",
  "dossier_number": <ID du dossier en tant qu'entier>
}
```

Au moment de la réponse, le dossier est orphelin. Il est rattaché à l'usager·ère après son authentification.

## Environnement

Demarches-simplifiees.fr ne propose pas d'environnement de test, intégration, préproduction ou sandbox, sur lequel réaliser votre intégration.

À la place, vous pouvez travailler sans risque directement sur la production (https://demarches-simplifiees.fr). En effet, au cours de l'intégration, vous allez créer des dossiers en brouillon, et ceux-ci :&#x20;

* **sont supprimés** périodiquement,
* **ne sont pas soumis à l'administration** tant qu'ils ne sont pas récupérés, complétés et déposés par un·e usager·ère,
* **sont invisibles** pour l'administration concernée tant qu'ils ne sont pas soumis.

Si vous le souhaitez, vous pouvez également prendre possession de ces dossiers en vous authentifiant, comme le ferait l'usager·ère, afin de les supprimer manuellement depuis la page [https://www.demarches-simplifiees.fr/dossiers](https://www.demarches-simplifiees.fr/dossiers).

## Démarrage rapide

Vous connaissez le nom de la démarche ? Alors rendez-vous directement sur la page `/preremplir/<nom-demarche>`.

Par exemple, si votre démarche est `soutien-experimentation-lycee`, alors ouvrez la page :&#x20;

{% hint style="success" %}
[https://www.demarches-simplifiees.fr/preremplir/soutien-experimentation-lycee](https://www.demarches-simplifiees.fr/preremplir/soutien-experimentation-lycee)
{% endhint %}

Vous y trouverez :&#x20;

* les champs de la démarche, avec leur ID, leur type, leur description et les valeurs possibles pour le préremplissage
* l'URL pour le [préremplissage en GET](api-de-preremplissage.md#preremplissage-en-get-par-url)
* un exemple de requête et de réponse pour le [préremplissage en POST](api-de-preremplissage.md#preremplissage-en-post)

## Structure de la démarche

L'équipe travaille actuellement au développement d'un nouveau point de terminaison, dans l'API de préremplissage, qui donne la structure d'une démarche au format JSON. Ce point de terminaison sera bientôt disponible.

## Statistiques sur le traitement des dossiers

Pour une démarche donnée, vous pouvez obtenir des statistiques sur le traitement des dossiers, de deux manières différentes :&#x20;

* en HTML, en vous rendant avec un navigateur sur la page `/statistiques/<nom-demarche>`
* en JSON, en envoyant une requête en GET à l'URL `/api/public/v1/demarches/<id-demarche>/stats`

