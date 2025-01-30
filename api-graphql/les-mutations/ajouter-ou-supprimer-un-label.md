# Ajouter ou supprimer un label

Les instructeurs peuvent ajouter (ou modifier) des labels à un dossier, qui agissent comme une sorte d’étiquette personnalisée. La liste des labels disponibles est au préalable définie dans l'interface l'administrateur (pas encore par API), ainsi que l’ordre dans lequel ils sont retournés.

Par défaut 5 labels sont créés : _Urgent, À examiner, À relancer, Complet, À signer_. Laissez libre court à votre imagination pour organiser l’instruction: _Prêt pour la commission, Validé par la hiérarchie_, …

L’API permet d’associer ou supprimer un label à un dossier. Récupérez d'abord [la liste des identifiants de labels disponibles](../les-queries/getdemarche.md#query-pour-recuperer-les-informations-dune-demarche) sur la démarche.

Il n’y a pas de limite sur le nombre de labels par dossier, mais dans l’interface Démarches Simplifiées il n’est généralement pas pratique d’en afficher plus de 5 ou 6 par dossier.

#### Query

### Ajouter un label

```graphql
mutation dossierAjouterLabel($input: DossierAjouterLabelInput!) {
	dossierAjouterLabel(input: $input) {
		dossier {
			id
			labels {
				id
				name
				color
			}
		}
		label {
			name
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
  "operationName": "dossierAjouterLabel",
  "variables": {
    "input": {
      "dossierId": "UHJvY4VkdXKlLTI5NTgw",
      "labelId": "TGFiZWwtMzA0MzQw"
    }
  }
}
```



{% hint style="info" %}
L’attribut `color` est un `string` parmi quelques noms de couleurs disponibles dans le Design Système de l'État. Il sera peut-être inutile dans le contexte de votre utilisation de l'API.
{% endhint %}



### Supprimer un label

#### Query

```graphql
mutation dossierSupprimerLabel($input: DossierSupprimerLabelInput!) {
	dossierSupprimerLabel(input: $input) {
		dossier {
			id
			labels { # Le ou les labels restant sur le dossier
				id
				name
				color
			}
		}
		label {
			name # Le label supprimé
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
  "operationName": "dossierSupprimerLabel",
  "variables": {
    "input": {
      "dossierId": "UHJvY4VkdXKlLTI5NTgw",
      "labelId": "TGFiZWwtMzA0MzQw"
    }
  }
}
```

