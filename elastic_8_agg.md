

## Aggregations

bucket by rating value:
```bash
curl -u elastic:changeme -H 'Content-Type: application/json' -XGET 'http://89.233.105.243:9200/ratings/_search?size=0&pretty' -d '
{
	"aggs": {
		"ratings": {
			"terms": {
				"field": "rating"
			}
		}
	}
}'
```

count only 5-star ratings:
```bash
curl -u elastic:changeme -H 'Content-Type: application/json' -XGET 'http://89.233.105.243:9200/ratings/_search?size=0&pretty' -d '
{
  "query": {
    "match": {
      "rating": 5
    }
  },
  "aggs": {
    "ratings": {
      "terms": {
        "field": "rating"
      }
    }
  }
}'
```

average rating for Star Wars:
```bash
curl -u elastic:changeme -H 'Content-Type: application/json' -XGET 'http://89.233.105.243:9200/ratings/_search?size=0&pretty' -d '
{
  "query": {
    "match_phrase": {
      "title": "Star Wars Episode IV"
    }
  },
  "aggs": {
    "avg_rating": {
      "avg": {
        "field": "rating"
      }
    }
  }
}'
```

### histograms

display ratings by 1.0-rating intervals
```bash
curl -u elastic:changeme -H 'Content-Type: application/json' -XGET 'http://89.233.105.243:9200/ratings/_search?size=0&pretty' -d '
{
  "aggs": {
    "whole_ratings": {
      "histogram": {
        "field": "rating",
        "interval": 1
      }
    }
  }
}'
```

count up movies from each decade

```bash
curl -u elastic:changeme -H 'Content-Type: application/json' -XGET 'http://89.233.105.243:9200/movies/_search?size=0&pretty' -d '
{
  "aggs": {
    "release": {
      "histogram": {
        "field": "year",
        "interval": 10
      }
    }
  }
}'
```


### timeseries

break down website hits by hour:
```bash
curl -u elastic:changeme -H 'Content-Type: application/json' -XGET 'http://89.233.105.243:9200/kibana_sample_data_logs/_search?size=0&pretty' -d '
{
  "aggs": {
    "timestamp": {
      "date_histogram": {
        "field": "@timestamp",
        "fixed_interval": "24h"
      }
    }
  }
}'
```

when does Windows user visit me?
```bash
curl -u elastic:changeme -H 'Content-Type: application/json' -XGET 'http://89.233.105.243:9200/kibana_sample_data_logs/_search?size=0&pretty' -d '
{
  "query": {
    "match": {
      "agent": "Windows"
    }
  },
  "aggs": {
    "timestamp": {
      "date_histogram": {
        "field": "@timestamp",
        "fixed_interval": "1h"
      }
    }
  }
}'
```

Unique user agent
```bash
curl -u elastic:changeme -H 'Content-Type: application/json' -XGET 'http://89.233.105.243:9200/kibana_sample_data_logs/_search?size=0&pretty' -d '
{
  "size": 0,
  "aggs": {
    "unique_agent": {
      "composite": {
        "size": 1000,
        "sources": [
          {
            "agent": {
              "terms": {
                "field": "agent.keyword"
              }
            }
          }
        ]
      }
    }
  }
}'
```

### Nested aggregations

what's the average rating for each Star Wars movie?

```bash
curl -u elastic:changeme -H 'Content-Type: application/json' -XGET 'http://89.233.105.243:9200/ratings/_search?size=0&pretty' -d '
{
  "query": {
    "match_phrase": {
      "title": "Star Wars"
    }
  },
  "aggs": {
    "titles": {
      "terms": {
        "field": "title.keyword"
      },
      "aggs": {
        "avg_rating": {
          "avg": {
            "field": "rating"
          }
        }
      }
    }
  }
}'
```