---
description: >-
  Exemple de cas d'usage : votre démarche est aujourd'hui close et vous
  souhaitez récuperer les informations des dossiers pour faire de l'analyse de
  donnée.
---

# Pagination – Récupérer tous les dossiers d'une démarche

{% hint style="info" %}
Avez-vous pris connaissance de notre mechanisme de [pagination](../pagination.md) ?
{% endhint %}

Pour paginer vos requetes, il faut ajouter les curseurs à votre requete GraphQL, Pour ce faire ajouter le `PageInfoFragment` et déclarez les variables de pagination sur la ressource paginée Ex :

```graphql
query getDemarche(
    $demarcheNumber: Int!
    $order: Order
    $first: Int
    $after: String
    $includeDossiers: Boolean = true
  ) {
    demarche(number: $demarcheNumber) {
      dossiers(
        order: $order
        first: $first
        after: $after
      ) @include(if: $includeDossiers) {
        pageInfo {
          ...PageInfoFragment
        }
        nodes {
          ...DossierFragment
        }
        pageInfo {
          endCursor
          hasNextPage
        }
      }
    }
  }

  fragment DossierFragment on Dossier {
    id
  }

  fragment PageInfoFragment on PageInfo {
    hasPreviousPage
    hasNextPage
    endCursor
  }
```

Vous pouvez tester en executant le script suivant avec les variables d'environnement adaptées :

<pre class="language-bash"><code class="lang-bash"><strong>API_TOKEN="votre_token" DEMARCHE_NUMBER=votre_numero_de_demarche ruby poller.rb
</strong></code></pre>

Pour faciliter la lecture du code, la query GraphQL est fournie en PJ

{% file src="../../.gitbook/assets/getDemarche.samplePagination.graphql" %}

Ensuite, vous pouvez executer ce code ruby qui :&#x20;

1. cherche a retrouver le dernier curseur que vous avez utiliser
2. si il est présent, le ré-utilise pour requeter la page suivante (sinon il repart a 0)
3. requete la page
4. sauve le curseur&#x20;

Vous pouvez ainsi executer le script plusieurs fois pour parcourir toutes les pages de dossier de votre démarche

{% code title="poller.rb" %}
```ruby
require 'net/http'
require 'uri'
require 'json'


ENDPOINT = URI('https://www.demarches-simplifiees.fr/api/v2/graphql')
CURSOR_STORAGE = 'lastCursor'
EMPTY_CURSOR = {}

### that's the GraphQL part.
# We store the query in a flat file because it's easier to read
def query
  File.read("getDemarche.samplePagination.graphql")
end

### that's the cursor / pagination part.
# We store the cursor in a .json file because it's easier for the demo
# Each demarche can have it's own cursor for continuous/batch polling
def cursor_file_path(demarche_number)
  "#{EMPTY_CURSOR}-#{demarche_number}.json"
end

# when we emit a request, we try to reuse last persisted cursor
def retrieve_last_persisted_cursor(demarche_number)
  JSON.parse(File.read(cursor_file_path(ENV.fetch('DEMARCHE_NUMBER') { raise 'missing env var DEMARCHE_NUMBER' }))) || {}
rescue Errno::ENOENT # first call, file was never written
  puts "Info: first time using cursor on #{demarche_number}"
  EMPTY_CURSOR
rescue JSON::ParserError # strange parse error
  puts "Warning: strange case, check your cursor data"
  EMPTY_CURSOR
end

# when we receive a response, we try keep the cursor for next call
def persist_last_cursor(response, demarche_number)
  cursor = response.dig("data", "demarche", "dossiers", "pageInfo")
  if cursor['endCursor'] && !cursor['hasNextPage']
    puts "end of cursor not yet reached, persist for next call: #{cursor.inspect}"
    File.write(cursor_file_path(demarche_number), JSON.dump(cursor.to_h), mode: 'w')
  else
    puts 'end of cursor reached, do not persist nil endCursor otherwise restart full listing'
  end
end

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
def request_page(http, last_cursor)
  # the data of our query
  data = {
    "query" => query,
    "operationName" => "getDemarche",
    "variables" => {
      "demarcheNumber": ENV.fetch('DEMARCHE_NUMBER') { raise 'missing env var DEMARCHE_NUMBER' }.to_i,
      "first": 100
    }
  }
  # continue pagination
  data['variables']['after'] = last_cursor['endCursor'] if last_cursor

  req = Net::HTTP::Post.new(ENDPOINT, request_headers)
  req.body = data.to_json

  response = http.request(req)

  data = JSON.parse(response.body)
  data
end

http = open_http_connection

# check if we persisted a cursor so we continue polling
last_cursor = retrieve_last_persisted_cursor(ENV['DEMARCHE_NUMBER'])
data = request_page(http, last_cursor)
persist_last_cursor(data, ENV['DEMARCHE_NUMBER'])
dossiers = data.dig('data', 'demarche', 'dossiers', 'nodes')
puts "Info: fetched dossiers ids: #{dossiers.map { _1['number'] }.join(', ')}"
puts "Debug: #{data.inspect}"

```
{% endcode %}

{% hint style="warning" %}
Tant qu'il n'y aura pas de nouvelle page a proposer, votre curseur renvera les même dossiers de la dernière page. Pensez a gérer l'idempotence de votre implementation pour ce cas las.
{% endhint %}

