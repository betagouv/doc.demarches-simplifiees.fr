# Le playground / Premiers pas

Une fois votre accréditation acquise, vous pouvez vous connecter en tant qu’administrateur, créer une démarche de test (car un administrateur ne peut que requêter sur ses propres démarches).

Une fois votre démarche créée, vous pouvez accéder à l’**éditeur de requêtes en ligne** (attention, ne confondez pas cette adresse avec celle de l’endpoint, qui y ressemble) : [https://www.demarches-simplifiees.fr/graphql](https://www.demarches-simplifiees.fr/graphql)

Cet éditeur de requêtes en ligne vous permet de requêter directement l'API graphql sur **vos démarches** sans que vous ayez encore à coder de client. C'est un bon outil pour essayer l'API et commencer à la prendre en main.

{% hint style="info" %}
Pour construire une requête et interpréter les réponses, consultez la [**documentation complète du schéma de l’API**](https://www.demarches-simplifiees.fr/graphql/schema/).
{% endhint %}

{% hint style="danger" %}
Attention, le playground **n'est pas une sandbox**, les requêtes effectuées sur le playground impactent directement les données de production. Si vous utilisez des opérations de mutations, pensez à utiliser une démarche de test et en aucun cas une démarche avec de vrais dossiers.
{% endhint %}
