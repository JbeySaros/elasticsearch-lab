# Requêtes Query DSL — `supervision-serveurs`

> Résultats obtenus sur les 30 documents indexés (`2-mise-en-place-projet.md`, Partie B). Chaque requête est suivie de son résultat réel.

---

## Étape 2 — Recherche

### 1. `match` sur `commentaire`

```bash
curl -s -X POST "http://localhost:9200/supervision-serveurs/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{ "query": { "match": { "commentaire": "disque" } } }'
```

**Résultat : 1 hit**

```json
{
  "hits": {
    "total": { "value": 1 },
    "hits": [
      {
        "_id": "16",
        "_source": {
          "hostname": "srv-db-01",
          "statut": "critique",
          "disque_percent": 92.0,
          "commentaire": "Disque proche saturation, verifier purge des logs"
        }
      }
    ]
  }
}
```

> `commentaire` est un champ `text` analysé : `match` tokenise "disque" et le retrouve peu importe la phrase exacte, contrairement à `term` qui aurait renvoyé 0 résultat sur ce champ.

---

### 2. `term` sur un champ `keyword`

```bash
curl -s -X POST "http://localhost:9200/supervision-serveurs/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{ "query": { "term": { "hostname": "srv-db-01" } } }'
```

**Résultat : 6 hits** (les 6 relevés de `srv-db-01`, cohérent avec 5 serveurs × 6 relevés = 30).

---

### 3. `range` sur une métrique

```bash
curl -s -X POST "http://localhost:9200/supervision-serveurs/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{ "query": { "range": { "cpu_percent": { "gte": 80 } } } }'
```

**Résultat : 3 hits**

| `_id` | `hostname` | `cpu_percent` |
|---|---|---|
| 4 | srv-web-01 | 91.5 |
| 26 | srv-batch-01 | 85.0 |
| 28 | srv-batch-01 | 95.0 |

---

### 4. `bool` : `filter` combinés (`must`/`filter`)

```bash
curl -s -X POST "http://localhost:9200/supervision-serveurs/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "bool": {
        "filter": [
          { "term": { "environnement": "production" } },
          { "term": { "role": "web" } },
          { "range": { "cpu_percent": { "gt": 70 } } }
        ]
      }
    }
  }'
```

**Résultat : 2 hits**

| `_id` | `hostname` | `cpu_percent` | `statut` |
|---|---|---|---|
| 4 | srv-web-01 | 91.5 | critique |
| 10 | srv-web-02 | 77.0 | attention |

> Les trois critères sont des critères binaires (égalité ou seuil), pas de la pertinence textuelle à scorer : `filter` est adapté (pas de calcul de score, mis en cache).

---

### 5. `bool` : exclusion `must_not`

```bash
curl -s -X POST "http://localhost:9200/supervision-serveurs/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{ "query": { "bool": { "must_not": [ { "term": { "environnement": "staging" } } ] } } }'
```

**Résultat : 24 hits** (30 − 6 relevés `srv-batch-01`, le seul serveur en `staging`).

---

## Étape 3 — Agrégations

### 6. Agrégation metric seule

```bash
curl -s -X POST "http://localhost:9200/supervision-serveurs/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{ "size": 0, "aggs": { "cpu_stats": { "stats": { "field": "cpu_percent" } } } }'
```

**Résultat :**

```json
{
  "aggregations": {
    "cpu_stats": {
      "count": 30,
      "min": 10.0,
      "max": 95.0,
      "avg": 41.6333,
      "sum": 1249.0
    }
  }
}
```

---

### 7. Agrégation bucket seule

```bash
curl -s -X POST "http://localhost:9200/supervision-serveurs/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{ "size": 0, "aggs": { "par_role": { "terms": { "field": "role" } } } }'
```

**Résultat :**

| `role` | `doc_count` |
|---|---|
| web | 12 |
| base-de-donnees | 6 |
| cache | 6 |
| traitement-batch | 6 |

---

### 8. Agrégation imbriquée : CPU moyen par rôle

```bash
curl -s -X POST "http://localhost:9200/supervision-serveurs/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{
    "size": 0,
    "aggs": {
      "par_role": {
        "terms": { "field": "role" },
        "aggs": { "cpu_moyen": { "avg": { "field": "cpu_percent" } } }
      }
    }
  }'
```

**Résultat :**

| `role` | `doc_count` | `cpu_moyen` |
|---|---|---|
| web | 12 | 42.50 |
| base-de-donnees | 6 | 58.50 |
| cache | 6 | 27.00 |
| traitement-batch | 6 | 37.67 |

> Sans surprise : la base de données tourne en moyenne plus chargée que le cache ; c'est le rôle `web` qui porte le plus de documents (2 serveurs web sur 5).

---

### 9. Recherche + agrégation combinées

```bash
curl -s -X POST "http://localhost:9200/supervision-serveurs/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{
    "size": 0,
    "query": { "bool": { "filter": [ { "term": { "environnement": "production" } } ] } },
    "aggs": {
      "par_serveur": {
        "terms": { "field": "hostname" },
        "aggs": { "disque_moyen": { "avg": { "field": "disque_percent" } } }
      }
    }
  }'
```

**Résultat (parmi les 24 relevés `production` uniquement) :**

| `hostname` | `doc_count` | `disque_moyen` |
|---|---|---|
| srv-web-01 | 6 | 42.33 |
| srv-web-02 | 6 | 39.00 |
| srv-db-01 | 6 | 71.17 |
| srv-cache-01 | 6 | 21.00 |

> `srv-db-01` ressort nettement au-dessus des autres serveurs de production (disque moyen 71%, avec un pic à 92% le 21/07 — voir le hit `_id: 16` de la requête `match` ci-dessus) : c'est le serveur à surveiller en priorité côté stockage.

---

## Compléments utiles pour le RAPPORT.md (bonus, non exigés mais utiles à citer)

- `terms` sur `statut` : `ok` = 24, `attention` = 3, `critique` = 3 (confirme le point de contrôle de la mise en place).
- Nombre de relevés `critique` par serveur : `srv-web-01` = 1, `srv-db-01` = 1, `srv-batch-01` = 1 (aucun sur `srv-web-02` ni `srv-cache-01`, qui n'ont que des `attention`).
