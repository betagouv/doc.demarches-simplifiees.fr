# Envoyer un message avec une PJ

Il est possible d'envoyer un message via les API GraphQL. Mais avant cela, voici un petit tour d'horizon du fonctionnement :&#x20;

## 3 étapes :

### 1. Vous demandez a notre **API une authorisation pour uploader un fichier sur notre object storage**.&#x20;

Cette requete implique de décrire le fichier que vous allez envoyer : le filename, byteSize, checksum (un digest md5, base64digesté) et son contenu. Ceci pour nous permettre de valider que nous echangeons le meme fichier, qu'il n'a pas été altéré etc.. Nous vous renvoyer&#x20;

1. les crédentials pour communiquer avec notre object storage
2. l'identifiant du fichier (blob\_signed\_id) a utiliser dans une autre requete pour le lier à une autre mutation&#x20;

### 2. Vous uploadez le fichier sur notre object storage, en réutilisant les crédentials de la 1ere requete&#x20;

<figure><img src="../../.gitbook/assets/evil-martions-direct-upload-architecture.webp" alt=""><figcaption><p>résumé des étapes 1 et 2,source: <a href="https://evilmartians.com/chronicles/active-storage-meets-graphql-direct-uploads">https://evilmartians.com/chronicles/active-storage-meets-graphql-direct-uploads</a></p></figcaption></figure>

### 3. Vous, client, faites une requete pour lier ce fichier (maintenant sur nos serveurs, identifié par le signed\_blob\_id) a un message



## 1ere étape : demander les crédentials

Utiliser la mutation `createDirectUpload`. Voici un exemple complet de script que vous pouvez executer :

{% code overflow="wrap" %}
```bash
FILE=./file.txt \
FILE_TYPE="text/plain" \
API_TOKEN="VOTRE_TOKEN" \
DOSSIER_ID="un identifiant de dossier, cf: dossiers.id (ce n'est pas le numero de dossier)" \
ruby ./get_credentials.rb 

```
{% endcode %}

{% code title="get_credentials.rb" %}
```ruby
require 'net/http'
require 'uri'
require 'json'
require 'byebug'
ENDPOINT = URI('https://www.demarches-simplifiees.fr/api/v2/graphql')

QUERY_UPLOAD_REQUEST = "
mutation createDirectUpload($input: CreateDirectUploadInput!) {
  createDirectUpload(input: $input) {
    directUpload {
      url
      headers
      blobId
      signedBlobId
    }
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
    "Authorization" => "Bearer #{ENV.fetch('API_TOKEN') { raise 'missing env var API_TOKEN=xxx' }}"
  }
end

def compute_checksum_in_chunks(io)
  Digest::MD5.new.tap do |checksum|
    while (chunk = io.read(5242880))
      checksum << chunk
    end

    io.rewind
  end.base64digest
end

def request_direct_upload_credentials(http, file)
  data = {
    "query" => QUERY_UPLOAD_REQUEST,
    "operationName" => "createDirectUpload",
    "variables" => {
      "input" => {
        "dossierId" => ENV.fetch("DOSSIER_ID") { raise "missing env var DOSSIER_ID=xxx" },
        "filename" => file.path,
        "byteSize" => file.size,
        "checksum" => compute_checksum_in_chunks(file),
        "contentType" => ENV.fetch("FILE_TYPE") { raise 'missing file type' }
      }
    }
  }

  req = Net::HTTP::Post.new(ENDPOINT, request_headers)
  req.body = data.to_json
  http.request(req)
end

fp = ENV.fetch('FILE') { raise 'missing env var FILE' }
File.open(fp, 'r') do |file|
  http = open_http_connection
  response = request_direct_upload_credentials(http, file)
  credentials = JSON.parse(response.body)['data']['createDirectUpload']['directUpload']
  credentials_headers = JSON.parse(credentials['headers'])

  puts <<~CURL
    curl -vvv \
    --data-binary @#{file.path} \
    -X PUT \
    -H "Content-Type: #{credentials_headers['Content-Type']}" \
    -H "ETag: #{credentials_headers['ETag']}" \
    "#{credentials['url']}"
  CURL
end

```
{% endcode %}

{% hint style="info" %}
Vous pouvez voir la documentation des parametres de la mutation graphQL ici : [https://www.demarches-simplifiees.fr/graphql/schema/index.html#definition-CreateDirectUploadInput](https://www.demarches-simplifiees.fr/graphql/schema/index.html#definition-CreateDirectUploadInput)
{% endhint %}

La réponse HTTP de notre API sera de la forme suivante

```json
{
   "data":{
      "createDirectUpload":{
         "directUpload":{
            "url":"https://static.demarches-simplifiees.fr:443/v1/AUTH_XXX/ds_activestorage_backup/2023/12/21/We/We9FmNxKuJfzEr6QSmKWSrxP263Z?temp_url_sig=XXX\u0026temp_url_expires=XXX",
            "headers":"{\"Content-Type\":\"image/png\",\"ETag\":\"d5d122f320e9d34faf716390a33429a7\"}",
            "blobId":"a number in a string",
            "signedBlobId":"a string"
         }
      }
   }
}
```



{% hint style="info" %}
Vous pouvez voir la documentation de la réponse GraphQL ici : [https://www.demarches-simplifiees.fr/graphql/schema/index.html#definition-DirectUpload](https://www.demarches-simplifiees.fr/graphql/schema/index.html#definition-DirectUpload).&#x20;
{% endhint %}

Aussi notre script echo un exemple pour uploader le fichier via `curl`, il vous suffit de le copier/coller pour envoyer le même fichier sur notre object storage

### 2eme étape : Uploader le fichier en utilisant les crédentials

```
curl -vvv --data-binary @./file.txt -X PUT -H "Content-Type: LE type de votre fichier" -H "ETag: LAVALEUR PRESENTE DANS LA REPONSE, au niveau de headers.etag [attention, c'est un json a parser]" "data[createDirectUpload][directUpload][url]"
```

### 3eme étape : Associer ce fichier lors d'un envoie de message à un usager.

En préambule, il vous faut envoyer ce message au nom d'un instructeur, nous vous renvoyons à la documentation pour lister les id des instructeurs

```ruby
// Some code
```

