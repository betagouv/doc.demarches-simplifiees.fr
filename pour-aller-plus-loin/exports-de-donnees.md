# Exports de données

## Export manuel

Il est possible d'exporter manuellement un tableau contenant la _majeure partie_ des informations relatives aux dossiers déposés sur une procédure donnée.

Pour cela, dans l'interface instructeur, cliquer sur la procédure concernée puis sur le bouton "Téléchargez tous les dossiers" en haut à droite et choisir le format d'export souhaité (.csv, .xls, .ods).

![](<../.gitbook/assets/Capture d’écran 2019-02-11 à 14.23.09.png>)

Il n'est pas possible d'effectuer cette opération dans l'interface administrateur, mais les administrateurs peuvent le faire en se nommant instructeur sur la procédure.

Le tableau exporté est composé d'une ligne par dossier, chaque ligne comportant les informations suivantes :

* métadonnées : numéro de dossier et dates ;
* données saisies par le demandeur : identité du demandeur et  champs du formulaire ;
* données d'instruction : emails des usagers suivant le dossier et annotations privées.

En revanche, les informations suivantes ne figurent pas dans le tableau :

* liens vers les pièces jointes ;
* avis ;
* messages échangés;

#### Export sous la forme d'un ZIP

![Une nouvelle option permet d'exporter l'ensemble des dossiers sous forme d'un zip](<../.gitbook/assets/Capture d’écran 2021-05-03 à 16.36.06.png>)

Une nouvelle option a été mis en place qui permet d'exporte non pas sous forme d'un tableau, mais sous forme de fichier ( un fichier par dossier) contenant les données, les PJ, et les métadonnées.

**Tout est alors prêt pour faire de l'archivage !**

#### Métadonnées

Les métadonnées du dossier comprennent les informations suivantes:&#x20;

* id: numéro du dossier
* created\_at: date de création du dossier
* updated\_at: date de la dernière modification du dossier
* archived: informe si le dossier est archive (_true_) ou non (_false_)
* email: email de l'usager
* state: correspond à l'état du dossier
  * initiated: en construction
  * received: en instruction
  * closed: accepté
  * refused: refusé
  * without\_continuation: classé sans suite
* initiated\_at: date du dépôt de dossier
* received\_at: date du passage en instruction
* processed\_at: date de décision du dossier
* motivation: motivation de la décision&#x20;
* email\_insructeurs: email de l'instructeur qui a donné la décision pour le dossier

![](../.gitbook/assets/CaptureExport2.PNG)

## Export par API

demarches-simplifiees.fr est fourni avec une API qui permet d'exporter de manière automatique la _totalité_ des informations relatives aux dossiers déposés sur une procédure donnée.

Cette API ne permet en revanche pas d'entrer des données dans l'application ou de commander des opérations.

Pour plus d'information, voir la [documentation de l'API](graphql.md).
