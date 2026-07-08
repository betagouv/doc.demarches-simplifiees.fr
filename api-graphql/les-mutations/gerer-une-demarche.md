# Gérer une démarche

Ces mutations permettent de piloter le cycle de vie d'une démarche directement via l'API. Elles nécessitent un jeton disposant des droits en **écriture**.

Dans chaque exemple, la démarche cible est identifiée par son numéro (`number`) ; vous pouvez aussi l'identifier par son `id`.

### Publier une démarche

Publie (ou republie) une démarche à l'adresse indiquée par `path`.

{% hint style="warning" %}
Si le `path` est déjà utilisé par une autre de vos démarches, celle-ci sera **dépubliée et remplacée**. Le champ `lienSiteWeb` (où les usagers trouveront le lien vers la démarche) est obligatoire.
{% endhint %}

```graphql
mutation {
  demarchePublier(input: {
    demarche: { number: 123 }
    path: "ma-demarche"
    lienSiteWeb: "https://mon-site.gouv.fr/ma-demarche"
    robotsIndexable: true
  }) {
    demarche { id number }
    errors { message }
    warnings { message }
  }
}
```

`robotsIndexable` (optionnel, `true` par défaut) autorise le référencement de la démarche par les moteurs de recherche.

### Modifier les paramètres d'une démarche

Met à jour un ou plusieurs paramètres de la démarche. Tous les arguments sont optionnels, mais au moins un doit être fourni.

```graphql
mutation {
  demarcheModifierParametres(input: {
    demarche: { number: 123 }
    title: "Nouveau titre de la démarche"
    description: "Nouvelle description"
    dateLimite: "2026-12-31"
  }) {
    demarche { id number }
    errors { message }
  }
}
```

Paramètres modifiables : `title`, `description`, `descriptionTargetAudience`, `descriptionPj`, `lienSiteWeb`, `lienDpo`, `dateLimite` (date limite de dépôt des dossiers) et `declarative`.

{% hint style="info" %}
La `dateLimite` doit être une date **dans le futur**, au format `AAAA-MM-JJ`.
{% endhint %}

### Cloner une démarche

Crée une copie de la démarche. Vous pouvez donner un titre à la copie et choisir de cloner également le **service** associé.

```graphql
mutation {
  demarcheCloner(input: {
    demarche: { number: 123 }
    title: "Copie de ma démarche"
    cloneService: true
  }) {
    demarche { id number }
    errors { message }
  }
}
```

`cloneService` (optionnel) : si `true`, le service de la démarche d'origine est également cloné et rattaché à la nouvelle démarche.
