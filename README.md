# Kibana Query to Find Calling Applications

This document contains an Elasticsearch DSL query to find all applications that are calling applicationA using RequestUUID correlation.

## Query Purpose
The query is designed to:
1. Find all applications that are calling applicationA
2. Use RequestUUID correlation to track the request flow
3. Exclude applicationA from the results to show only the calling applications

## Elasticsearch DSL Query

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "REQUESTTYPE": "operation1"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "cls.application": "applicationA"
          }
        }
      ],
      "filter": {
        "terms": {
          "RequestUUID": {
            "index": "your_index_name",
            "id": "your_index_name",
            "path": "RequestUUID",
            "query": {
              "bool": {
                "must": [
                  {
                    "match": {
                      "cls.application": "applicationA"
                    }
                  },
                  {
                    "match": {
                      "REQUESTTYPE": "operation1"
                    }
                  }
                ]
              }
            }
          }
        }
      }
    }
  },
  "aggs": {
    "calling_applications": {
      "terms": {
        "field": "cls.application",
        "size": 10000
      },
      "aggs": {
        "request_uuids": {
          "terms": {
            "field": "RequestUUID",
            "size": 10000
          }
        }
      }
    }
  },
  "size": 0
}
```

## Query Explanation

### Query Components:
1. **Main Query**:
   - Matches documents with REQUESTTYPE "operation1"
   - Excludes documents from applicationA
   - Filters for RequestUUIDs that are also used by applicationA

2. **Aggregations**:
   - Groups results by calling applications
   - For each calling application, shows the RequestUUIDs used
   - Sets size limits to 10000 to ensure comprehensive results

### Expected Output Structure:
```json
{
  "aggregations": {
    "calling_applications": {
      "buckets": [
        {
          "key": "applicationB",
          "doc_count": 5,
          "request_uuids": {
            "buckets": [
              {
                "key": "uuid1",
                "doc_count": 3
              }
            ]
          }
        }
      ]
    }
  }
}
```

## Usage Instructions
1. Replace "your_index_name" with your actual Elasticsearch index name
2. Adjust the size parameters if needed based on your data volume
3. Modify REQUESTTYPE "operation1" if looking for different operation types

## Notes
- The query excludes applicationA from the results to show only calling applications
- Results are grouped by calling application for better visibility
- Each calling application shows its associated RequestUUIDs and request counts
