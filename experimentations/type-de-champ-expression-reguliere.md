# Type de champ expression régulière

## Nouveau Type de Champ expérimental : Expression régulière

⚠️ Ce nouveau type de champ est à titre expérimental. La manière dont il est implémenté pourra changer à l'avenir.

Une expression régulière (regex) est une séquence de caractères utilisée pour décrire un ensemble spécifique de chaînes de caractères selon certaines syntaxes. Elle permet d'effectuer des recherches, des remplacements et des validations de données dans une chaîne de caractères.

Dans le cadre de la validation d'un champ de formulaire elle sert à valider les données entrées par un utilisateur en vérifiant si elles correspondent à un certain format. Par exemple, un regex peut vérifier qu'une adresse email entrée dans un champ de formulaire ressemble bien à une adresse email (ex: [nom@example.com](mailto:nom@example.com)) ou à un numéro de téléphone.



Afin d'activer ce type de champ par démarche, veuillez vous rapprocher de notre équipe support (contact@demarches-simplifiees.fr) avec les numéros de démarches sur lesquelles vous voulez tester.\
\
Une fois que la fonctionnalité est activée, vous pouvez vous rendre dans l'éditeur de champs et choisir un type de champ expression régulière :&#x20;

<figure><img src="../.gitbook/assets/image (248).png" alt=""><figcaption><p>Choix du type de champ Expression régulière</p></figcaption></figure>

Des informations sont à renseigner :&#x20;

<figure><img src="../.gitbook/assets/image (249).png" alt=""><figcaption></figcaption></figure>

* Saisissez votre expression régulière : vous pouvez la tester sur [https://rubular.com/](https://rubular.com/)
  * Exemple pour valider un code APE : ^\[0-9]{4}\[A-Z]$
    * Explication étape par étape :
      1. `^` et `$` : Ces symboles indiquent le début et la fin de la chaîne, respectivement. Ils s'assurent que tout le texte doit correspondre au modèle.
      2. `[0-9]{4}` : Ceci vérifie que les quatre premiers caractères sont des chiffres de 0 à 9.
      3. `[A-Z]` : Ceci vérifie qu'il y a une lettre majuscule à la fin.
* Saisissez un exemple qui passe votre expression régulière : pour que la démarche soit publiable, l'exemple doit être valide. Sinon il vous sera impossible de publier la démarche.
* Message d'erreur à afficher à l'usager : Il s'agit du message que vous voulez transmettre à l'usager en cas d'erreur de saisie&#x20;

Exemples d'expression régulières que nous testons actuellement :

* je souhaite contraindre la saisie qu'à des chiffres, pas d'espace possible, ni caractère spécial : l'expression régulière à saisir est : "^\d+$" (sans les guillemets)
* je souhaite contraindre la saisie à 9 chiffres pas un de plus ni un de moins, l'expression régulière à saisir est  : "^\d{9}$" (sans les guillemets)
