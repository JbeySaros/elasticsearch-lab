# Supervision Elasticsearch/Kibana — projet de fin de formation

## Contexte

Ce projet met en situation un administrateur systèmes et opérations (ASO) junior chargé de superviser une petite infrastructure : deux serveurs web, une base de données et un cache en production, plus un serveur de traitement par lots en environnement de test (staging). Les relevés système (CPU, mémoire, disque) de ces 5 serveurs sur 3 jours sont déjà indexés dans Elasticsearch (index `supervision-serveurs`, 30 documents), mais rien n'est encore exploité au départ : pas de recherche, pas d'agrégation, pas de dashboard.

## Objectif

Transformer ces documents JSON bruts en un outil de supervision exploitable : interroger les données via l'API Elasticsearch (Query DSL), les résumer via des agrégations, puis construire dans Kibana une exploration (Discover), des visualisations et un dashboard interactif, avant de gérer le cycle de vie de l'index (Index Management, ILM).

## Ce que couvre le projet

1. **Vérification des données** : contrôle du mapping (`_mapping`) et des comptages (documents, serveurs, statuts).
2. **Requêtes Query DSL** : `match` (recherche texte libre), `term` (valeur exacte), `range` (intervalle numérique), `bool` combiné (`must`/`filter`/`must_not`).
3. **Agrégations** : metric seule, bucket seule, bucket + metric imbriquée, et requête combinant `query` + `aggs`.
4. **Kibana — Discover** : data view sur `@timestamp`, requêtes KQL, filtres cliquables.
5. **Kibana — Visualisations** : au moins 3 visualisations (pie, bar, line...) assemblées en un dashboard nommé, avec filtre global et interaction (drill-down).
6. **Administration** : consultation d'Index Management (health, docs count, storage size) et création d'une policy ILM (phases Hot + Delete).

## Livrables

- `requetes.md` : toutes les requêtes Query DSL et agrégations exécutées, avec leurs résultats réels.
- `RAPPORT.md` : synthèse et interprétation du travail, captures d'écran Kibana à l'appui (dossier `captures/`).

## Cadre

Travail individuel, réalisé en local sur un cluster Elasticsearch local sans authentification (pas de bonus de sécuritée activée (flemme))
