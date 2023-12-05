---
description: >-
  Exemple d'implémentation pour récupérer l'URL des fichiers des champs d'un
  dossier. Une fois ces URL récupérées vous pouvez les télécharger avec la
  méthode de votre choix.
---

# Télécharger les fichiers uploadé par un usager sur son dossier

{% hint style="info" %}
Si intéragissez sur une démarche a forte volumétrie, **nous recommandons de paralléliser / asynchroniser le téléchargement des fichiers**. En effet, un dossier peut avoir de nombreuses PJ, qui elle même peuvent peser jusqu'a 200Mo.
{% endhint %}

Pour ajouter les URLs des fichiers à votre code existant, Il vous faut ajouter le `FileFragment` au `ChampFragment` :

```graphql
  fragment ChampFragment on Champ {
    ... on PieceJustificativeChamp {
      files {
        ...FileFragment
      }
    }
   
  }
  
  fragment FileFragment on File {
    filename
    contentType
    checksum
    byteSize: byteSizeBigInt
    url
  }
```

Vous pouvez tester en executant le script suivant avec les variables d'environnement adaptées :

<pre class="language-bash"><code class="lang-bash"><strong>API_TOKEN="votre_token" DEMARCHE_NUMBER=votre_numero_de_demarche ruby downloader.rb
</strong></code></pre>

Pour faciliter la lecture du code, la query complète GraphQL est fournie en PJ

{% file src="../../.gitbook/assets/getDemarche.listOnlyFilesUrl.graphql" %}
query GraphQL minimaliste pour lister les type de champs Piece Justificative ainsi que les fichiers associéés
{% endfile %}

Ensuite, vous pouvez executer ce code ruby

{% code title="downloader.rb" %}
```ruby
require 'net/http'
require 'uri'
require 'json'
ENDPOINT = URI('https://www.demarches-simplifiees.fr/api/v2/graphql')

### that's the GraphQL part.
# We store the query in a flat file because it's easier to read
def query
  File.read("getDemarche.graphql")
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
      "demarcheNumber": ENV['DEMARCHE_NUMBER'].to_i,
      "first": 100
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

  dossiers = data.dig('data', 'demarche', 'dossiers', 'nodes')
  puts "Info: fetched dossiers ids: #{dossiers.map{ _1['number'] }.join(', ')}"

  all_champs = dossiers.flat_map { |dossier| dossier['champs'] }
  all_champs_pj = all_champs.filter { |champ| champ['files'] }
  all_url_to_download = all_champs_pj.flat_map { |pj| pj['files'].map{ |file| file['url'] } }

  puts "Info: now you can download each of those url: #{all_url_to_download.join(', ')}"
rescue StandardError => e
  puts "Error: #{e.inspect}"
  http.close if !http.closed?
end
```
{% endcode %}
