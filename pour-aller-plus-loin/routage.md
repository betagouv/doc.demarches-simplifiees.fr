---
description: >-
  Il est possible de répartir les dossiers entre différents groupes
  d'instructeurs, en se basant sur les informations saisies par les
  utilisateurs.
---

# Routage des dossiers

## Version vidéo

{% embed url="https://www.loom.com/share/59b913777264477b8a603a77d8045fc7?sid=92477667-8b77-4b32-bf39-0516db9c82d4" %}

## 1. **Présentation**&#x20;

Pour faciliter le travail des instructeurs, il est possible de configurer à l'avance la répartition automatique des dossiers en activant le « routage ».

Cette fonctionnalité permet d'affecter les dossiers uniquement à certains instructeurs, en fonction du contenu du formulaire rempli par l'usager.

Ce système est particulièrement adapté aux démarches nationales instruites localement.

## **2. Configuration du routage**&#x20;

* Plusieurs types de champ du formulaire permettent de router les dossiers, notamment « Choix simple », « Choix multiple », « Case à  cocher»,  « Nombre entier »,  « Nombre décimal », « Communes »,  « EPCI », « Départements » et « Régions ».
* Dans la page « Configuration des champs », créez un champ de l'un de ces types, par exemple « Choix simple ». Ce champ peut être déplacé au sein du formulaire. Cliquez ensuite sur « Continuer ».

<figure><img src="../.gitbook/assets/Screenshot from 2023-09-21 10-56-17.png" alt=""><figcaption></figcaption></figure>

* Cliquez sur la tuile « Instructeurs » de votre interface de configuration, puis sur « Options » dans le menu latéral, puis sur le bouton « Configurer le routage ».

<figure><img src="../.gitbook/assets/Screenshot from 2023-09-21 10-58-03.png" alt=""><figcaption></figcaption></figure>

* Deux options sont possibles : routage à partir d'un champ et routage avancé.

<figure><img src="../.gitbook/assets/Screenshot from 2023-09-21 10-58-32.png" alt=""><figcaption></figcaption></figure>

## **3.  Routage à partir d'un champ**

* Ce mode de configuration du routage vous permet de créer automatiquement les groupes d'instructeurs et les règles de routage à partir du contenu d'un champ.
* Une fois le mode « Routage à partir d'un champ » sélectionné, choisissez le champ du formulaire à partir duquel router les dossiers.

<figure><img src="../.gitbook/assets/Screenshot from 2023-09-21 10-59-13.png" alt=""><figcaption></figcaption></figure>

* Au clic sur « Créer les groupes », un groupe d'instructeurs et la règle de routage associée seront créés pour chaque option du champ.

<figure><img src="../.gitbook/assets/Screenshot from 2023-09-21 10-59-46.png" alt=""><figcaption></figcaption></figure>

* Dans le cas d'un routage à partir d'un champ de type  « Choix simple », le nombre de groupes créés correspond au nombre d'options du champ.
* Dans le cas d'un routage à partir d'un champ de type « Régions », 18 groupes sont créés (un par région).
* Dans le cas d'un routage à partir d'un champ de type « Départements », 110 groupes sont créés (un par département).
* Dans le cas d'un routage à partir d'un champ de type « Communes » ou « EPCI », 110 groupes (un par département) sont créés également. Ainsi, lors du dépôt du dossier, le code du département correspondant à la commune ou à l'EPCI choisi par l'usager permet de router le dossier vers le bon groupe d'instructeurs.

## 4. Routage avancé&#x20;

* Ce mode de configuration du routage crée deux groupes d'instructeurs. Ensuite, libre à vous de les renommer et de leur attribuer des règles de routage à partir du ou des champs « routables » de votre formulaire.

<figure><img src="../.gitbook/assets/Screenshot from 2023-09-21 11-00-49.png" alt=""><figcaption></figcaption></figure>

* Si un badge « Règle invalide » apparaît , c'est que la règle de routage du groupe doit être définie.
* Si deux groupes ont la même règle de routage, les dossiers seront envoyés au premier groupe de la liste par ordre d'affichage.

## 5. Gestion d'un groupe

* Dans la page d'un groupe d'instructeurs, il est possible de renommer le groupe, de changer ses règles de routage, de le désactiver ou encore de le supprimer.

<figure><img src="../.gitbook/assets/Screenshot from 2023-09-21 11-05-20.png" alt=""><figcaption></figcaption></figure>

* Par défaut, une règle de routage est construite de la façon suivante : « Router si le champ Votre ville **est égal à** Lyon ».
* Il est possible de changer l'opérateur pour que la règle devienne : « Router si le champ Votre ville **n'est pas** Lyon ».

## 6. Ajout de groupes

* Une fois le routage mis en place, vous pouvez créer de nouveaux groupes d'instructeurs. Pour leur associer de nouvelles règles de routage, il faut ajouter des options dans le champ utilisé pour le routage, depuis la page « Champs du formulaire ». Une fois les modifications du formulaire publiées, vous pouvez sélectionner la règle de routage dans la page du groupe d'instructeurs.&#x20;

## 7. Routage et conditionnel

* Il est possible de conditionner l'affichage d'un champ dédié au routage (voir l'exemple ci-dessous). Si le champ est affiché, le dossier sera routé en fonction du choix de l'usager. Si le champ n'est pas affiché, le dossier sera routé vers le groupe d'instructeurs par défaut (à définir en bas de la page « Gestion des groupes »)

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## 8. Supprimer le routage

* Pour supprimer le routage, vous pouvez soit supprimer tous les groupes d'instructeurs, soit aller dans la page « Options » et cliquer sur « Supprimer le routage ».

## 9. Le routage en tant qu'instructeur&#x20;

* En tant qu'instructeur d'une démarche utilisant le routage, l'onglet « instructeurs » apparaît en dessous du titre de la démarche.

![](<../.gitbook/assets/Screenshot 2020-01-31 at 10.56.48.png>)

* Ce menu permet à l'instructeur de voir l'ensemble des groupes auxquels il a été affecté.&#x20;

![](<../.gitbook/assets/Screenshot 2020-01-31 at 11.28.49.png>)

* Pour chaque groupe d'instructeurs, il peut visualiser l'ensemble des instructeurs qui y sont affectés.

![](<../.gitbook/assets/Screenshot 2020-01-31 at 11.29.53.png>)

* Tous les instructeurs d'un groupe ont la possibilité d'ajouter un nouvel instructeur à ce groupe ou d'en retirer un. La gestion des groupes peut donc se faire en autonomie et ne nécessite pas l'intervention de l'administrateur.&#x20;

## 10. Réaffectation d'un dossier

* Si l'usager se trompe en sélectionnant une option du champ dédié au routage, son dossier est routé vers le mauvais groupe d'instructeurs. Cependant, tant que le dossier est en construction, l'usager peut modifier lui-même la valeur du champ pour que son dossier soit réaffecté au bon groupe d'instructeurs.
* En tant qu'instructeur, suite à une erreur de l'usager, il est aussi possible de réaffecter un dossier à un autre groupe d'instructeurs.&#x20;



<figure><img src="../.gitbook/assets/Screenshot from 2023-07-25 10-12-43.png" alt=""><figcaption></figcaption></figure>



