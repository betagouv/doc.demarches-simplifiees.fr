---
description: Liste de nos endpoints en écriture
---

# Les mutations

### L'API offre les endpoints suivant en écriture ([mutations](https://demarche.numerique.gouv.fr/graphql)) :

#### Dossier

Pour rappel, les usagers déposent les dossiers, les instructeurs les instruisent. Nous mettons donc a disposition toutes les mutations nécéssaires pour traiter un dossier dans tout son cycle de vie.

Pour rappel, le cycle de vie d'un dossier dans DS est le suivant : brouillon -> en construction -> en instruction -> accepté ou refusé ou classé sans suite.

Les mutations possibles pour un dossier sont les suivantes :

* Passer un dossier en instruction
* Accepter un dossier
* Refuser un dossier
* Classer un dossier sans suite
* Repasser un dossier en construction
* Repasser un dossier en instruction
* Archiver un dossier
* Ajouter un message à la messagerie du dossier / usager
* Changer un dossier de groupe d'instructeur
* Ajouter un label
* Retirer un label

#### Demarche :

Pour rappel, la démarche décrit le formulaire présenté aux usagers, la durée de conservation des données, les instructeurs et groupes d'instructeurs qui instruisent les dossiers déposé sur la démarche.

Les mutations possibles pour une démarche sont les suivantes ([voir le détail et des exemples](gerer-une-demarche.md)) :

* Publier une démarche
* Modifier les paramètres d'une démarche (titre, description, date limite de dépôt, etc.)
* Cloner une démarche (avec, en option, son service)

#### AnnotationPrivées :

* De modifier une annotation privée (de type checkbox, date, datetime, nombre entier et text)

#### GroupeInstructeur :

* Créer un groupe d'instructeur
* Ajouter un instructeur à un groupe d'instructeur
* Modifier un groupe d'instructeur
* Supprimer un instructeur d'un groupe d'instructeur

#### PieceJointe :

* Uploader un PJ pour la liée à un message envoyé a l'usager
