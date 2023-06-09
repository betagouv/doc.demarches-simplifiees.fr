---
description: Stabilité et bonnes pratiques
---

# Exports et Macros

Les scripts (macros) sont un moyen très efficace pour gagner du temps. Néanmoins, ils nécessitent des mises jour fréquentes, en particulier lorsque le format de la feuille change (et il changera).

## Pourquoi les colonnes changent-elles ?

1. Si une nouvelle version de la démarche est publiée
2. Suite à une évolution applicative

## Comment faire pour que mon script ~~ne~~ casse ~~pas~~ moins ?

1. Utilisez les noms de colonne au lieu des index

## Mais j'ai vraiment besoin que mon script marche tout le temps !

1. Laissez tomber l'export excel, et utilisez l'[api](graphql.md)
2. Gardez un bon contact / contrat avec le développeur des macros pour qu'il puisse les mettre à jour régulièrement
