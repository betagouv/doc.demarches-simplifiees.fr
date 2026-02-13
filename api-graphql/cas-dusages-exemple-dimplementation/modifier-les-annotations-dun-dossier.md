---
description: >-
  Exemple d'implémentation pour modifier les annotations privées d'un dossier.
  Les annotations sont des champs visibles uniquement par les instructeurs.
---

# Modifier les annotations d'un dossier

Les annotations sont des champs privés d'un dossier, visibles uniquement par les instructeurs. Elles permettent d'ajouter des informations internes de traitement sans que l'usager puisse les voir. Cette mutation permet à un instructeur de modifier la valeur d'une ou plusieurs annotations.

{% hint style="info" %}
Pour récupérer l'identifiant d'une annotation et son type, vous pouvez interroger un dossier en incluant les champs `annotations` dans votre requête GraphQL.
{% endhint %}

{% hint style="info" %}
Pour connaître l'identifiant d'un instructeur, consultez la documentation [lister-les-id-des-instructeurs.md](lister-les-id-des-instructeurs.md "mention").
{% endhint %}

## Types d'annotations supportés

La mutation `dossierModifierAnnotations` supporte les types de champs suivants :

| Type de champ | Clé dans `value` | Type de valeur | Exemple |
|---------------|------------------|----------------|---------|
| Texte | `text` | `String` | `{ text: "Ma valeur" }` |
| Zone de texte | `textarea` | `String` | `{ textarea: "Long texte..." }` |
| Adresse électronique | `email` | `String` | `{ email: "user@example.com" }` |
| Case à cocher | `checkbox` | `Boolean` | `{ checkbox: true }` |
| Oui/Non | `yesNo` | `Boolean` | `{ yesNo: false }` |
| Date | `date` | `ISO8601Date` | `{ date: "2024-01-15" }` |
| Date et heure | `datetime` | `ISO8601DateTime` | `{ datetime: "2024-01-15T10:30:00Z" }` |
| Nombre entier | `integerNumber` | `Int` | `{ integerNumber: 42 }` |
| Nombre décimal | `decimalNumber` | `Float` | `{ decimalNumber: 3.14 }` |
| Liste déroulante | `dropDownList` | `String` | `{ dropDownList: "Option A" }` |
| Choix multiple | `multipleDropDownList` | `[String]` | `{ multipleDropDownList: ["Option A", "Option B"] }` |
| Répétition | `repetition` | `Int` | `{ repetition: 3 }` (ajoute 3 lignes) |

{% hint style="info" %}
La valeur utilise un format "one_of" : vous devez fournir **exactement une** des clés correspondant au type de votre annotation.
{% endhint %}

## Exemple 1 : Modifier une annotation texte

Vous pouvez tester en executant le script suivant avec les variables d'environnement adaptées :

<pre class="language-bash"><code class="lang-bash"><strong>API_TOKEN="votre_token" DOSSIER_ID="l_id_du_dossier" INSTRUCTEUR_ID="l_id_de_l_instructeur" ANNOTATION_ID="l_id_de_l_annotation" ANNOTATION_VALUE="nouvelle_valeur" ruby modifierAnnotationTexte.rb
</strong></code></pre>

{% code title="modifierAnnotationTexte.rb" %}
```ruby
# frozen_string_literal: true

require 'net/http'
require 'uri'
require 'json'
ENDPOINT = URI('https://www.demarches-simplifiees.fr/api/v2/graphql')

### that's the GraphQL part.
QUERY = <<~GRAPHQL_QUERY
  mutation dossierModifierAnnotations($input: DossierModifierAnnotationsInput!) {
    dossierModifierAnnotations(input: $input) {
      annotations {
        id
        label
        stringValue
        updatedAt
      }
      errors {
        message
      }
    }
  }
GRAPHQL_QUERY

### that's the HTTP part
# open an http connexion to our GraphQL endpoint
def open_http_connection
  http = Net::HTTP.new(ENDPOINT.host, ENDPOINT.port)
  if ENDPOINT.scheme == 'https'
    http.use_ssl = true
    http.verify_mode = OpenSSL::SSL::VERIFY_NONE
  end
  http
end

# the headers of our http query, include auth
def request_headers
  {
    "Content-Type" => "application/json",
    "Authorization" => "Bearer #{ENV.fetch('API_TOKEN') { raise 'missing env var API_TOKEN' }}"
  }
end

# given an http connexion, request the API
def modifier_annotation_texte(http)
  # the data of our query
  data = {
    "query" => QUERY,
    "operationName" => "dossierModifierAnnotations",
    "variables" => {
      "input": {
        instructeurId: ENV.fetch('INSTRUCTEUR_ID') { raise 'missing env var INSTRUCTEUR_ID' },
        dossierId: ENV.fetch('DOSSIER_ID') { raise 'missing env var DOSSIER_ID' },
        annotations: [
          {
            id: ENV.fetch('ANNOTATION_ID') { raise 'missing env var ANNOTATION_ID' },
            value: {
              text: ENV.fetch('ANNOTATION_VALUE') { raise 'missing env var ANNOTATION_VALUE' }
            }
          }
        ]
      }
    }
  }

  # the HTTP format
  req = Net::HTTP::Post.new(ENDPOINT, request_headers)
  # the request body format
  req.body = data.to_json

  response = http.request(req)

  data = JSON.parse(response.body)
  data
end

http = open_http_connection
data = modifier_annotation_texte(http)

puts "Info: annotation(s) modifiée(s): #{data}"

```
{% endcode %}

## Exemple 2 : Modifier plusieurs annotations de types différents

Vous pouvez modifier plusieurs annotations en une seule requête. Voici un exemple qui modifie une date, un nombre entier et une case à cocher :

```ruby
# Modifier plusieurs annotations
data = {
  "query" => QUERY,
  "operationName" => "dossierModifierAnnotations",
  "variables" => {
    "input": {
      instructeurId: ENV['INSTRUCTEUR_ID'],
      dossierId: ENV['DOSSIER_ID'],
      annotations: [
        {
          id: "annotation_date_id",
          value: { date: "2024-12-25" }
        },
        {
          id: "annotation_nombre_id",
          value: { integerNumber: 42 }
        },
        {
          id: "annotation_checkbox_id",
          value: { checkbox: true }
        }
      ]
    }
  }
}
```

## Exemple 3 : Ajouter des lignes à un champ répétable

Pour un champ de type répétition, vous spécifiez le nombre de lignes à ajouter :

```ruby
data = {
  "query" => QUERY,
  "operationName" => "dossierModifierAnnotations",
  "variables" => {
    "input": {
      instructeurId: ENV['INSTRUCTEUR_ID'],
      dossierId: ENV['DOSSIER_ID'],
      annotations: [
        {
          id: "annotation_repetition_id",
          value: { repetition: 3 }  # Ajoute 3 nouvelles lignes
        }
      ]
    }
  }
}
```

{% hint style="warning" %}
Attention : le format de la clé dans l'objet `value` doit correspondre **exactement** au type de champ de l'annotation. Si vous utilisez la mauvaise clé (par exemple `text` pour un champ `integerNumber`), vous recevrez une erreur "L'annotation n'est pas de type attendu".
{% endhint %}

{% hint style="info" %}
Vous pouvez modifier plusieurs annotations en une seule mutation en ajoutant plusieurs objets dans le tableau `annotations`. Cela permet d'optimiser le nombre d'appels API.
{% endhint %}

{% hint style="info" %}
Vous pouvez voir la documentation complète de cette mutation ici : [https://www.demarches-simplifiees.fr/graphql/schema/index.html#definition-DossierModifierAnnotationsPayload](https://www.demarches-simplifiees.fr/graphql/schema/index.html#definition-DossierModifierAnnotationsPayload)
{% endhint %}
