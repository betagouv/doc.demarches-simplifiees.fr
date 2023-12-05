---
description: Comment fonctionne notre système d'authentification ?
---

# Jeton d'authentification

## Comment accéder aux jetons ?

Connectez-vous avec votre compte administrateur et cliquez sur _Voir mon profil_

<figure><img src="../../.gitbook/assets/Screenshot 2023-12-01 at 11.03.59 AM.png" alt="" width="344"><figcaption></figcaption></figure>

##

## Comment créer un jeton ?&#x20;

Cliquez sur le bouton _Créer et afficher un nouveau jeton_.&#x20;

<figure><img src="../../.gitbook/assets/Screenshot 2023-12-01 at 11.04.14 AM.png" alt=""><figcaption><p>comment crééer un jeton</p></figcaption></figure>

{% hint style="warning" %}
Attention, le jeton n'est affiché qu'à la création !&#x20;
{% endhint %}

## Comment fonctionnent les jetons ?

Les jetons peuvent être configurés pour :&#x20;

### Accéder en lecture et/ou écriture à **toutes les démarches** de l'administrateur ayant généré le jeton

<figure><img src="../../.gitbook/assets/Screenshot 2023-12-01 at 11.04.51 AM.png" alt=""><figcaption><p>exemple d'un jeton qui donne accès à toutes les démarches de l'administrateur</p></figcaption></figure>

### Accéder en lecture et/ou écriture uniquement à une liste fermée des démarches de l'administrateur ayant généré le jeton&#x20;

<figure><img src="../../.gitbook/assets/Screenshot 2023-12-01 at 11.05.24 AM.png" alt=""><figcaption><p>exemple d'un jeton qui donne accès à certaines démarches de l'administrateur</p></figcaption></figure>

{% hint style="info" %}
Pour des raisons de sécurité, nous recommandons l'usage de ce type de jeton qui restreint l'accès aux démarches nécessaires.&#x20;
{% endhint %}

## Comment utiliser le jeton ?

Ce jeton doit être fourni dans l’en-tête HTTP `Authorization` de la requête.

&#x20;`Authorization: Bearer token=valeur_du_jeton`.

{% swagger baseUrl="https://www.demarches-simplifiees.fr" path="/api/v2/graphql" method="post" summary="GraphQL" %}
{% swagger-description %}
Le point d’entrée de l’API GraphQL.
{% endswagger-description %}

{% swagger-parameter in="header" name="Content-Type" type="string" required="true" %}
application/json
{% endswagger-parameter %}

{% swagger-parameter in="header" name="Authorization" type="string" required="true" %}
Le jeton de l’administrateur
{% endswagger-parameter %}

{% swagger-parameter in="body" name="query" type="string" required="true" %}
La requête GraphQL
{% endswagger-parameter %}

{% swagger-parameter in="body" name="variables" type="object" required="true" %}
Les variables de la requête
{% endswagger-parameter %}

{% swagger-response status="200" description="Réponse GraphQL" %}
```json
{
    "data": {
    	"demarche": {
    	    "id": "UHJvY2VkdXJlLTYyNTE1",
	    "dossiers": {
	    	"nodes": [
		    {
		        "id": "RG9zc2llci05Mzk5MTkw",
		        "demandeur": {
			    "civilite": "M",
			    "nom": "Franklin",
			    "prenom": "Benjamin"
			}
		    },
	            {
			"id": "RG9zc2llci05NjQzMDIz",
			"demandeur": {
			    "civilite": "Mme",
		            "nom": "Curie",
			    "prenom": "Marie"
			}
		    },
		    {
			"id": "RG9zc2llci0xMzI2Njg1NQ==",
                        "demandeur": {
                            "civilite": "M",
			    "nom": "Pasteur",
			    "prenom": "Louis"
			}
		    }
		]
	    }
        }
    }
}
```
{% endswagger-response %}
{% endswagger %}

### Exemple de requête

#### Pour tester que tout fonctionne bien

Pour tester l’API, le plus simple est d’effectuer une requête [curl](https://fr.wikipedia.org/wiki/CURL) telle que ci dessous. Le principe est le même avec un autre client HTTP : remplacez **votre\_token** et **votre\_numero\_de\_demarche** par les valeurs souhaitez, et n’oubliez pas de préciser le **content-type** qui est toujours `application/json` **:**

```shell
curl 'https://www.demarches-simplifiees.fr/api/v2/graphql' \
    --header 'Authorization: Bearer <votre_token>' \
    --header 'Content-Type: application/json' \
    --data '{ "query": "query getDemarche($demarcheNumber: Int!) { demarche(number: $demarcheNumber) { id dossiers { nodes { id demandeur { ... on PersonnePhysique { civilite nom prenom } ... on PersonneMorale { siret } } } } } }", "variables": { "demarcheNumber": <votre_numero_de_demarche> } }'
```

Vous devriez alors obtenir des informations en sortie. S’il y a des dossiers dans votre démarche, cette requête vous donne les noms des demandeurs. Si la démarche s’adresse à des entreprises, vous aurez le numéro SIRET des demandeurs.

{% hint style="info" %}
Exemple si mon jeton est `A9Knc34tP==` avec le numéro de démarche `12345`:&#x20;
{% endhint %}

```sh
curl 'https://www.demarches-simplifiees.fr/api/v2/graphql' \
    --header 'Authorization: Bearer A9Knc34tP==' \
    --header 'Content-Type: application/json' \
    --data '{ "query": "query getDemarche($demarcheNumber: Int!) { demarche(number: $demarcheNumber) { id dossiers { nodes { id demandeur { ... on PersonnePhysique { civilite nom prenom } ... on PersonneMorale { siret } } } } } }", "variables": { "demarcheNumber": 12345 } }'
```

Voici la réponse de l'API pour cette requête :

```json
{
    "data": {
    	"demarche": {
    	    "id": "UHJvY2VkdXJlLTYyNTE1",
	    "dossiers": {
	    	"nodes": [
		    {
		        "id": "RG9zc2llci05Mzk5MTkw",
		        "demandeur": {
			    "civilite": "M",
			    "nom": "Franklin",
			    "prenom": "Benjamin"
			}
		    },
	            {
			"id": "RG9zc2llci05NjQzMDIz",
			"demandeur": {
			    "civilite": "Mme",
		            "nom": "Curie",
			    "prenom": "Marie"
			}
		    },
		    {
			"id": "RG9zc2llci0xMzI2Njg1NQ==",
                        "demandeur": {
                            "civilite": "M",
			    "nom": "Pasteur",
			    "prenom": "Louis"
			}
		    }
		]
	    }
        }
    }
}
```



{% hint style="warning" %}
Si vous obtenez une erreur, vérifiez que :&#x20;

* chaque paramètre `header` est correct
* il n'y a pas d'espace ou de guillemet en trop ou en moins&#x20;
* les majuscules et miniscules sont respectées
* l'url, le jeton et le numéro de démarche sont corrects
* lisez le message d'erreur et consultez nos pages [Problèmes fréquents d'authentification](problemes-frequents.md) et [la gestion d'erreur](../gestion-des-erreurs.md)
{% endhint %}
