---
description: >-
  Exemple de cas d'usage : votre démarche a moins de 100 changements sur ses
  dossiers par jour et vous souhaitez synchroniser tous les dossiers dans votre
  SI.
---

# Pagination – Synchroniser une démarche à faible volumétrie (polling simple)

Nos API renvoient jusqu'à 100 dossiers maximum par requête. De fait, si il y a moins de 100 dossiers modifiés par jours sur votre démarche, une tache planifiée (ex quotidienne) récupérant tous les derniers dossiers modifiées vous permettra de suivre tous les dossiers.

Ainsi il est plus intéressant d'éviter la compléxité d'un mechanisme de pagination au profit d'un mechanisme de polling.

<pre class="language-bash"><code class="lang-bash"><strong>API_TOKEN="votre_token" DEMARCHE_NUMBER=votre_numero_de_demarche ruby get_last_updated_dossiers.rb
</strong></code></pre>

{% code title="get_last_updated_dossiers.rb" %}
```ruby
require 'date'
require 'net/http'
require 'uri'
require 'json'

ENDPOINT = URI('https://www.demarches-simplifiees.fr/api/v2/graphql')

### that's the GraphQL part.
QUERY = <<~QUERY
query getDemarche(
    $demarcheNumber: Int!
    $last: Int
    $after: String
    $updatedSince: ISO8601DateTime
    $includeDossiers: Boolean = true
  ) {
    demarche(number: $demarcheNumber) {
      dossiers(
        last: $last
        after: $after
        updatedSince: $updatedSince
      ) @include(if: $includeDossiers) {
        pageInfo {
          ...PageInfoFragment
        }
        nodes {
          id
          dateDerniereModification
        }
        pageInfo {
          endCursor
          hasNextPage
        }
      }
    }
  }

  fragment PageInfoFragment on PageInfo {
    hasPreviousPage
    hasNextPage
    endCursor
  }
QUERY

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

# given an http connexion, request the API for page
def request_page(http)
  # the data of our query
  data = {
    "query" => QUERY,
    "operationName" => "getDemarche",
    "variables" => {
      "demarcheNumber": ENV.fetch('DEMARCHE_NUMBER') { raise 'missing env var DEMARCHE_NUMBER' }.to_i,
      "last": 1, # récupérer le derniers, le sens de la pagination
      "updatedSince": Date.new(2020, 2, 22).iso8601 # récuperer les dossiers par ordre de derniere date de mise a jour
    }
  }

  req = Net::HTTP::Post.new(ENDPOINT, request_headers)
  req.body = data.to_json

  response = http.request(req)
  data = JSON.parse(response.body)
  data
end

http = open_http_connection

# check if we persisted a cursor so we continue polling
data = request_page(http)
dossiers = data.dig('data', 'demarche', 'dossiers', 'nodes')
puts "Info: fetched dossiers ids: #{dossiers.map { _1['number'] }.join(', ')}"
puts "Debug: #{data.inspect}"

```
{% endcode %}

{% hint style="warning" %}

{% endhint %}
