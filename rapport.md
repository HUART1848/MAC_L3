---
author: Olivier D'Ancona & Hugo Huart & Nelson Jeanrenaud
title: "MAC - Labo 3 : Indexing and Search with Elasticsearch"

documentclass: extarticle
fontsize: 20pt
geometry: margin=2cm
output: pdf_document
---

# 2.2 Indexing

Using the following pipeline:

```json
PUT _ingest/pipeline/my_pipeline

{
  "processors": [
    {
      "csv": {
        "field": "_row",
        "target_fields": [
          "id",
          "author",
          "title",
          "date",
          "summary"
        ],
        "separator": "\t",
        "quote": "§"
      }
    },
    {
      "split": {
        "field": "author",
        "separator": ";",
        "ignore_missing": true
      }
    },
    {
      "remove": {
        "field": "_row"
      }
    }
  ]
}
```

## D.1
API requests to create **`cacm_standard`**:

Mappings:

```json
PUT /cacm_standard

{
  "mappings": {
    "properties": {
      "author": {
        "type": "keyword"
      },
      "date": {
        "type": "date"
      },
      "id": {
        "type": "unsigned_long"
      },
      "summary": {
        "type": "text",
        "fielddata": true
      },
      "title": {
        "type": "text",
        "fielddata": true
      }
    }
  }
}
```


Reindex:

```json
POST _reindex

{
  "source": {
    "index": "cacm_raw"
  },
  "dest": {
    "index": "cacm_standard",
    "pipeline": "my_pipeline"
  }
}
```

## D.2

API requests to create **`cacm_termvector`**:

Mappings:

```json
PUT /cacm_termvector

{
  "mappings": {
    "properties": {
      "author": {
        "type": "keyword"
      },
      "date": {
        "type": "date"
      },
      "id": {
        "type": "unsigned_long"
      },
      "summary": {
        "type": "text",
        "term_vector": "with_positions"
      },
      "title": {
        "type": "text"
      }
    }
  }
}
```


Reindex:
```json
POST _reindex

{
  "source": {
    "index": "cacm_raw"
  },
  "dest": {
    "index": "cacm_termvector",
    "pipeline": "my_pipeline"
  }
}
```
