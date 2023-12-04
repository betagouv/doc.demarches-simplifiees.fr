# Récupérer tous les dossiers (pagination)

Exemple d'implémentation pour lister tous les dossiers d'une démarche avec la pagination

Vous pouvez executer ce script en lançant la commande suivante dans votre terminal

```bash
API_TOKEN="votre_token" DEMARCHE_NUMBER=votre_numero_de_demarche ruby poller.rb
```

Pour faciliter la lecture du code, la query GraphQL est fournie en PJ

{% file src="../../.gitbook/assets/getDemarche.graphql" %}

Ensuite, vous pouvez executer ce code ruby

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
  File.read("getDemarche.graphql")
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
  # http.use_ssl = true
  # http.verify_mode = OpenSSL::SSL::VERIFY_NONE
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
begin
  # check if we persisted a cursor so we continue polling
  last_cursor = retrieve_last_persisted_cursor(ENV['DEMARCHE_NUMBER'])
  data = request_page(http, last_cursor)
  persist_last_cursor(data, ENV['DEMARCHE_NUMBER'])

  dossiers = data.dig('data', 'demarche', 'dossiers', 'nodes')
  puts "Info: fetched dossiers ids: #{dossiers.map{ _1['number'] }.join(', ')}"
rescue StandardError => e
  puts "Error: #{e.inspect}"
  http.close if !http.closed?
end
```
{% endcode %}
