# Champ référentiel avancé (à configurer)

_Dernière mise à jour : 16 juillet 2025_&#x20;

_Des questions ? via crisp ou tchap_

Le champ référentiel avancé permet de connecter un champ de formulaire à une base de données officielle ou métier, via une API.

Il aide à :

* Vérifier automatiquement qu’un identifiant saisi existe vraiment
* Pré-remplir d’autres champs du formulaire pour éviter les ressaisies
* Afficher des informations utiles à l’usager (publiques, non modifiables)
* Afficher des informations réservées à l’instructeur (privées, non modifiables)

Ce champ vous aide à fiabiliser la saisie et à simplifier l’expérience des usagers.

> **Limite actuelle** Le champ référentiel avancé fonctionne uniquement pour la recherche d’un identifiant unique. Il ne propose pas encore de liste de résultats ou d’autocomplétion. L’usager doit donc saisir précisément l’identifiant recherché.

***

## Vidéo

{% embed url="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L7_aKvpAJdAIEfxHudA%2Fuploads%2FULpp01kmWBCrRRsOwXpH%2Foutput-tiny-h264.mp4?alt=media&token=4df1430f-cd92-4536-a6e4-a3f84859eea8" %}

## Sommaire

* [Présentation générale](champ-referentiel-avance-a-configurer.md#champ-référentiel-avancé)
* [Vidéo](champ-referentiel-avance-a-configurer.md#vidéo)
* [Cas d’usage : valider un identifiant (exemple : numéro RNB)](champ-referentiel-avance-a-configurer.md#cas-dusage--valider-un-identifiant-exemple--numéro-rnb)
* [Cas d’usage : pré-remplissage automatique des champs via le référentiel](champ-referentiel-avance-a-configurer.md#cas-dusage--pré-remplissage-automatique-des-champs-via-le-référentiel)
* [Cas d’usage : afficher des données publiques à l’usager](champ-referentiel-avance-a-configurer.md#cas-dusage--afficher-des-données-publiques-à-lusager)
* [Cas d’usage : afficher des données privées à l’instructeur](champ-referentiel-avance-a-configurer.md#cas-dusage--afficher-des-données-privées-à-linstructeur)
* [Gestion de l’authentification](champ-referentiel-avance-a-configurer.md#gestion-de-lauthentification)
* [Questions fréquentes](champ-referentiel-avance-a-configurer.md#questions-fréquentes)
* [Évolutions à venir](champ-referentiel-avance-a-configurer.md#évolutions-à-venir)
* [Table de correspondance des types de données](champ-referentiel-avance-a-configurer.md#table-de-correspondance-des-types-de-données)
* [Que se passe-t-il en cas d’erreur lors de l’appel à l’API ?](champ-referentiel-avance-a-configurer.md#que-se-passe-t-il-en-cas-derreur-lors-de-lappel-à-lapi)
* [Glossaire](champ-referentiel-avance-a-configurer.md#glossaire)

***

### Cas d’usage : valider un identifiant (exemple : numéro RNB)

#### Fonctionnement

* L’usager saisit un identifiant dans le champ référentiel avancé.
* Lors de la validation, la plateforme interroge l’API du référentiel externe avec l’identifiant saisi.
* Si l’API renvoie une réponse valide, la saisie est acceptée. Sinon, un message d’erreur s’affiche.

{% file src="../.gitbook/assets/recherche-id.mov" %}

#### Comment configurer ce cas d’usage

1. **Ajoutez un champ référentiel avancé**
   * Dans l’éditeur de formulaire, ajoutez un champ de type « référentiel avancé à configurer ».
2. **Cliquez sur "Configurer le champ"**
3. **Configurez l’URL du référentiel**
   * Renseignez l’URL de l’API à interroger, par exemple : https://rnb-api.beta.gouv.fr
   * **Utilisation du placeholder `{id}`** : Vous devez inclure `{id}` dans l’URL (ex : `https://rnb-api.beta.gouv.fr/api/alpha/buildings/{id}/`). Lors de l’utilisation, `{id}` sera remplacé par la valeur saisie par l’usager dans le champ (ex : la plaque d’immatriculation).

<figure><img src="../.gitbook/assets/Capture d’écran 2025-06-17 à 9.58.27 AM.png" alt=""><figcaption></figcaption></figure>

5. **Contraintes de sécurité**
   * L’URL doit pointer vers un service de confiance, sécurisé (HTTPS).
   * L’API ne doit pas exposer de données sensibles sans authentification.
   * L’URL ne doit pas permettre d’exécuter du code ou de modifier des données côté serveur.
   * Seules les URL explicitement autorisées par l’administrateur de la plateforme sont acceptées (les .gouv.fr sont par défaut autorisées).
6. **Utilisez le champ `Exemple de saisie valide (affiché à l'usager et utilisé pour tester la requête)`**
   * Ce champ permet de simuler une réponse de l’API lors de la configuration.
   * L’API sera appelée avec cette valeur pour vérifier qu’elle répond selon nos standards (actuellement, seuls les référentiels qui répondent à une requête HTTP GET, au format JSON, avec un statut HTTP 200 sont acceptés).

#### Exemple de validation attendue (statut HTTP 200)

Pour que la validation soit acceptée, l’API doit répondre avec un statut HTTP **200** (succès) et un corps JSON valide, même minimal (par exemple : `{}`).

```json
{}
```

#### Bonnes pratiques

* Vérifiez que l’URL commence bien par `https://`.
* Indiquez clairement à l’usager où trouver son identifiant.
* Si la saisie échoue, vérifiez l’orthographe et le format de l’identifiant.

***

### Cas d’usage : pré-remplissage automatique des champs via le référentiel

#### Fonctionnement

Le pré-remplissage permet d’automatiser la saisie de certains champs du formulaire à partir de données issues d’une base externe (référentiel), après validation d’un identifiant (ex : numéro RNB). Les données récupérées (adresse, statut, etc.) sont injectées automatiquement dans les champs du formulaire selon la correspondance (mapping) que vous définissez.

#### Comment configurer ce cas d’usage

1. **Visualisez la réponse de l’API**
   * Une fois l’URL et l’identifiant de test renseignés, la réponse JSON s’affiche dans l’interface.

<figure><img src="../.gitbook/assets/Capture d’écran 2025-07-16 à 10.27.53 AM.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Capture%20d%E2%80%99%C3%A9cran%202025-06-17%20%C3%A0%2010.02.03%E2%80%AFAM.png" alt=""><figcaption></figcaption></figure>

2. **Décidez l’usage de chaque propriété du JSON**
   * Selon le type de donnée, l’interface vous propose les champs compatibles avec votre formulaire.
   * Vous pouvez choisir d’utiliser la donnée pour pré-remplir un champ, ou l’afficher dans le formulaire usager et/ou dans l’interface instructeur.

<figure><img src="../.gitbook/assets/Capture d’écran 2025-07-16 à 10.28.03 AM.png" alt=""><figcaption></figcaption></figure>

3. **Faites le mapping**
   * Utilisez l’interface de mapping pour lier chaque propriété (ex : `addresses[0].street`, `status`) au champ cible du formulaire.

<figure><img src="../.gitbook/assets/Capture d’écran 2025-07-16 à 10.30.37 AM.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Capture%20d%E2%80%99%C3%A9cran%202025-06-17%20%C3%A0%2010.03.52%E2%80%AFAM.png" alt=""><figcaption></figcaption></figure>

4. **Gérez les répétitions (tableaux)**
   * Si la donnée source est un tableau (ex : plusieurs adresses), chaque élément peut être mappé sur une répétition du formulaire.

Une fois la configuration terminée, un résumé du pré-remplissage s’affiche. Il liste les correspondances entre les données du référentiel et les champs du formulaire.

<figure><img src="../.gitbook/assets/Capture d’écran 2025-07-16 à 10.27.16 AM.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Capture%20d%E2%80%99%C3%A9cran%202025-06-17%20%C3%A0%2010.07.46%E2%80%AFAM.png" alt=""><figcaption></figcaption></figure>

**Vérifiez l’affichage** Lors d’un test, les champs sont automatiquement pré-remplis pour l’usager.

<figure><img src="../.gitbook/assets/Capture d’écran 2025-07-16 à 10.40.15 AM.png" alt=""><figcaption></figcaption></figure>

#### Exemple de réponse API (pré-remplissage automatique)

Voici un exemple de réponse typique reçue lors du pré-remplissage : chaque propriété du JSON ci-dessous peut être associée à un ou plusieurs champs du formulaire grâce à l’interface de mapping. Les tableaux (comme `addresses`) permettent de remplir automatiquement des répétitions (ex : plusieurs adresses). Les propriétés simples (ex : `status`, `is_active`) peuvent être reliées à des champs individuels.

```json
{
  "rnb_id": "PG46YY6YWCX8",
  "status": "constructed",
  "is_active": true,
  "addresses": [
    {
      "id": "33063_1155_00002",
      "street": "place de la bourse",
      "city_name": "Bordeaux",
      "city_zipcode": "33000",
      "street_number": "2"
    },
    {
      "id": "33063_1155_00008",
      "street": "place de la bourse",
      "city_name": "Bordeaux",
      "city_zipcode": "33000",
      "street_number": "8"
    }
  ]
}
```

> **Astuce** Pour chaque propriété, vérifiez qu’elle correspond bien au type de champ cible (texte, nombre, case à cocher, etc.). Pour les tableaux, chaque élément sera mappé sur une ligne de répétition du formulaire.

#### Bonnes pratiques

* Vérifiez que chaque champ cible correspond bien au type de donnée attendu (ex : texte, nombre, date…).
* Pour les champs à choix (liste déroulante, cases à cocher…), assurez-vous que les valeurs du référentiel sont compatibles avec les options du formulaire.
* En cas de doute, testez le pré-remplissage sur un dossier de test.

***

### Cas d’usage : afficher des données publiques à l’usager

#### Fonctionnement

Après validation d’un identifiant, certaines informations issues du référentiel (ex : adresse, statut, nom du bâtiment…) peuvent être affichées à l’usager dans le formulaire, sans qu’il puisse les modifier. Cela permet de donner du contexte, de rassurer l’usager ou de l’aider à vérifier qu’il a saisi le bon identifiant.

#### Comment configurer ce cas d’usage

1. **Configurez le champ référentiel avancé** (voir les étapes précédentes pour l’URL et l’identifiant de test)
2. **Faites le mapping des propriétés à afficher**
   * Dans l’interface de mapping, cochez l’option “Afficher à l’usager” pour chaque donnée que vous souhaitez rendre visible.

<figure><img src="../.gitbook/assets/Capture d’écran 2025-07-16 à 10.37.06 AM.png" alt=""><figcaption></figcaption></figure>

3. **Vérifiez l’affichage**
   * Lors d’un test, les données publiques apparaissent en lecture seule dans le formulaire, sous le champ référentiel.

<figure><img src="../.gitbook/assets/Capture d’écran 2025-07-16 à 10.40.03 AM.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Capture%20d%E2%80%99%C3%A9cran%202025-06-17%20%C3%A0%2010.07.46%E2%80%AFAM.png" alt=""><figcaption></figcaption></figure>

> **Astuce** Affichez uniquement les informations utiles à l’usager (ex : adresse, nom, statut), pas des identifiants techniques.

#### Exemple de réponse API (affichage de données publiques)

Voici un exemple de réponse API permettant d’afficher des données publiques à l’usager : seules les propriétés simples (texte, nombre, booléen, etc.) sont supportées. Les tableaux ne peuvent pas être affichés directement à l’usager.

```json
{
  "nom_batiment": "Tour Alpha",
  "adresse": "2 place de la Bourse, 33000 Bordeaux",
  "statut": "En service",
  "est_actif": true
}
```

#### Bonnes pratiques

* N’affichez que des données compréhensibles et utiles pour l’usager.
* Privilégiez la clarté : ajoutez un libellé explicite à chaque donnée affichée.
* Vérifiez le rendu sur un dossier de test.

#### Questions fréquentes

**Q : L’usager peut-il modifier les données affichées ?**\
R : Non, ces informations sont affichées en lecture seule.

**Q : Peut-on masquer certaines données du référentiel à l’usager ?**\
R : Oui, il suffit de ne pas cocher l’option “Afficher à l’usager” dans le mapping.

***

### Cas d’usage : afficher des données privées à l’instructeur

#### Fonctionnement

Après validation d’un identifiant, certaines informations issues du référentiel (ex : statut, historique, données techniques…) peuvent être affichées uniquement à l’instructeur, dans l’interface d’instruction du dossier. L’usager ne voit pas ces informations.

#### Comment configurer ce cas d’usage

1. **Configurer le champ référentiel avancé** (voir les étapes précédentes pour l’URL et l’identifiant de test)
2. **Faire le mapping des propriétés à afficher à l’instructeur** Dans l’interface de mapping, cochez l’option “Afficher à l’instructeur” pour chaque donnée que vous souhaitez rendre visible uniquement côté instructeur.

<figure><img src="../.gitbook/assets/Capture d’écran 2025-07-16 à 10.37.15 AM.png" alt=""><figcaption></figcaption></figure>

1. **Vérifier l’affichage** Lors d’un test, les données privées apparaissent dans la fiche du dossier, visibles uniquement par l’instructeur.

<figure><img src="../.gitbook/assets/Capture d’écran 2025-07-16 à 10.42.10 AM.png" alt=""><figcaption></figcaption></figure>

> **Astuce** Utilisez cette fonctionnalité pour transmettre des informations sensibles ou techniques qui ne concernent pas l’usager (ex : statut d’un bâtiment, données cadastrales, historique de contrôles).

#### Exemple de réponse API (affichage de données privées à l’instructeur)

Voici un exemple de réponse API permettant d’afficher des données privées à l’instructeur : seules les propriétés simples (texte, nombre, booléen, etc.) sont supportées. Les tableaux ne peuvent pas être affichés directement côté instructeur.

```json
{
  "statut": "Contrôlé le 12/07/2025",
  "historique": "Dernier contrôle conforme",
  "code_instructeur": 12345,
  "est_confidentiel": true
}
```

#### Bonnes pratiques

* N’affichez côté instructeur que des données utiles à la prise de décision ou à l’instruction du dossier.
* Ajoutez un libellé explicite à chaque donnée affichée.
* Vérifiez le rendu sur un dossier de test avec un compte instructeur.

#### Questions fréquentes

**Q : L’usager voit-il ces données ?**\
R : Non, seules les personnes ayant accès à l’interface d’instruction peuvent les consulter.

**Q : Peut-on afficher une donnée à la fois à l’usager et à l’instructeur ?**\
R : Oui, il suffit de cocher les deux options dans le mapping.

***

### Gestion de l’authentification

Vous pouvez connecter le champ référentiel avancé à une API nécessitant une authentification.

Actuellement, seule l’authentification par en-tête HTTP est prise en charge.

**Exemple concret** : l’intégration avec Grist (voir la vidéo ci-dessus).

Pour configurer l’authentification :

1. Accédez à l’écran « Configuration du champ » (nom du champ concerné).
2. Activez l’option « Authentification ».
3. Choisissez l’en-tête à utiliser (par exemple : `Authorization`).
4. Saisissez la valeur attendue, par exemple : `Bearer {votre_token}`.

L’en-tête et sa valeur seront ajoutés à chaque requête API.

<figure><img src="../.gitbook/assets/Capture d’écran 2025-07-16 à 2.18.58 PM.png" alt=""><figcaption></figcaption></figure>

> **Astuce** Protégez toujours vos jetons d’authentification et ne les partagez pas publiquement.

#### Questions fréquentes

**Q : Mon token est-il stocké de manière sécurisée ?** R : Oui, nous appliquons plusieurs mesures de sécurité :

* Tous les tokens sont chiffrés avant d’être enregistrés.
* Les tokens ne sont jamais affichés dans l’interface, même pour les administrateurs.
* Lorsqu’une démarche est clonée, les tokens ne sont pas copiés : il faut les reconfigurer dans la nouvelle démarche.

> **Astuce** Pensez à renouveler régulièrement vos tokens et à limiter leur durée de validité pour renforcer la sécurité.

***

### Questions fréquentes

**Q : Que se passe-t-il si une donnée n’est pas trouvée ou est incomplète ?**\
R:

* Le système fait du “best effort” : il préremplit ce qu’il peut, et laisse les autres champs vides.
* La validation du formulaire reste active : l’usager pourra compléter ou corriger les champs si besoin.

**Q : Peut-on supprimer automatiquement des lignes de répétition si le tableau du référentiel est vide ?**\
R : Non, seules des lignes supplémentaires sont créées lors du pré-remplissage. Les suppressions doivent être faites manuellement.

**Q : Que faire si un champ n’est pas prérempli comme attendu ?**\
R : Vérifiez le mapping dans la configuration et assurez-vous que la donnée existe bien dans le référentiel.

**Q : Que faire si l'API change** R : Si votre API change, il vous faudra mettre à jour le champ référentiel avancé à configurer

**Q : Quelles sont les évolutions à venir ?** R : Elles seront nombreuses. Vous pouvez avoir un aperçu [ici](https://github.com/demarches-simplifiees/demarches-simplifiees.fr/issues/11161). Pour faire court :

* l'autocomplete
* le support du conditionnel pour les donnée affichées aux usagers/instructeur
* le support des balises (attestation/mail) pour les données affichées aux usagers/instructeurs
* le support des filtres pour les données affichées aux usagers/instructeurs

***

### Évolutions à venir

Pour suivre les évolutions prévues ou en cours, consultez la page dédiée : [Suivi des évolutions](https://github.com/demarches-simplifiees/demarches-simplifiees.fr/issues/11161).

Principales fonctionnalités à venir :

* Autocomplétion (autocomplete)
* Support du conditionnel pour les données affichées aux usagers/instructeurs
* Support des balises (attestation/mail) pour les données affichées aux usagers/instructeurs
* Support des filtres pour les données affichées aux usagers/instructeurs

***

### Table de correspondance des types de données

> **À quoi sert cette table ?** Elle vous aide à savoir comment chaque type de donnée reçu depuis le référentiel (API) peut être relié à un champ du formulaire.

| **Type de donnée du référentiel (JSON)** | **Type de champ cible dans le formulaire** | **Remarques principales**                    |
| ---------------------------------------- | ------------------------------------------ | -------------------------------------------- |
| string                                   | text, textarea, formatted                  | Texte court, long ou formaté                 |
| integer                                  | integer\_number                            | Nombre entier                                |
| float / number                           | decimal\_number                            | Nombre à virgule                             |
| boolean                                  | yes\_no, checkbox                          | Case à cocher ou Oui/Non                     |
| date (ISO8601 ou dd/mm/yyyy)             | date                                       | Date simple, format automatique              |
| datetime (ISO8601, dd/mm/yyyy hh:mm)     | datetime                                   | Date et heure, format automatique            |
| array de valeurs simples                 | multiple\_drop\_down\_list                 | Liste à choix multiples                      |
| array d’objets                           | repetition                                 | Répétition (tableau de sous-champs)          |
| objet                                    | groupement de champs (mapping explicite)   | Peut être mappé sur plusieurs champs simples |
| string                                   | drop\_down\_list (avec option “autre”)     | Saisie libre si activée                      |
| string                                   | siret, iban, rna                           | Champs spécialisés, validation intégrée      |

**À retenir :**

* Le mapping est automatique pour les types simples (string, integer, float, boolean, date, datetime).
* Pour les tableaux, chaque élément peut être mappé sur une ligne de répétition.
* Les objets imbriqués nécessitent un mapping explicite champ par champ.
* Les types avancés (SIRET, IBAN, etc.) sont mappés directement, puis validés à la soumission du dossier.

### Que se passe-t-il en cas d’erreur lors de l’appel à l’API ?

Lorsque le champ référentiel avancé interroge une API externe, plusieurs types d’erreurs peuvent survenir. Le système gère automatiquement la plupart des cas pour garantir la meilleure expérience possible à l’usager et à l’administrateur.

* Si l’API répond avec un code d’erreur temporaire (ex : surcharge, indisponibilité…), la plateforme réessaie automatiquement l’appel avant d’afficher un message d’erreur.
* Si l’API répond avec un code d’erreur permanent (ex : identifiant non trouvé, accès refusé…), un message explicite est affiché à l’usager.
* Si l’API ne répond pas ou renvoie un format inattendu, un message d’erreur générique est affiché.

#### Messages d’erreur affichés à l’usager

Selon le code de retour de l’API, l’usager verra l’un des messages suivants :

| Code HTTP | Message affiché à l’usager                                  |
| --------- | ----------------------------------------------------------- |
| 429       | Trop de demandes. Nous réessayons pour vous.                |
| 500       | Erreur du serveur. Nous réessayons pour vous.               |
| 503       | Service indisponible. Nous réessayons pour vous.            |
| 408       | La demande a pris trop de temps. Nous réessayons pour vous. |
| 502       | Problème de connexion. Nous réessayons pour vous.           |
| 404       | Résultat introuvable. Vérifiez vos informations.            |
| 400       | Demande incorrecte. Vérifiez vos informations.              |
| 403       | Accès refusé. Vous n’avez pas les droits nécessaires.       |
| 401       | Non autorisé. Connectez-vous pour continuer.                |
| Autre     | Aucun résultat ne correspond à votre recherche.             |

> **Astuce** En cas d’erreur temporaire, l’usager n’a rien à faire : la plateforme gère les tentatives automatiquement. En cas d’erreur permanente, il doit vérifier l’identifiant saisi ou contacter l’assistance.

***

### Glossaire

> Besoin d’aide ou d’une suggestion ? Contactez-nous via Crisp ou Tchap (voir le site d’administration pour les liens directs).

**API** : Interface permettant à deux applications de communiquer automatiquement.

**Mapping** : Association entre une donnée reçue (ex : propriété JSON) et un champ du formulaire.

**Répétition** : Groupe de champs pouvant être dupliqué automatiquement pour gérer des listes (ex : plusieurs adresses).

**Identifiant** : Valeur unique permettant d’identifier un élément dans un référentiel (ex : numéro RNB, SIRET).

**Référentiel** : Base de données officielle ou métier utilisée pour vérifier ou enrichir les informations saisies dans le formulaire.

**Champ référentiel avancé** : Champ de formulaire connecté à un référentiel externe via une API, avec des fonctionnalités de validation, pré-remplissage et affichage de données.
