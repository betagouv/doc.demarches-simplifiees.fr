# Champs liés à API Particulier

Dans le cadre du programme Dites-le-nous une fois, l'État met à disposition des administrations le bouquet API Particulier permettant aux administrations de s’échanger les données des usagers. Fini les justificatifs papiers numérisés : les données remontent directement pour les usagers qui s'identifient via FranceConnect.

Lorsque les informations sont récupérées avec succès, l'usager n'a pas besoin de transmettre de justificatifs. Cette automatisation simplifie la démarche pour l'usager et limite les pièces justificatives à instruire. Les informations proviennent directement des organismes producteurs (CAF, MSA, CNOUS…), elles sont donc certifiées et actualisées.

### Les données accessibles via le bouquet API Particulier

Démarches Numérique intègre progressivement les données disponibles dans le bouquet d’API Particulier. Voici l'état actuel des données disponibles et à venir :

| API                                                 | Données                                                                                                                      | Disponibilité          |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ---------------------- |
| Quotient familial CAF & MSA                         | Quotient (valeur, période d'appilication, date de calcul), composition familial (allocataires, enfants) et adresse du foyer  | Disponible             |
| Statut étudiant boursier                            | Statut, échelon, période versement, établissement,  identité                                                                 | En cours d'intégration |
| Allocation aux adultes handicapés (AAH)             | Statut, date début de droit                                                                                                  | En cours d'intégration |
| Allocation d'éducation de l'enfant handicapé (AEEH) | Statut, date début de droit                                                                                                  | En cours d'intégration |
| Allocation de rentrée scolaire (ARS)                | Non défini                                                                                                                   | Prochainement          |

### Ajout d'un champ d'API Particulier par l'administrateur

**Prérequis**

L'utilisation de ces champs nécessite qu'un **jeton API Particulier** soit configuré sur la démarche. Voir en bas de page pour l'obtention du jeton.

<figure><img src="../.gitbook/assets/Capture d’écran 2026-07-06 à 17.55.29.png" alt=""><figcaption></figcaption></figure>

Depuis l'éditeur de formulaire :

1. Ajoutez un nouveau champ.
2. Sélectionnez le type souhaité.
3. Enregistrez vos modifications.

<figure><img src="../.gitbook/assets/Capture d’écran 2026-07-06 à 17.58.33.png" alt=""><figcaption><p>Exemple avec le champ quotient familial</p></figcaption></figure>

> **À noter : les** champs API Particulier ne sont pas personnalisables : libelle, description, champ de substitution.

### Fonctionnement pour l'usager

Pour un usager France connecté, les données sont récupérées automatiquement lorsque celui-ci commence un nouveau dossier.

Deux situations sont possibles :

1. **Les données sont récupérées avec succès** : les données sont présentées dans le dossier et aucun justificatif n'est demandé.

<figure><img src="../.gitbook/assets/Capture d’écran 2026-07-06 à 18.14.59.png" alt=""><figcaption><p>Exemple avec le champ quotient familial</p></figcaption></figure>

L'usager a ensuite la possibilité de confirmer ou non l'exactitude des données récupérées. Dans l'affirmative, aucune autre action n'est requise. Dans la négative, l'usager doit en revanche fournir un justificatif en pièce jointe.

<figure><img src="../.gitbook/assets/Capture d’écran 2026-07-06 à 18.19.31.png" alt=""><figcaption><p>Exemple avec le champ quotient familial</p></figcaption></figure>

2. **Les données ne peuvent pas être récupérées**, car :

* l'usager ne s'est pas connecté via FranceConnect ; ou
* l'usager s'est connecté via FranceConnect mais la récupération automatisée des données a échoué.

L'usager doit donc fournir un justificatif en pièce jointe :&#x20;

<figure><img src="../.gitbook/assets/Capture d’écran 2026-07-06 à 18.21.20.png" alt=""><figcaption><p>Exemple avec le champ quotient familial</p></figcaption></figure>

### Utilisation par les instructeurs

Les données automatiquement récupérées d'API Particulier sont présentées directement dans le dossier, et l'instructeur en est informé :&#x20;

<figure><img src="../.gitbook/assets/Capture d’écran 2026-07-06 à 18.39.58.png" alt=""><figcaption><p>Exemple avec le champ quotient familial</p></figcaption></figure>

Les données peuvent être présentées au niveau du tableau d'instruction, être utilisées pour filtrer sur les dossiers, être ajoutées dans les exports.

### Jeton API Particulier

Une démarche contenant un de ces champs ne peut être publiée que si un jeton API Particulier est configuré.

En l'absence de jeton, un message d'erreur indique le ou les champs concernés afin de permettre leur correction avant la publication.

<figure><img src="../.gitbook/assets/Capture d’écran 2026-07-06 à 18.24.45.png" alt=""><figcaption></figcaption></figure>

**Comment obtenir un jeton API Particulier ?**

Faites une [demande d’accès sur DataPass](https://datapass.api.gouv.fr/formulaires/api-particulier/demande/nouveau).

Il s'agit d'une demande d’habilitation dans laquelle vous devrez compléter les parties suivantes : &#x20;

* nom de la démarche\
  &#x20;\=> vous pouvez reprendre les informations renseignées dans Démarche Numérique
* présentation de la démarche \
  &#x20;\=> vous pouvez reprendre les informations renseignées dans Démarche Numérique
* durée de conservation des données et les personnes qui traiteront ces données (agent par exemple)\
  &#x20;\=> vous pouvez reprendre les informations renseignées dans Démarche Numérique
* cadre juridique qui vous autorise à accéder aux données que vous demandez (arrêté, délibération etc.)\
  &#x20;\=> vous pouvez reprendre les informations renseignées dans Démarche Numérique
* coordonnées du délégué à la protection des données \
  &#x20;\=> vous pouvez reprendre les informations renseignées dans Démarche Numérique
* coordonnées du contact métier
