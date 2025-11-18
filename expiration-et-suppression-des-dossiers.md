# Expiration et suppression des dossiers

### Suppression automatique des dossiers

Quel que soit leur état, les dossiers ne sont conservés sur la plateforme que pendant la durée de conservation définie au moment de la création de la démarche. Cette durée de conservation peut être modifiée a posteriori.

Deux semaines avant l’expiration d’un dossier, l’usager et l’administration sont notifiés par e-mail. Un bandeau est affiché sur la page du dossier, pour signaler que ce dossier va bientôt expirer. **L’usager a la possibilité de demander une extension de la conservation d'une durée égale à la durée de conservation d'origine (renouvelable)**. Les instructeurs peuvent aussi demander une extension de conservation, mais d'un mois (renouvelable).

Une fois le délai de conservation dépassé, le dossier est définitivement supprimé de la plateforme.

{% hint style="info" %}
Temporairement, il existe deux exceptions afin de permettre aux usagers et aux administrations de s'habituer aux nouvelles règles.

* Les dossiers dans l’état « en instruction » n'expirent pas – on attend d'avoir une conversation avec les administrations sur les modalités d'expiration possibles dans cet état.
* Les dossiers en état « terminé » (accepté, refusé et classé sans suite) ne sont expirés que sur les démarches dont l'administrateur a demandé l'activation de la fonctionnalité.

Les outils d'export sont disponibles sur votre interface et la suppression des dossiers est activée sur toutes les démarches.
{% endhint %}

<figure><img src=".gitbook/assets/cycle vie dossier.png" alt=""><figcaption></figcaption></figure>

### Suppression automatique des comptes usagers

Les usagers inactifs (qui ne se sont pas connectés pendant deux ans et n'ayant pas de dossier en instruction) sont automatiquement supprimés. Deux semaines avant la suppression du compte,  l’usager est notifié par e-mail. Après la suppression, il pourra plus tard recréer un compte avec la même adresse électronique. Si l'usager ne souhaite pas que nous supprimions son compte automatiquement, il lui suffit de se connecter une fois à son compte avec ses identifiants habituels. Le délai sera réinitialisé et le compte conservé à nouveau pendant 2 ans.

Concernant les dossiers des usagers inactifs, nous supprimons : les dossiers en brouillon et les dossiers en construction (nous considérons qu'un dossier en construction inactif pendant 2 ans ne sera jamais déposé).

Les dossiers terminés sont conservés, mais le lien avec l'usager n'est plus possible car son compte a été détruit.

### Supprimer un dossier manuellement

La suppression manuelle d’un dossier est possible en fonction de l'état du dossier :

* **Dans l’état « brouillon »** : un dossier n’est visible que par l’usager, et peut être supprimé par ce dernier à tout moment. Le dossier supprimé est placé dans l'état « supprimé récemment », et la suppression peut être annulée pendant deux semaines.
* **Dans l'état « en construction »** : le dossier ne peut être supprimé que par l’usager. Cela revient à retirer la demande. Dans ce cas, l'administration est notifiée par email de la suppression du dossier par l’usager. L'usager dispose de deux semaines pour changer d'avis. Passé ce délai, le dossier est définitivement supprimé.
* Dans l'état « en instruction » : le dossier ne peut pas être supprimé – ni par l’usager, ni par les instructeurs.
* **Dans l'état « terminé »** (accepté, refusé et classé sans suite) : l’usager et les instructeurs peuvent faire une demande de suppression. Le dossier est alors placé dans l'état « supprimé récemment », mais de manière indépendante pour les usagers et l'administration. Le dossier n'est supprimé définitivement de la plateforme que deux semaines après que les deux parties aient demandé la suppression. **Si l’une des parties ne demande jamais la suppression, le dossier sera supprimé définitivement à la date d’expiration programmée**.\
  Tant que le dossier est dans l'état « supprimé récemment », la suppression peut être annulée. Si l’administration change d’avis sur un dossier dont la suppression était demandée par l’usager, le dossier sera restauré dans l’interface de l’usager. La double suppression est là pour s’assurer que l’administration comme les usagers aient le temps d’archiver les dossiers – tout en permettant à chacune des parties d’avoir un contrôle sur les dossiers visibles dans leur interface.

### Supprimer une démarche

La suppression des démarches est en grande partie basée sur les règles des dossiers. Un administrateur peut demander la suppression d’une démarche à tout moment, à condition qu'il n'y ait pas de dossiers en état « en instruction » dessus.

Une fois la suppression demandée, la démarche sera immédiatement masquée. Les dossiers en état « brouillon » et « en construction » seront supprimés après une semaine. Les dossiers terminés seront marqués comme « supprimé par l'administration », mais resteront visibles par les usagers jusqu’à leur expiration (ou jusqu'à ce que l’usager demande leur suppression).

La démarche est définitivement supprimée une fois qu’il ne reste plus de dossiers dessus. La demande de suppression d’une démarche peut être annulée pendant la période d’attente de suppression des dossiers.
