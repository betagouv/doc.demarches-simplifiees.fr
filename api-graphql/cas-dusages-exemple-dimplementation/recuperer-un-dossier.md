---
description: >-
  Exemple de cas d'usage : vous souhaitez remonter les dossiers unitairement
  dans votre SI, ex en laissant un usager saisir son numero de dossier DS.
---

# Récupérer un dossier

Exemple d'implémentation pour récuperer les informations d'un dossier (dans notre cas, nous remontons uniquement quelques informations : le status du dossier, différentes dates liées a l'instruction de celui ci, l'id et le nom de la démarche et enfin les messages echangés entre l'administration et l'usager).

{% code title="get_dossier.rb" %}
```ruby
require 'net/http'
require 'uri'
require 'json'

ENDPOINT = URI('https://www.demarches-simplifiees.fr/api/v2/graphql')
QUERY_DOSSIER = "
query getDossier($dossierNumber: Int!) {
  dossier(number: $dossierNumber) {
    id
    number
    state
    dateDerniereModification
    dateDepot
    datePassageEnConstruction
    datePassageEnInstruction
    dateTraitement
    dateExpiration
    dateSuppressionParUsager
    usager {
      email
    }
    demarche {
      title
      id
    }
    messages {
      id
      email
      body
      createdAt
    }
  }
}
"
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
def get_dossier(http)
  # the data of our query
  data = {
    "query" => QUERY_DOSSIER,
    "operationName" => "getDossier",
    "variables" => {
      "dossierNumber": ENV.fetch('DOSSIER_NUMBER') { raise 'missing env var DOSSIER_NUMBER' }.to_i
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
data = get_dossier(http)
puts "Info: dossier #{data}"

```
{% endcode %}

Vous pouvez tester en executant le script suivant avec les variables d'environnement adaptées :

<pre class="language-bash"><code class="lang-bash"><strong>API_TOKEN="votre_token" DOSSIER_NUMBER=votre_numero_de_demarche ruby get_dossier.rb
</strong></code></pre>

{% hint style="info" %}
Ce type d'appel à l'API permet une **intégration très légère de DS dans votre application**. Un exemple que nous avons croisé étant de laisser l'utilisateur saisir son numéro de dossier, puis vous pouvez ainsi remonter l'état d'avancement de son dossier, le nom de la démarche etc, le dernier message.&#x20;
{% endhint %}

{% hint style="info" %}
Aussi vous pouvez renvoyer l'usager sur DS en reconstruisant le lien web vers son dossier en respectant le schema d'url suivant : https://www.demarches-simplifiees.fr/dossiers/${id\_du\_dossier}
{% endhint %}
