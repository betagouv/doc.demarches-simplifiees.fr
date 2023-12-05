---
description: Exemple d'implémentation pour requeter l'API en étant authentifié
---

# Autentification

{% hint style="info" %}
Avant de pouvoir envoyer des requêtes authentifiées, il vous faudra les[accreditation.md](../accreditation.md "mention")s nécessaire. Aussi pensez a consulter notre doc sur l'[jeton-dauthentification](../jeton-dauthentification/ "mention").
{% endhint %}

Vous pouvez tester en executant le script suivant avec les variables d'environnement adaptées :

<pre class="language-bash"><code class="lang-bash"><strong>API_TOKEN="votre_token" DEMARCHE_NUMBER=votre_numero_de_demarche ruby auth.rb
</strong></code></pre>

{% code title="auth.rb" %}
```ruby
require 'net/http'
require 'uri'
require 'json'
ENDPOINT = URI('https://www.demarches-simplifiees.fr/api/v2/graphql')

### that's the GraphQL part.
def query
  <<~GRAPHQL_QUERY
query getDemarche($demarcheNumber: Int!) {
  demarche(number: $demarcheNumber) {
    id
    dossiers {
      nodes {
        id
        demandeur {
          ... on PersonnePhysique {
            civilite
            nom
            prenom
          }
          ... on PersonneMorale {
            siret
          }
        }
      }
    }
  }
}
GRAPHQL_QUERY
end


### that's the HTTP part
# open an http connexion to our GraphQL endpoint
def open_http_connection
  http = Net::HTTP.new(ENDPOINT.host, ENDPOINT.port)
  http.use_ssl = true
  http.verify_mode = OpenSSL::SSL::VERIFY_NONE
  http
end

# the headers of our http query, include auth
def request_headers
  {
    "Content-Type" => "application/json",
    "Authorization" => "Bearer #{ENV['API_TOKEN']}"
  }
end

# given an http connexion, request the API for page
def request_page(http)
  # the data of our query
  data = {
    "query" => query,
    "operationName" => "getDemarche",
    "variables" => {
      "demarcheNumber": ENV['DEMARCHE_NUMBER'].to_i
    }
  }

  req = Net::HTTP::Post.new(ENDPOINT, request_headers)
  req.body = data.to_json

  response = http.request(req)

  data = JSON.parse(response.body)
  data
end

http = open_http_connection
begin
  # check if we persisted a cursor so we continue polling
  data = request_page(http)

  puts "Info: fetched it should work: #{data}"
rescue StandardError => e
  puts "Error: #{e.inspect}"
  http.close if !http.closed?
end

```
{% endcode %}
