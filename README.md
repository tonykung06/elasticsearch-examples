## Sample elasticsearch DSL
GET /_cat/indices

DELETE /my_blog

POST /my_blog
{
  "mappings": {
    "post": {
      "properties": {
        "user_id": {
          "type": "integer"
        },
        "post_text": {
          "type": "string"
        },
        "post_date": {
          "type": "date"
        }
      }
    }
  }
}

POST /my_blog
{
  "mappings": {
    "post": {
      "_source": {
        "enabled": false
      },
      "properties": {
        "user_id": {
          "type": "integer",
          "store": true
        },
        "post_text": {
          "type": "string"
        },
        "post_date": {
          "type": "date"
        }
      }
    }
  }
}


POST /my_blog
{
  "settings": {
    "index": {
      "number_of_shards": 5
    }
  },
  "mappings": {
    "post": {
      "properties": {
        "user_id": {
          "type": "integer"
        },
        "post_text": {
          "type": "string"
        },
        "post_date": {
          "type": "date",
          "format": "YYY-MM-DD"
        }
      }
    }
  }
}

GET /my_blog/_mapping

POST /my_blog/post
{
  "user_id": 1,
  "post_date": "2016-12-27",
  "post_text": "testing content"
}

POST /my_blog/post
{
  "user_id": 1,
  "post_date": "2016-12-26T21:00:01",
  "post_text": "another testing content"
}

POST /my_blog/post
{
  "user_id": 2,
  "post_date": "2016-12-26",
  "post_text": "yet another testing content"
}

GET /my_blog/post/_search

GET /my_blog/post/lP4xgpXHQoO5iuBMvlqLjA

POST /my_blog/post/1
{
  "user_id": 2,
  "post_date": "2016-12-26",
  "post_text": "testing blog post with our own id"
}

GET /my_blog/post/1?fields=user_id,post_text


# routing and aliases

POST /eventlog-2016-12-27
{
  "mappings": {
    "event": {
      "properties": {
        "error": {
          "type": "string"
        }
      }
    }
  }
}

POST /eventlog-2016-12-26
{
  "mappings": {
    "event": {
      "properties": {
        "error": {
          "type": "string"
        }
      }
    }
  }
}

POST /eventlog-2016-12-27/event
{
  "error": "my error"
}

POST /eventlog-2016-12-26/event
{
  "error": "my error 2"
}

GET /eventlog-2016-12-27/event/_search

GET /eventlog-2016-12-27/_search

POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "eventlog-2016-12-27",
        "alias": "eventlog"
      }
    }
  ]
}

POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "eventlog-2016-12-26",
        "alias": "eventlog"
      }
    }
  ]
}

GET /eventlog/_search

GET /eventlog/event/_search

# doing queries

DELETE /my_blog

POST /my_blog
{
  "mappings": {
    "post": {
      "properties": {
        "user_id": {
          "type": "integer"
        },
        "post_text": {
          "type": "string"
        },
        "post_date": {
          "type": "date"
        },
        "post_word_count": {
          "type": "integer"
        }
      }
    }
  }
}

POST /my_blog/post
{
  "user_id": 1,
  "post_date": "2016-12-27",
  "post_text": "testing content",
  "post_word_count": 2
}

POST /my_blog/post
{
  "user_id": 1,
  "post_date": "2016-12-27",
  "post_text": "another testing content",
  "post_word_count": 3
}

POST /my_blog/post
{
  "user_id": 2,
  "post_date": "2016-12-27",
  "post_text": "yet another testing content",
  "post_word_count": 4
}

POST /my_blog/post
{
  "user_id": 2,
  "post_date": "2016-12-26",
  "post_text": "yet another testing content on another date",
  "post_word_count": 4
}


# Lite Query

GET /my_blog/post/_search?q=post_text:testing
GET /my_blog/post/_search?q=post_text:yet

# using elasticsearch query Domain Specific Language

GET /my_blog/post/_search
{
  "query": {
    "match": {
      "post_text": "yet another omg ooooooo"
    }
  }
}

# find exact sequence of words
# highlighting
GET /my_blog/post/_search
{
  "query": {
    "match_phrase": {
      "post_text": "yet another testing content"
    }
  },
  "highlight": {
    "fields": {
      "post_text": {}
    }
  }
}

GET /my_blog/post/_search
{
  "query": {
    "filtered": {
      "filter": {
        "range": {
          "post_date": {
            "gt": "2016-12-26"
          }
        }
      },
      "query": {
        "match_phrase": {
          "post_text": "another testing content"
        }
      }
    }
  }
}

GET /my_blog/post/_search
{
  "query": {
    "filtered": {
      "filter": {
        "term": {
          "user_id": "2"
        }
      },
      "query": {
        "match_phrase": {
          "post_text": "another testing content"
        }
      }
    }
  }
}

# aggregation

GET /my_blog/post/_search
{
  "query": {
    "match": {
      "post_text": "testing"
    }
  },
  "aggs": {
    "my_all_words": {
      "terms": {
        "field": "post_text"
      }
    },
    "my_avg_word_count": {
      "avg": {
        "field": "post_word_count"
      }
    }
  }
}

# test analyzer
POST _analyze?analyzer=standard
{
  "body": "Convert the title-case text using the ToLower(string) command."
}

POST _analyze?analyzer=whitespace
{
  "body": "Convert the title-case text using the ToLower(string) command."
}

DELETE /my_blog

POST /my_blog
{
  "mappings": {
    "post": {
      "properties": {
        "user_id": {
          "type": "integer"
        },
        "post_text": {
          "type": "string",
          "analyzer": "standard"
        },
        "post_date": {
          "type": "date"
        },
        "post_word_count": {
          "type": "integer"
        }
      }
    }
  }
}

POST /my_blog
{
  "mappings": {
    "post": {
      "properties": {
        "user_id": {
          "type": "integer",
          "index": "not_analyzed"
        },
        "post_text": {
          "type": "string",
          "analyzer": "whitespace"
        },
        "post_date": {
          "type": "date"
        },
        "post_word_count": {
          "type": "integer",
          "index": "not_analyzed"
        }
      }
    }
  }
}

POST /my_blog/post
{
  "user_id": 1,
  "post_date": "2016-12-27",
  "post_text": "Convert the title-case text using the ToLower(string) command.",
  "post_word_count": 8
}

GET /my_blog/post/_search
{
  "query": {
    "term": {
      "post_text": "ToLower(string)"
    }
  }
}

GET /my_blog/post/_search
{
  "query": {
    "term": {
      "post_text": "tolower"
    }
  }
}

GET /my_blog/post/_search
{
  "query": {
    "match": {
      "post_text": "title-case"
    }
  },
  "aggs": {
    "my_all_words": {
      "terms": {
        "field": "post_text"
      }
    },
    "my_avg_word_count": {
      "avg": {
        "field": "post_word_count"
      }
    }
  }
}









