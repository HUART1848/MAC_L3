---
author: Olivier D'Ancona & Hugo Huart & Nelson Jeanrenaud
title: "MAC - Labo 3 : Indexing and Search with Elasticsearch"

fontsize: 12pt
geometry: margin=1.5cm
output: pdf_document
toc: true
---

\pagebreak
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

\pagebreak
## D.1

API requests to create **`cacm_standard`**:

#### Mappings:

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

#### Reindex:

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

\pagebreak
## D.2

API requests to create **`cacm_termvector`**:

#### Mappings:

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

#### Reindex:

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

\pagebreak
## D.3

API request to query a term vector:

```
GET /cacm_termvector/_termvectors/gNa1ZYAB7VfE5TWZZFs7
```

`gNa1ZYAB7VfE5TWZZFs7` being the ID of a document that has a `summary` field

## D.4

The official documentation of Elasticsearch describes a term vector as the following:

>_Term vectors contain information about the terms produced by the analysis process, including:_

>* _A list of terms._
>* _The position (or order) of each term._
>* _The start and end character offsets mapping the term to its origin in the original string._
>* _Payloads (if they are available) — user-defined binary data associated with each term position._

## D.5

Sizes of the indexes:

* `cacm_raw` : **1.34MB**
* `cacm_standard` : **1.48MB**
* `cacm_termvector` : **2.07MB**

\pagebreak
# 2.3 Reading Index

## D.6

Using the following request, we observe that **Thacher Jr., H. C.** is the author
with the highest number of publications. He has **38** publications.

#### Request:

```json
GET /cacm_standard/_search
{
  "aggs": {
    "genres": {
      "terms": {
        "field": "author",
        "size": 1
      }
    }
  }
}
```

## D.7

Using the following request, we observe that the top 10 terms are:

1. _of_
2. _algorithm_
3. _a_
4. _for_
5. _the_
6. _and_
7. _in_
8. _on_
9. _an_
10. _computer_

#### Request:

```json
GET /cacm_standard/_search
{
  "aggs": {
    "genres": {
      "terms": {
        "field": "title",
        "size": 10
      }
    }
  }
}
```

\pagebreak
# 2.4 Using different Analyzers

## D.8

The following requests create indexes with the required analyzers.

#### `whitespace` analyzer

```json
PUT /cacm_standard_whitespace
{
  "settings": {
    "analysis": {
      "analyzer": "whitespace"
    }
  },
  "mappings": {
    "properties": {
      "id":{"type": "unsigned_long"},
      "author": {"type": "keyword"},
      "title":{"type": "text", "fielddata": true},
      "date":{"type": "date"},
      "summary":{"type": "text", "fielddata" : true}
    }
  }
}

POST _reindex
{
  "source": {
    "index": "cacm_raw"
  },
  "dest": {
    "index": "cacm_standard_whitespace",
    "pipeline": "my_pipeline"
  }
}
```

\pagebreak
#### `english` analyzer

```json
PUT /cacm_standard_english
{
  "settings": {
    "analysis": {
      "analyzer": "english"
    }
  },
  "mappings": {
    "properties": {
      "id":{"type": "unsigned_long"},
      "author": {"type": "keyword"},
      "title":{"type": "text", "fielddata": true},
      "date":{"type": "date"},
      "summary":{"type": "text", "fielddata" : true}
    }
  }
}

POST _reindex
{
  "source": {
    "index": "cacm_raw"
  },
  "dest": {
    "index": "cacm_standard_english",
    "pipeline": "my_pipeline"
  }
}
```

\pagebreak
#### `standard` analyzer with shingles of size 1 and 2

```json
PUT /cacm_standard_myanalyzer1
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer1": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "custom_shingle"
          ]
        }
      },
      "filter": {
        "custom_shingle": {
          "type": "shingle",
          "max_shingle_size": 2
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "unsigned_long"
      },
      "author": {
        "type": "keyword"
      },
      "title": {
        "type": "text",
        "fielddata": true
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true
      }
    }
  }
}

POST _reindex
{
  "source": {
    "index": "cacm_raw"
  },
  "dest": {
    "index": "cacm_standard_myanalyzer1",
    "pipeline": "my_pipeline"
  }
}
```

\pagebreak
#### `standard` analyzer with shingles of size 3

```json
PUT /cacm_standard_myanalyzer2
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer2": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "custom_shingle"
          ]
        }
      },
      "filter": {
        "custom_shingle": {
          "type": "shingle",
          "min_shingle_size": 3,
          "max_shingle_size": 3
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "unsigned_long"
      },
      "author": {
        "type": "keyword"
      },
      "title": {
        "type": "text",
        "fielddata": true
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true
      }
    }
  }
}

POST _reindex
{
  "source": {
    "index": "cacm_raw"
  },
  "dest": {
    "index": "cacm_standard_myanalyzer2",
    "pipeline": "my_pipeline"
  }
}
```

\pagebreak
#### `stop` analyzer

```json
PUT /cacm_standard_stopwords
{
  "settings": {
    "analysis": {
      "analyzer": {
        "stopwords": {
          "tokenizer": "whitespace",
          "filter": [ "custom_stopwords" ]
        }
      },
      "filter" : {
        "custom_stopwords" : {
          "type" : "stop",
          "stopwords_path" : "data/common_words.txt"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "unsigned_long"
      },
      "author": {
        "type": "keyword"
      },
      "title": {
        "type": "text",
        "fielddata": true
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true
      }
    }
  }
}

POST _reindex
{
  "source": {
    "index": "cacm_raw"
  },
  "dest": {
    "index": "cacm_standard_stopwords",
    "pipeline": "my_pipeline"
  }
}
```

## D.9

Explanation of the analyzers, according to the Elasticsearch documentation:

* `whitespace` : Breaks text into terms whenever a whitespace is encountered.
* `english` : Targeted for english text. It features relevant stop words,
plural to singular conversion and other similar language-specific filters.
* `standard` with shingles of size 1 and 2 : Produce shingles (or word n-gram) up to a size of two

  The text `"I Love MAC"` would produce `["I", "I Love", "Love", "Love MAC", "MAC"]`.
* `standard` with shingles of size 3 only : Produce shingles (or word n-gram) of size 3.

  The text `"I Love MAC"` would produce `["I", "I Love MAC", "Love", "MAC"]`.
* `stop` : Uses a list of words as stop words that will be removed from the the requested text

## D.10

