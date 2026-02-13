---
description: >-
  Exemple d'implémentation pour réaffecter un dossier à un autre groupe
  d'instructeurs au sein d'une même démarche.
---

# Changer le groupe instructeur d'un dossier

Lorsqu'une démarche comporte plusieurs groupes d'instructeurs, il peut être nécessaire de réaffecter un dossier d'un groupe à un autre. Cette mutation permet de changer le groupe instructeur responsable du traitement d'un dossier.

{% hint style="info" %}
Pour récupérer la liste des groupes instructeurs d'une démarche et leurs identifiants, consultez la documentation [lister-les-id-des-instructeurs.md](lister-les-id-des-instructeurs.md "mention").
{% endhint %}

Vous pouvez tester en executant le script suivant avec les variables d'environnement adaptées :

<pre class="language-bash"><code class="lang-bash"><strong>API_TOKEN="votre_token" DOSSIER_ID="l_id_du_dossier" GROUPE_INSTRUCTEUR_ID="l_id_du_groupe_instructeur" ruby changeGroupeInstructeur.rb
</strong></code></pre>

{% code title="changeGroupeInstructeur.rb" %}
```ruby
# frozen_string_literal: true

require 'net/http'
require 'uri'
require 'json'
ENDPOINT = URI('https://www.demarches-simplifiees.fr/api/v2/graphql')

### that's the GraphQL part.
QUERY = <<~GRAPHQL_QUERY
  mutation dossierChangerGroupeInstructeur($input: DossierChangerGroupeInstructeurInput!) {
    dossierChangerGroupeInstructeur (input: $input) {
      dossier {
        id
        groupeInstructeur {
          id
          label
        }
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
def request_dossier_changer_groupe_instructeur(http)
  # the data of our query
  data = {
    "query" => QUERY,
    "operationName" => "dossierChangerGroupeInstructeur",
    "variables" => {
      "input": {
        dossierId: ENV.fetch('DOSSIER_ID') { raise 'missing env var DOSSIER_ID' },
        groupeInstructeurId: ENV.fetch('GROUPE_INSTRUCTEUR_ID') { raise 'missing env var GROUPE_INSTRUCTEUR_ID' }
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
data = request_dossier_changer_groupe_instructeur(http)

puts "Info: dossier reassigned to groupe instructeur: #{data}"

```
{% endcode %}

{% hint style="info" %}
La mutation retourne le dossier avec son nouveau groupe instructeur. Vous pouvez voir la documentation complète de cette mutation ici : [https://www.demarches-simplifiees.fr/graphql/schema/index.html#definition-DossierChangerGroupeInstructeurPayload](https://www.demarches-simplifiees.fr/graphql/schema/index.html#definition-DossierChangerGroupeInstructeurPayload)
{% endhint %}
