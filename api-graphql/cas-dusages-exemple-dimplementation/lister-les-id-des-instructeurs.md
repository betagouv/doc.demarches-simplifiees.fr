# Lister les Id des instructeurs

Pour lister la liste des instructeurs d'une démarche, vous pouvez passez par l'operation getDemarches, voici un exemple :

```bash
API_TOKEN="" DEMARCHE_NUMBER="" ruby ./list_instructeur.rb
```

{% code title="list_instructeur.rb" %}
```ruby
require 'net/http'
require 'uri'
require 'json'

ENDPOINT = URI('https://www.demarches-simplifiees.fr/api/v2/graphql')
QUERY_INSTRUCTEURS = "
query getDemarche(
  $demarcheNumber: Int!
  $includeInstructeurs: Boolean = true
) {
  demarche(number: $demarcheNumber) {
    id
    number
    groupeInstructeurs @include(if: $includeInstructeurs) {
      ...GroupeInstructeurFragment
    }
  }
}

fragment GroupeInstructeurFragment on GroupeInstructeur {
  id
  number
  label
  instructeurs @include(if: $includeInstructeurs) {
    id
    email
  }
}
"
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
    "Authorization" => "Bearer #{ENV.fetch('API_TOKEN') { raise 'missing env var API_TOKEN' }}"
  }
end

# given an http connexion, request the API for page
def list_instructeurs(http)
  # the data of our query
  data = {
    "query" => QUERY_INSTRUCTEURS,
    "operationName" => "getDemarche",
    "variables" => {
      "demarcheNumber": ENV.fetch('DEMARCHE_NUMBER') { raise 'missing env var DEMARCHE_NUMBER' }.to_i,
      "includeInstructeurs": true
    }
  }
  # continue pagination
  req = Net::HTTP::Post.new(ENDPOINT, request_headers)
  req.body = data.to_json

  response = http.request(req)

  data = JSON.parse(response.body)
  data
end

http = open_http_connection

# check if we persisted a cursor so we continue polling
data = list_instructeurs(http)
puts "Info: fetched instructeurs ids: #{data}"
```
{% endcode %}

{% hint style="info" %}
L'id de l'instructeur est a utilisé la ou il est demandé, exemple, sur une mutation comme `dossierEnvoyerMessage`
{% endhint %}

