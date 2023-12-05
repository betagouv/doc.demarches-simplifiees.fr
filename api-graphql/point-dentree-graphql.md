# Point d'entrée GraphQL

Vous avez maintenant les pré-requis pour avancer sur votre propre client des API de Démarche Simplifiées :&#x20;

* vous disposez d'un compte administrateur sur demarches-simplifiees.fr&#x20;
* vous avez une démarche de test publiée (vous pouvez en créer une)
* vous arrivez à structurer des appels à l'API en combinant la `query` qui détermine les données que vous souhaitez récupérer, et les `variables` qui déterminent la logique métier que nous devons appliquer. Bref, vous avez pris vos aises sur le playground, avec votre démarche de test et votre compte administrateur.

Vous pouvez maintenant ouvrir votre éditeur de code préféré et définir l'host de notre API : **https://www.demarches-simplifiees.fr/api/v2/graphql**&#x20;

{% hint style="info" %}
**Cette adresse n’est pas visitable dans un navigateur**. Elle renvoie des données au format JSON, à travers un transport HTTPS. Pour communiquer avec le endpoint, **veuillez toujours utiliser le verbe http POST**.
{% endhint %}
