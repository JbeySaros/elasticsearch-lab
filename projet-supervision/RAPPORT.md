   # Rapport de supervision : supervision-serveurs

   ## 1. Vérification des données
   (comptages obtenus, cohérents avec la mise en place : 30 documents, 5 serveurs, 3 statuts)

   ## 2. Requêtes Query DSL
   Pour chaque requête (match, term, range, bool must/filter, bool must_not) :
   - la question métier posée (ex : "quels serveurs sont en alerte critique ?")
   - la requête JSON
   - le résultat obtenu (nombre de hits + un exemple de document)

   ### Exemple
   **Question : quels relevés ont un CPU supérieur à 80% ?**
   - Requête : `{ "query": { "range": { "cpu_percent": { "gt": 80 } } } }`
   - Résultat : 3 documents (ids 4, 16... à adapter selon TA requête réelle)

   ## 3. Agrégations
   Pour chaque agrégation (metric, bucket, imbriquée, combinée à une query) :
   - la question métier posée (ex : "quel est le CPU moyen par rôle de serveur ?")
   - la requête JSON
   - le résultat (valeurs obtenues)

   ## 4. Kibana : exploration et visualisations
   - captures ou description du data view, de Discover (KQL testés)
   - captures des visualisations créées, avec leur configuration (bucket/metric utilisés)
   - capture du dashboard assemblé, du filtre global testé, de l'interaction testée

   ## 5. Administration
   - relevé Index Management (health, docs count, storage size)
   - policy ILM créée (nom, phases, seuils choisis) et pourquoi ces seuils sont raisonnables pour ce cas d'usage

   ## 6. Ce qui reste à faire / limites
   (honnêteté : ex. policy ILM créée mais non branchée à un index template + alias de rollover, comme vu en `<details>` du Jour 2 a-m)
