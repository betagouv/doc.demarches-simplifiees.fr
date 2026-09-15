# Point d'entrée et Schéma GraphQL

Vous avez maintenant les pré-requis pour avancer sur votre propre client des API de **demarche.numerique.gouv.fr** :

* vous disposez d'un compte administrateur sur **demarche.numerique.gouv.fr**
* vous avez une démarche de test publiée (vous pouvez en créer une)
* vous arrivez à structurer des appels à l'API en combinant la `query` qui détermine les données que vous souhaitez récupérer, et les `variables` qui déterminent la logique métier que nous devons appliquer. Bref, vous avez pris vos aises avec un client GraphQL, votre démarche de test et votre jeton.

Vous pouvez maintenant ouvrir votre éditeur de code préféré et garder en tête que :

{% hint style="danger" %}
Les requêtes sont à envoyer au endpoint : **https://demarche.numerique.gouv.fr/api/v2/graphql**
{% endhint %}

{% hint style="danger" %}
Les requêtes doivent être envoyées avec le verbe HTTP **POST**
{% endhint %}

{% hint style="danger" %}
Les **headers doivent contenir le jeton d'authentification au format attendu**, cf. : [jeton d'authentification](jeton-dauthentification/)
{% endhint %}

{% hint style="danger" %}
Le corps de votre requête POST, **doit être un JSON respectant la spécification GraphQL**
{% endhint %}

{% hint style="info" %}
**Cette adresse n’est pas visitable dans un navigateur**. Elle renvoie des données au format JSON, à travers un transport HTTPS.
{% endhint %}

Pour plus d'information concernant le schéma, vous pouvez :

* consulter la documentation : [https://www.demarche.numerique.gouv.fr/graphql/schema/index.html](https://demarche.numerique.gouv.fr/graphql/schema/index.html)
* consulter le fichier graphql.schema : [https://github.com/demarche-numerique/demarche.numerique.gouv.fr/blob/main/app/graphql/schema.graphql](https://github.com/demarche-numerique/demarche.numerique.gouv.fr/blob/main/app/graphql/schema.graphql)
