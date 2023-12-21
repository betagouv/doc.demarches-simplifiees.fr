---
description: >-
  Exemple d'implémentation pour lister tous les dossiers mis à jour sur votre
  démarche
---

# Synchroniser les dossiers modifiés sur ma démarche

{% hint style="info" %}
Avez-vous pris connaissance de notre mechanisme de [pagination](../pagination.md) ?
{% endhint %}

<pre class="language-bash"><code class="lang-bash"><strong>API_TOKEN="votre_token" DEMARCHE_NUMBER=votre_numero_de_demarche UPDATED_SINCE="iso-date, ex: 2020-02-22" ruby sync.rb
</strong></code></pre>

Pour faciliter la lecture du code, la query GraphQL est fournie en PJ

{% file src="../../.gitbook/assets/getDemarche.updatedSince.graphql" %}

Ensuite, vous pouvez executer ce code ruby

{% code title="sync.rb" %}
```ruby
require 'net/http'
require 'uri'
require 'json'
require 'byebug'
ENDPOINT = URI('https://www.demarches-simplifiees.fr/api/v2/graphql')
CURSOR_STORAGE = 'lastCursor'
EMPTY_CURSOR = {}

### that's the GraphQL part.
# We store the query in a flat file because it's easier to read
def query
  File.read("getDemarche.updatedSince.graphql")
end

### that's the cursor / pagination part.
# We store the cursor in a .json file because it's easier for the demo
# Each demarche can have it's own cursor for continuous/batch polling
def cursor_file_path(demarche_number)
  "#{EMPTY_CURSOR}-#{demarche_number}.json"
end

# when we emit a request, we try to reuse last persisted cursor
def retrieve_last_persisted_cursor(demarche_number)
  JSON.parse(File.read(cursor_file_path(ENV['DEMARCHE_NUMBER']))) || {}
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

  if cursor['endCursor'] && cursor['hasNextPage']
    puts "end of cursor not yet reached, persist for next call: #{cursor.inspect}"
    File.write(cursor_file_path(demarche_number), JSON.dump(cursor.to_h), mode: 'w')
  else
    puts "end of cursor reached, do not persist nil endCursor otherwise restart full listing: #{cursor.inspect}"
  end
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
def request_page(http, last_cursor)
  # the data of our query
  data = {
    "query" => query,
    "operationName" => "getDemarche",
    "variables" => {
      "demarcheNumber": ENV['DEMARCHE_NUMBER'].to_i,
      "updatedSince": ENV['UPDATED_SINCE'],
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

def log_dossiers_ids(ids)
  File.write('logsince-bis.ids', "\n#{ids.map { _1.join(',') }.join("\n")}", mode: 'a')
end

all_dossier_ids = []
http = open_http_connection
last_cursor = retrieve_last_persisted_cursor(ENV['DEMARCHE_NUMBER'])
last_cursor ||= {}
loop do
  # check if we persisted a cursor so we continue polling
  data = request_page(http, last_cursor)
  dossiers = data.dig('data', 'demarche', 'dossiers', 'nodes')
  dossier_ids = dossiers.map { [_1['number'], _1['dateDerniereModification']] }
  log_dossiers_ids(dossier_ids)
  all_dossier_ids.concat(dossier_ids)

  persist_last_cursor(data, ENV['DEMARCHE_NUMBER'])
  last_cursor = retrieve_last_persisted_cursor(ENV['DEMARCHE_NUMBER'])

  puts "Info: total count: #{all_dossier_ids.size}, fetched dossiers ids: #{dossiers.map { _1['number'] }.join(', ')}"
  break if !(last_cursor['endCursor'] && last_cursor['hasNextPage'])
end

```
{% endcode %}

{% hint style="warning" %}
Tant qu'il n'y aura pas de nouvelle page a proposer, votre curseur renvera les même dossiers de la dernière page. Pensez a gérer l'idempotence de votre implementation pour ce cas las.
{% endhint %}
