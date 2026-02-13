---
description: >-
  Exemple d'implémentation pour créer un nouveau groupe d'instructeurs au sein
  d'une démarche et y affecter des instructeurs.
---

# Créer un groupe instructeur

Lorsque vous gérez une démarche avec plusieurs services ou équipes, vous pouvez créer des groupes d'instructeurs pour organiser le traitement des dossiers. Cette mutation permet de créer un nouveau groupe et d'y affecter des instructeurs dès sa création.

{% hint style="info" %}
Le numéro de la démarche est celui visible dans l'URL de la démarche : `https://www.demarches-simplifiees.fr/administrateurs/procedures/{numero}`
{% endhint %}

Vous pouvez tester en executant le script suivant avec les variables d'environnement adaptées :

<pre class="language-bash"><code class="lang-bash"><strong>API_TOKEN="votre_token" DEMARCHE_NUMBER="numero_de_la_demarche" GROUPE_LABEL="Nom du groupe" INSTRUCTEUR_EMAIL="email@instructeur.fr" ruby creerGroupeInstructeur.rb
</strong></code></pre>

{% code title="creerGroupeInstructeur.rb" %}
```ruby
# frozen_string_literal: true

require 'net/http'
require 'uri'
require 'json'
ENDPOINT = URI('https://www.demarches-simplifiees.fr/api/v2/graphql')

### that's the GraphQL part.
QUERY = <<~GRAPHQL_QUERY
  mutation groupeInstructeurCreer($input: GroupeInstructeurCreerInput!) {
    groupeInstructeurCreer(input: $input) {
      groupeInstructeur {
        id
        number
        label
        closed
        instructeurs {
          id
          email
        }
      }
      warnings {
        message
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
def creer_groupe_instructeur(http)
  # the data of our query
  data = {
    "query" => QUERY,
    "operationName" => "groupeInstructeurCreer",
    "variables" => {
      "input": {
        demarcheId: ENV.fetch("DEMARCHE_NUMBER") { raise 'missing env var DEMARCHE_NUMBER' }.to_i,
        label: ENV.fetch('GROUPE_LABEL') { raise 'missing env var GROUPE_LABEL' },
        closed: false,
        instructeurs: [
          {
            email: ENV.fetch('INSTRUCTEUR_EMAIL') { raise 'missing env var INSTRUCTEUR_EMAIL' }
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
data = creer_groupe_instructeur(http)

puts "Info: groupe instructeur créé: #{data}"

```
{% endcode %}

{% hint style="warning" %}
Si vous créez un groupe avec le paramètre `closed: true`, le groupe sera créé mais ne pourra pas recevoir de nouveaux dossiers. Utilisez `closed: false` pour un groupe actif.
{% endhint %}

{% hint style="info" %}
Vous pouvez ajouter plusieurs instructeurs dès la création en ajoutant des éléments dans le tableau `instructeurs`. La mutation retourne également des `warnings` en cas d'avertissements non bloquants (par exemple, si un email d'instructeur n'existe pas).
{% endhint %}

{% hint style="info" %}
Vous pouvez voir la documentation complète de cette mutation ici : [https://www.demarches-simplifiees.fr/graphql/schema/index.html#definition-GroupeInstructeurCreerPayload](https://www.demarches-simplifiees.fr/graphql/schema/index.html#definition-GroupeInstructeurCreerPayload)
{% endhint %}
