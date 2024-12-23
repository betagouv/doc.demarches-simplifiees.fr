---
description: Présentation du fonctionnement de l'horodatage
---

# Horodatage

## But : garantir l'intégrité des données

### Fonctionnement vulgarisé

Afin de prouver l'intégrité des dossiers, chaque jour, demarches-simplifiees.fr fait une photo de tous les dossiers modifiés. On envoie ensuite l'empreinte de cette photo a un tiers de confiance qualifié qui nous renvoie un jeton d'horodatage déclarant : \
« Moi, Tiers de Confiance, je certifie que demarches-simplifiees.fr m'a envoyé cette empreinte à cette date ».

### En quoi cela prouve-t-il quoi que soit ?

Le jeton d'horodatage est inaltérable et son auteur peut être identifié. On est donc sûr que l'empreinte de la photo a bien été signée à telle date.\
\
Si on veut vérifier qu'un dossier n'a pas été altéré depuis son traitement :\
\- on prend la prend la photo qui correspond à la dernière modification du dossier\
\- on s'assure que le dossier n'a pas changé depuis cette photo\
\- finalement on vérifie l'intégrité de la photo en calculant son empreinte et en la comparant à l'empreinte enregistré dans le jeton d'horodatage

### C'est quoi cette histoire d'empreinte ?

Voir [https://fr.wikipedia.org/wiki/Fonction\_de\_hachage](https://fr.wikipedia.org/wiki/Fonction\_de\_hachage). En simplifié c'est une opération mathématique qui à partir d'un fichier fournit toujours le même grand nombre (empreinte) de sorte que :\
\- pour 2 fichiers différents il y a toujours 2 empreintes différentes\
\- il n'est pas possible de reconstruire le fichier à partir de son empreinte

## Fonctionnement dans le détail

### Les opérations

À chaque changement d'état du dossier, demarches-simplifiees.fr crée un fichier contenant \
\- le numéro de dossier\
\- la date\
\- l'action\
\- la valeur des champs du dossier

{% code title="procedure-xxx/dossier-yyy/horodatage/operation-zzz-date.json" overflow="wrap" %}
```
{
  "operation": "accepter",
  "dossier_id": 1017534,
  "author": {
    "id": "Instructeur#741",
    "email": "instructeur@beta.gouv.fr"
  },
  "subject": {
    "id": "RG9zc2llci0xMDE3NTM0",
    "number": 1017534,
    "archived": false,
    "state": "accepte",
    "dateDerniereModification": "2021-06-11T15:50:52+02:00",
    "datePassageEnConstruction": "2019-10-28T10:07:04+01:00",
    "datePassageEnInstruction": "2019-10-28T10:08:10+01:00",
    "dateTraitement": "2021-06-11T15:50:52+02:00",
    "instructeurs": [
      {
        "email": "instructeur@beta.gouv.fr"
      }
    ],
    "groupeInstructeur": {
      "label": "défaut"
    },
    "champs": [
      {
        "id": "Q2hhbXAtNDA5MDA=",
        "label": "un premier champ",
        "stringValue": ""
      }
    ],
    "annotations": [],
    "avis": [],
    "demandeur": {
    "civilite": "M",
      "nom": "Nom",
      "prenom": "Prénom",
      "dateDeNaissance": "2019-10-22"
    },
    "motivation": "",
    "motivationAttachment": null,
    "revision": {
      "id": "UHJvY2VkdXJlUmV2aXNpb24tMTMzOA=="
    }
  },
  "automatic_operation": false,
  "executed_at": "2021-06-11T15:50:52+02:00"
}
```
{% endcode %}

L'empreinte de ce fichier est `b193534bd4e8f2f32841e6010286ed98be6dc8e24f89dbd7fae0d5e4496cbcc2`

### L'empreinte des opérations du jour

Toute les nuits, demarches-simplifiees.fr crée un fichier avec toutes les empreintes des opérations de la journée. Il se présente sous le nom demarches-simplifiees-operations-date.json

{% code title="procedure-xxx/bills/horodatage/demarches-simplifiees-operations-date.json" overflow="wrap" %}
```
{
  "1296796": "b193534bd4e8f2f32841e6010286ed98be6dc8e24f89dbd7fae0d5e4496cbcc2",
  "1283268": "8718c89dcf5eff30099b40b8b144c2b3efb51156e9e4263645250723d78cb6ae"
  ...
}
```
{% endcode %}

On retrouve ici l'empreinte de notre opération.

### L'horodatage de l'empreinte des opérations du jour

demarches-simplifiees.fr fait horodater ce fichier par un service d'horodatage qualifié par l'ANSSI. Le jeton d'horodatage retourné est présent ici :\
`procedure-xxx/bills/horodatage/demarches-simplifiees-signature-date.der`

### Preuve de l'intégrité de vos dossiers

Pour un dossier donné :\
\- vérifier visuellement la cohérence entre la version pdf et le dernier fichier opération de dossier\
\- puis lancer le script suivant pour vérifier l'horodatage du fichier `command.sh dossier/horodatage/operation`



```
#! /bin/bash

set -o errexit
set -o pipefail
set -o nounset

if [ $# -eq 0 ]
  then
    echo "usage: ./command.sh operation_file"
  exit 1
fi

OPERATION_FILE="${1:-}"
echo -e "\nChecking $OPERATION_FILE\n"

HASH=$(sha256sum $OPERATION_FILE | cut -d ' ' -f1)
echo "HASH: $HASH"

# find matching files
BILL=$(grep -l $HASH bills/horodatage/*)
echo "Bill: $BILL"

DATE=$(echo $BILL| sed 's/.*demarches-simplifiees-operations-//' | sed 's/-.....json//')

TOKEN=$(find . -iname "*signature*$DATE*")
echo -e "Token: $TOKEN\n"

# retrieve provider certificates
mkdir -p store
cd store

## Certigna
curl -sLO http://autorite.certigna.fr/ACcertigna.crt
curl -sLO http://autorite.certigna.fr/entityca.crt

## Universign
curl -sLO https://www.universign.com/documents/certificates/root-ca.pem
curl -sLO https://www.universign.com/documents/certificates/tsa-root-2019.pem
curl -sLO https://www.universign.com/documents/certificates/tsa-root-2015.pem

cd ../

# final verification
openssl ts -verify -CAfile <(cat store/*) -data $BILL -in $TOKEN -token_in
```

Certains horodatages émis par Universign ne sont pas compatibles avec openssl,\
`rsa routines:RSA_padding_check_PKCS1_type_1:invalid padding`\
\
Ils doivent les vérifier eux-même en attendant de mettre à disposition une API. Contactez nous si besoin.

Astuce pour inspecter le contenu d'un jeton :\
`openssl asn1parse -in token -inform der`
