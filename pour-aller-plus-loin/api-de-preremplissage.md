---
description: >-
  Grâce à l'API de préremplissage, il est possible de préremplir un dossier pour
  une démarche donnée.
---

# API de préremplissage

Vous disposez de données sur vos usager·ères et vous souhaitez les utiliser pour **préremplir** un dossier ?

## Présentation

demarches-simplifiees.fr propose une **API de préremplissage.** Pour une démarche donnée, elle permet de préremplir un dossier, avec les données dont vous disposez déjà.

{% hint style="info" %}
Pour accéder à cette fonctionnalité, la démarche doit :&#x20;

* être soit en brouillon, soit publiée (donc la foncionnalité n'est pas disponible pour les démarches closes)
* être en opendata (configurable depuis la tuile "Présentation")
{% endhint %}

Le dossier est créé en **brouillon** et vous pouvez y diriger votre usager·ère afin qu'iel s'authentifie, poursuive son remplissage, et le dépose auprès de l'administration.

Cette API est REST et ne nécessite pas d'authentification.

{% embed url="https://vimeo.com/792561801" %}
Présentation de l'API de préremplissage
{% endembed %}

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

demarches-simplifiees.fr ne propose pas d'environnement de test, intégration, préproduction ou sandbox, sur lequel réaliser votre intégration.

À la place, vous pouvez travailler sans risque directement sur la production (https://demarches-simplifiees.fr). En effet, au cours de l'intégration, vous allez créer des dossiers en brouillon, et ceux-ci :&#x20;

* **sont supprimés** périodiquement,
* **ne sont pas soumis à l'administration** tant qu'ils ne sont pas récupérés, complétés et déposés par un·e usager·ère,
* **sont invisibles** pour l'administration concernée tant qu'ils ne sont pas soumis.

Si vous le souhaitez, vous pouvez également prendre possession de ces dossiers en vous authentifiant, comme le ferait l'usager·ère, afin de les supprimer manuellement depuis la page [https://www.demarches-simplifiees.fr/dossiers](https://www.demarches-simplifiees.fr/dossiers).

## Démarrage rapide

Vous connaissez le nom de la démarche ? Alors rendez-vous directement sur la page `/preremplir/<nom-demarche>`.

Par exemple, si votre démarche est `une-demarche-a-preremplir`, alors ouvrez la page :&#x20;

{% hint style="success" %}
[https://www.demarches-simplifiees.fr/preremplir/une-demarche-a-preremplir](https://www.demarches-simplifiees.fr/preremplir/une-demarche-a-preremplir)
{% endhint %}

Vous y trouverez :&#x20;

* les champs de la démarche, avec leur ID, leur type, leur description et les valeurs possibles pour le préremplissage
* l'URL pour le [préremplissage en GET](api-de-preremplissage.md#preremplissage-en-get-par-url)
* un exemple de requête et de réponse pour le [préremplissage en POST](api-de-preremplissage.md#preremplissage-en-post)

## Structure de la démarche

### Requête

Un point de terminaison vous donnant accès à une description en JSON du schéma de la démarche est à votre disposition. La requête doit être adressée en GET à l'URLl `/preremplir/<nom-demarche>/schema`.

Par exemple, si votre démarche est `une-demarche-a-preremplir`, la requête doit être adressée à :&#x20;

{% hint style="success" %}
[https://www.demarches-simplifiees.fr/preremplir/une-demarche-a-preremplir/schema](https://www.demarches-simplifiees.fr/preremplir/une-demarche-a-preremplir/schema)
{% endhint %}

L'API répond en JSON. La réponse contient des informations génériques sur la démarche ainsi que l’identifiant stable, le titre, la description de chaque champ de la démarche. Elle indique aussi les champs requis ou non.

### Réponse

La réponse prend la forme suivante :&#x20;

```json
{
  "id": "ID_démarche",
  "number": "Numéro_démarche",
  "title": "Titre_démarche",
  "description": "Description_démarche",
  "state": "publiee",
  "declarative": null,
  "dateCreation": "2023-01-16T11:10:48+01:00",
  "datePublication": "2023-01-16T11:18:32+01:00",
  "dateDerniereModification": "2023-01-16T16:55:21+01:00",
  "dateDepublication": null,
  "dateFermeture": null,
  "notice": null,
  "deliberation": null,
  "cadreJuridiqueUrl": "http://cadredemarche.fr/cadre",
  "revision": {
    "id": "ID_Revision",
    "datePublication": "2023-01-16T16:55:21+01:00",
    "champDescriptors": [
      {
        "__typename": "TextChampDescriptor",
        "id": "Stable_ID_Démarche1",
        "label": "Titre_Démarche1",
        "description": "Description_Démarche1",
        "required": true
      },
      {
        "__typename": "TextareaChampDescriptor",
        "id": "Stable_ID_Démarche2",
        "label": "Titre_Démarche2",
        "description": "Description_Démarche2",
        "required": false
      }
    ]
  }
}
```

## Statistiques sur le traitement des dossiers

Pour une démarche donnée, vous pouvez obtenir des statistiques sur le traitement des dossiers, de deux manières différentes :&#x20;

* en HTML, en vous rendant avec un navigateur sur la page `/statistiques/<nom-demarche>`
* en JSON, en envoyant une requête en GET à l'URL `/api/public/v1/demarches/<id-demarche>/stats`

