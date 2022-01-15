---
layout: post
title: Elastic Quick Start Notes and Keywords Explained
subtitle: By Kaus
# cover-img: img/in-post/2022-01-13-elastic-search.png
header-img: img/in-post/2022-01-13-elastic-search.png
header-mask : 0.4
thumbnail-img: img/in-post/2022-01-13-ELASTICSEARCH.png
share-img: img/in-post/2022-01-13-elastic-search.png
tags: [elasticseach, notes, big data]
---

# Elastic Search Notes
[Original Github Link](https://github.com/codingexplained/complete-guide-to-elasticsearch/)

# Intro
Elasticsearch is an open-source and distributed analytics search engine. It is written in Java and was released in the year 2010. Elasticsearch is built on top of Lucene. It is a document-oriented engine that stores, searches, and analyzes data in large quantities. It is optimized to work well on huge data sets such that the search happens in near real-time.

##### Installing the Elasticseach 
Search the web : Lot of sites to understand process of Installing 

##### Some Basic Commands to Check Statuses

```
# GET health of Cluster
GET /_cluster/health

# Checking the shard distribution
GET /_cat/shards?v

# Listing the cluster's nodes
GET /_cat/nodes?v

# Listing the cluster's indices
GET /_cat/indices?v
```
##### Creating Index
```
PUT /sample_1
```
##### Adding Data to Index
```
POST /sample_1/_doc
{
  "name": "Kaustubh Pathak",
  "age" : "22"
}
```
##### Putting Data to specific ID

```
PUT /sample_1/_doc/id_here_can_be_anything
{
  "name": "sample_name",
  "age": "100"
}
```

##### Retrieving the ID from Index

```
GET /sample_1/_doc/id_here_can_be_anything
```
##### Update field from the ID

```
POST /sample_1/_update/id_here_can_be_anything
{
  "doc":{
    "name": "Sample Name"
  }
}
```
##### Update the new Field in ID

```
POST /sample_1/_update/id_here_can_be_anything
{
  "doc":{
    "variable": 100
  }
}
```
* Note - Elastic Internally never Update any value rather simply overwrite the Previous ID with new ID

##### Replace the Data completely using ID

```
PUT /sample_1/_doc/user102
{
  "name": "sample_user_4",
  "variable": 1000
}
```
##### Delete a Document

```
DELETE /sample_1/_doc/user102
```
# Scripting

##### Updating any variable by adding the Int to it
ctx - Context

```
POST /sample_1/_update/id_here_can_be_anything
{
  "script": {
    "source": "ctx._source.variable++"
  }
}
```

##### Passing the Parameters as Variable

```
POST /sample_1/_update/id_here_can_be_anything
{
  "script": {
    "source": "ctx._source.variable += params.quantity",
    "params": {
      "quantity": 4
    }
  }
}
```

##### Updating if exist otherwise Adding the Data

```
POST /sample_1/_update/user102
{
  "script": {
    "source": "ctx._source.variable++"
  },
  "upsert": {
    "name": "sample_user_3",
    "price": 100,
    "age": 12,
    "variable": 100
  }
}
GET /sample_1/_doc/user102
```



# Notes:
* Shards are allocated based on hashing of ID and divided by number of Shards reminder provide Shard number
* Thats the main reason the number of shards cant be increased once the index is created
* ARS - Adaptive Replica Selection decide the best Replica from Shards to get data
* Write Operation os forwarded to Primary Shard and then it passes the operation to other replicas in Parellel
_version, _seq_no, _primary_term help the Elasticsearch to update the Primary Shard and Maintain Update in All of its Replica Shards.
* As it also Maintain the Checkpoints which all Systems are completely Saved in all which are not checked yet.
* _version : Start at one and increamented everytime the ID is Updated. Everytime when deleted the Version is retailed for 60 seconds otherwise when updated afterwards will start with one.
* Used to Maintain the updated version and used in Lacegy version when Elasticsearch was used along with RDMS.
* Elasticsearch have option to update when certain version and SeqNo is Matched.

##### Optimistic Cuncurrency Control
seq_no and primary_no check are added to check before Updating

```
POST /sample_1/_update/user102?if_primary_term=1&if_seq_no=13
{
  "doc":{
    "in_stock": "True"
  }
}
```

##### Updating Multiple Queries
(All in this Case) in single Turn (Snapshot of complete data is taken and Conflicts are checked and if confirmed then it is executed)

```
POST /sample_1/_update_by_query
{
  "conflicts": "proceed",
  "script": {
    "source": "ctx._source.variable --",
    "lang": "painless"
  },
  "query": {
    "match_all": {}
  }
}
```
##### Condisional Deletion of Data (Delete all rows)

```
POST /sample_1/_delete_by_query
{
  "conflicts": "proceed",
  "query": {
    "match_all": {}
  }
}
```
# Multiple Updation and Deletion is achieved using Bulk API

```
POST /_bulk
{"index" : { "_index" : "sample_1" , "_id" : "user_200"}}
{ "name": "name 1" , "age": 10}
{"create": { "_index" : "sample_1" , "_id" : "user_201"}}
{ "name": "name 2" , "age": 15}
```

* Note: index vs create : Index Replace if exist but create will raise an error if already exist

Other Way of writing this is to enter Index along in URL

```
POST /sample_1/_bulk
{ "update" : { "_id": "user_200" }}
{ "doc" : {"number" : "9123921391"} }
{ "delete" : { "_id": "user_201"}}
```
* Notes : Content_Type should be application/x-ndjson
* Each line should have \n ending when sending json data with request
Last Line of file should be empty

##### Print All

```
GET /sample_1/_search
{
  "query":{
    "match_all": {}
  }
}
```
---
# Mapping and  Text Analyser
Each Text before input is processed by Analyer using
1. Character Filter - Removing Tags like \<b> from text and special characters before feeding it to Tokenizer.
2. Tokenizer  - Tokenizer convert Data to Tokens and convert it to indexes.
3. Token Filter - Filter the Data Generated by Tokenizer like Lowercasing the Data.

##### Testing Standard Analyser
Testing Standard Analyser in Elasticsearch & default setting of analyser

```
POST /_analyze
{
  "text": "So I was asked the past tense of 'think' in a English test today.....I thought and thought and thought and finally wrote 'thunk'.",
  "analyzer": "standard"
}

POST /_analyze
{
  "text": "So I was asked the past tense of 'think' in a English test today.....I thought and thought and thought and finally wrote 'thunk'.",
  "char_filter": [],
  "tokenizer": "standard",
  "filter": ["lowercase"]
}
```
##### Stemming Analyzer
Words are stemmed and then tokenized to reduce vocab size. (Both the Field data and search Query is analyzed in same format).

```
POST /_analyze
{
  "text": "So I was asked the past tense of 'think' in a English test today.....I thought and thought and thought and finally wrote 'thunk'.",
  "analyzer": "standard"
}
```
Others Analyzers are :
1. simple
2. whitespace
3. keyword
4. pattern

Analyzers can be configured according to language and to StopWords.

```
# Custom Analyzer
PUT /analyzer_test
{
  "settings":{
    "analysis": {
      "analyzer":{
        "custom_analyzer_1":{
          "type": "custom",
          "char_filter":["html_strip"],
          "tokenizer": "standard",
          "filter":[
            "lowercase",
            "stop",
            "asciifolding"
          ]
        }
      }
    }
  }
}
GET /analyzer_test/_analyze
{
  "analyzer":"custom_analyzer_1",
  "text": "Text Here"
}
```
##### Adding a Custom Analyzer to settings
When updating the settings its often better to Close the Index for Readnad Write and Open once the settings are reconfigured.

```
POST /analyzer_test/_close
PUT /analyzer_test/_settings
{
  "analysis":{
    "analyzer":{
      "custom_analyzer_2":{
        "type": "custom",
        "char_filter":["html_strip"],
        "tokenizer": "standard",
        "filter":[
          "lowercase",
          "stop",
          "asciifolding"
        ]
      }
    }
  }
}
POST /analyzer_test/_open
```

##### Updating all the Data rows with updated analyzer

```
POST /analyzer_test/_update_by_query?conflicts=proceed
```

##### Inverted Indices
* The Lookup table created for text field with Token Text as Key and Document in which Occur along with its Occurance Count
* It s Mapping between terms and which documents contain them
* The Different Table is used for every different filed in Document (One Inverted index per Text Field)
* Kept Sorted Alphabatically for faster Lookup
* Only used for Text. (For other Data Types, BKD tree type is used as it is much faster)
* Maintained by Apahe Lucene

### Mapping
* Mapping can be related as Schema in Relational Database
* Defined structure and How data is saved in Index
* 2 types of Mapping : <b>Explicit Mapping</b> (When field mapping is defined like MySQL Schema) and <b>Dynamic Mapping</b> (When Fields are allocated Randomly Like in NoSQL)
* Nested Object are flattened to Save the Data in Lucene as lucene doesn't support Object
* Mapping once defined cant be changed or updated  or removed.
* Since Dynamic mapping is automatic, this often lead to Waste of storage since its not always very efficient.

Different Types of Data Types in Mapping:

1. **object Type** : Any type of JSON Type. Although when flattened the data types are stacked and can cause error while searching.
2. **nested Type** : Much like object but items are stored as Document so not available for Searching.
3. **keyword Type** : Keyword are data type used when exact searching to be implemented rather than text Searching. Like in Tags or status.

##### Keywork Data Type
Since they are used for Exact searching, the Analyzer used for this is no-operation Analyzer which Doesn't modify Sentence. Rather it Convert Sentence to One Token without removing special characters and without Lowercasing.

```
POST /_analyze
{
  "text": "So I was asked the past tense of 'think' in a English test today.....I thought and thought and thought and finally wrote 'thunk'.",
  "analyzer": "keyword"
}
```
##### Coersion In Mapping
* Elasticsearch when indexing data check to convert wrong Datatype to Previously selected Mapped Datatype.
* Example: If Data type used is FLOAT and str is applied, coersion convert STR to Float and save it to create BKD Tree/Inverted Indices and save exact format to _source (Str in this case).
* Not used for Dynamic Mapping.

##### Array in Mapping
* Array are not supported in Lucene but rather they are merged as single string to create a Inverted Indices.
* Array should have same type of values.
* Nested Array are stacked in single Dimension for indexing
* Use type Nested for object of arrays to query the objects independently.

##### Creating Sample Custom Explicit Mapping

```
PUT /sample_2
{
  "mappings": {
    "properties": {
      "rating": {"type": "float"},
      "content": {"type": "text"},
      "product_id": {"type": "integer"},
      "author": {
        "properties": {
          "first_name": {"type": "text"},
          "last_name":{"type": "text"},
          "email" : {"type": "keyword"}
        }
      }
    }
  }
}
# Other Way Dot Notation Method
PUT /sample_2
{
  "mappings": {
    "properties": {
      "rating": {"type": "float"},
      "content": {"type": "text"},
      "product_id": {"type": "integer"},
      "author.first_name": { "type" : "text"},
      "author.last_name": { "type" : "text"},
      "author.email": { "type" : "keyword"}
    }
  }
}
```
##### Get Mapping of Index or Any Field of index

```
GET /sample_2/_mapping
GET /sample_2/_mapping/field/rating
```
##### Adding Field in Indices

```
PUT /sample_2/_mapping
{
  "properties":{
    "createdAt":{
      "type" : "date"
    }
  }
}
GET /sample_2/_mapping
```
### Date Format
Elasticsearch Support multiple time format including:
* Simple Date without time (2021-07-18)
* Date with time (2021-07-18T00:42:10Z) where Z define time zone.
* Date and Time with Timezone (2021-7-18T00:43:11+05:30) where 05:20 define timezone of country
* Unix Timestamp (Milliseconds since 1970-01-01)
* Date are stored as long format
* Formatted according to ISO 8601 standards.
* Coersion convert each of these format for indexing to standard time.

### Mapping Parameters
* **format** : Define the format of type like custom mapping of Date format like "dd/mm/yyyy" or "epoch_second"
* **properties** : Meta type of Format to used while defining Mapping
* **coerce** : Enable or Disable Coerce for Cell or for complete Index
* **doc_values** : A seperate table is also maintained apart from inverted indices that is reverse of it. It can be disabled from Mapping to save disk space and increase throughput time.
* **norms** : Normalisation is maintained to create relevance score of output when searching. Can be disabled for saving disk space when relevance is not important like email field.
* **index** : Index can be set to False and the field cant be used for searching but still be stored.
* **null_value** : To replace the explicit null values by default value while indexing enabling searching and indexing.It Should be of same type.
* **copy_to**: To copy field from Once to another. The new field will not be the part of _source
* Set doc_value to false if you dont need sorting, aggregation and Scripting.
* Set norms to false if you dont need relevence scoring
* Set index to false if you dont need filter on values.

```
PUT /sample_2
{
  "settings":{
    "index.mapping.coerce": false ## Setting coerce for all fields.
  }
  "mappings":{
    "properties":{
      "amount":{
        "type": "float",
        "coerce": true, ## Setting Coerce to True
        "doc_values": false, ## Setting doc_values to False
        "norms": false ,## Setting Norms to False
        "index": false ,## Setting Index to False
        "null_value": 1.0
      }
    }
  }
}
```
### Notes - Handling Missing Values
* Elasticsearch allow not setting any field even when Explicitly defined.
* Mapping doesnt make the field required.

##### Reindexing
Since Elasticsearch is not very elastic about remapping or deleteing or changing shards or reallocating. So its common practice to reindex to new mappings.
It work on Scroll API that is similar to Query API

```
POST /_reindex
{
  "source": {
    "index": "old_index",
    "_source": ["field_1" , "field_2" , "field_3"]
  },
  "dest":{
    "index": "new_index"
  }
  "script": {
    "source":"""
      # Rename the content to Comment
      ctx._source.comment = ctx._source.remove("content");
      # Dont Copy of Value is smaller
      if(ctx._source.rating < 4.0){
        ctx.op = "noop"
      }
    """
  }
}
```

##### Renaming without Reindexing using Alias
Use Alias to map one field to another

```
PUT /sample_1/_mapping
{
  "properties": {
    "field_one":{
      "type": "alias",
      "path": "renamed_field"
    }
  }
}
```
##### Mult-Indexing for the Fields
You can use multi-indexing onto one single field for better searching

```
PUT /sample_1
{
  "mappings":{
    "properties":{
      "field_one": {
        "type":"text",
        "fields": {
          "keyword":{ # Name of Alternative index and can be anything
            "type": "keyword" # Setting alternative index as Keyword
          }
        }
      }
    }
  }
}
```

##### INDEX Template
Templates can be created for similar Index so mapping and defining can be easier. They use pattern to define similar Indexes

```
PUT /_template/access-logs ## Index Template Name
{
  "index_patterns": ["access-logs-*"],
  "settings":{
    "number_of_shards": 2,
    "number_of_replicas": 3,
    "index.mapping.coerce": false
  }
  "mappings" : {
    "properties":{
      "field_1":{
        "type": "data"
      }
    }
  }
}
PUT /access-logs-2021
```

##### ECS
Elastic Common Schema is common standard Schema to be used when processing data from Different sources but have to be indexed in Common Field. ECS Process timestamp, logs, matrix from different popular sources and process them in way that they can be Processed and Plotted Properly using Kibana.

##### Configuring Dynamic mapping
Default behaviour of Mapping is set to True. Other options are :

1. True : Mapping will be create for new incoming fields and saved to Index.
2. False : New Field wil be saved to _source but mapping will not be created and Index will not be created resulting in ghosting while searching from Field.
3. Strict : Error will be raised when Seen new field.
4. For Production always set to Strict

```
PUT /sample_3
{
  "mappings": {
    "dynamic" : false,
    "numeric_detection": true, ## String will be converted to Int/Float when first encountered.
    "date_detection": true, ## Detect date when encountered
    "properties": {
      ...
    }
  }
}
```

##### Dynamic Templates
Templates can be build for defining the data type based on data encountered or name format.

```
## Simple Example
PUT /Dynaic_name
{
  "mappings":{
    "dynamic_templates":{
      "integers":{
        "match_mapping_type" : "long",
        "mapping": {
          "type": "integer"
        }
      },
      "strings":{
        "match_mapping_type": "string",
        "mapping":{
          "type": "text",
          "fields": {
            "keywords": {
              "type": "keyword",
              "ignore_above": 512
            }
          }
        }
      }
    }
  }
}
```

---
# Searching
* Request can end up to any of node in cluser and That node become Coordinator Node.
* Coordiantor Node than broadcast the request to other nodes in Cluster
* Coordinator node collect the data from all nodes merges them and response with response.
* Relevence is the field that contain the Score of Similarity between Search and Document.
* Earlier Elasticsearch used TF-IDF to calculate score. Now it use Okapi BM25
* BM25 uses Nonlinear Term Frequency Saturation to calucate Score. It is better in Stop words and improves Field - Length norm factor.
* Query vs Filter : Query calcualte the Score while Filter simply decide whether row should be selected or not.
* Term vs Match : when used match, query is passed thorugh Analyzer and then searched while in case of term, exact term is required

##### Searching with Request URI

```
# Matching all Documents
GET /lyrics_db/_search/?q=*

# Search any term in lyrics fields
GET /lyrics_db/_search/?q=lyrics:LOVE
```

##### Getting all the Match using Query DSL

```
GET /lyrics_db/_search
{
  "query": {
    "match_all": {}
  }
}
```

##### Explain how Relevance score is Calculated

```
GET /lyrics_db/_search
{
  "explain": true,
  "query": {
    "match": {
      "lyrics": "lobster"
    }
  }
}
```

##### Search VS Term

```
GET /lyrics_db/_search
{
  "explain": true,
  "query": {
    "match": {
      "lyrics": "lobster"
    }
  }
}
GET /lyrics_db/_search
{
  "explain": true,
  "query": {
    "term": {
      "lyrics": "lobster"
    }
  }
}
```

##### Other Searching Types

```
# Searching multiple Tags
GET /products/_search
{
  "query": {
    "terms": {
      "tags.keyword": [ "Soup", "Cake" ]
    }
  }
}
# Searching Multiple Tags
{
  "query": {
    "ids": {
      "values": [ 1, 2, 3 ]
    }
  }
}
# Get Between Range and formatting using giving format
{
  "query": {
    "range": {
      "created": {
        "gte": "01-01-2010",
        "lte": "31-12-2010",
        "format": "dd-MM-yyyy"
      }
    }
  }
}
# Rounding the Date 'now' by Month and subtracting one year and one day
{
  "query": {
    "range": {
      "created": {
        "gte": "2020/01/12||/M-1y"
        "lte": "now/M-1M"
      }
    }
  }
}
# Working and finding Non-Null values
{
  "query": {
    "exists": { # check if Field Exists.
      "field": "tags"
    },
    "prefix": { # Start with given Query
      "tags.keyword": "Vega"
    },
    "wildcard": { # Wildcard Help to search based on * and ? similar to Regex
      "tags.keyword": "Veg*ble"
    },
    "regexp": { # Regex Impression to search based on Regex Expression
      "tags.keyword": "Veg[a-zA-Z]+ble"
    }
  }
}
```

---
# Full Text Searching

##### Simple Searching
```
GET /sample_index/_search
{
  "query":{
    "match" : {
      "search_field" :{
        "query" : "item_1 item_2",
        "operator": "and" # Match all and rank based on relevance (Default is 'or')
      }
    }
  }
}
```

##### When Order of Occurance is important
```
GET /sample_index/_search
{
  "query": {
    "match_phrase": {
      "search_field": "puttanesca spaghetti"
    }
  }
}
```
##### Searching for Multiple Fields
```
GET /sample_index/_search
{
  "query":{
    "multi_match": {
      "query": "query_1 text_1",
      "fields": ["field_1" , "field_2"]
    }
  }
}
```

### Searching using Bool Operations

Match operations internally uses the bool query to perform operations. Bool have 3 basic queries :

1. must : query must be present inside the field to appear in results.
2. should : Query should be present inside the field and it help to decide the relevances Score of the results. (Not Necessary but good to have)
3. must_not : Query present here not not be present inside any of the result.
4. _name : is included inside match which help for debugging . Name Here help to debug and how they influence Relevance (more like _explain).

##### Simple Boolean Example for Searching

```
GET /sample_index/_search 
{
  "query" : {
    "bool" : {
      "must" : [ # Query must be Present inside Field.
        {
          "match" : {
            "field_to_search" : {
              "query" : "text_1",
              "_name" : "text_1_must_match_name" # Name Here help to debug and how they inflence Relevance (more like _explain)
            }
          }
        },
        {
          "match" : {
            "field_to_search" : {
              "query" : "text_2",
              "_name" : "text_2_must_match_name"
            }
          }
        }
      ],
      "must_not" : [ # Query must not be Present inside Field.
        {
          "match" : {
            "field_to_search" : {
              "query" : "text_3",
              "_name" : "text_3_must_not_match_name" 
            }
          }
        }
      ],
      "should" : [ # Query should be present inside Field. (Not necessary to contain but good to have)
        {
          "match" : {
            "field_to_search" : {
              "query" : "text_4 text_5 text_6",
              "_name" : "text_4_should_match_name" 
            }
          }
        }
      ],
      "filter" : [ # Filtering Tag
        {
          "range" : {
            "field_to_search" : {
              "lte" : 15,
		"get" : 5,
              "_name" : "lte_filter_range_name" 
            }
          }
        }
      ]
    }
  }
}
```

##### To Querying nested Object since they can't be directly accessed Use the command : 
**inner_hits**  is used to show only filter Query in Nested Object rather than complete Query

```
GET /Index_Here/_search
{
  "query" : {

    "nested" : {
      "path" : "Nested_Field_Here",
      "query" : {
        "bool": {
          ... # Query Here
        }
      }
    }
  }
}
```

### Join 
Much like in RDMS, rather than simply saving data in Object, you can create Join where they can merge different type of data.
You can also create Multi-Level Structure of data in Elasticsearch but this is not advised as this slows down there retrieval process.
Check this link for [Multi-Level Examples](https://github.com/codingexplained/complete-guide-to-elasticsearch/blob/master/Joining%20Queries/multi-level-relations.md)
Also study [Terms Lookup Mechanism](https://github.com/codingexplained/complete-guide-to-elasticsearch/blob/master/Joining%20Queries/terms-lookup-mechanism.md)

```
#  Creating the Join Structure in Elasticsearch and Inserting the Data in index.
PUT /sample_4
{
  "mappings" : {
      "properties": {
        "join_field_name" : {
          "type" : "join",
          "relations" : {
            "department" : "employee" # Define Relationship here..
          }
        }
    }
  }
}

PUT /sample_4/_doc/1
{
  "name" : "Development",
  "join_field_name" : "department"
}

PUT /sample_4/_doc/2
{
  "name" : "marketing",
  "join_field_name" : "department"
}

PUT /sample_4/_doc/3?routing=1 # routing is added to add relation to same shards as parent Cluster
{
  "name" : "kaustubh pathak",
  "gender" : "M",
  "join_field" : {
    "name" : "employee",
    "parent" : 1
  }
}

PUT /sample_4/_doc/4?routing=2 # routing is added to add relation to same shards as parent Cluster
{
  "name" : "Johnny Doe",
  "gender" : "F",
  "join_field" : {
    "name" : "employee",
    "parent" : 2
  }
}
```
##### Search inside Join using Parent ID or Parent  Keyword

```
GET /sample_4/_search
{
  "query": {
    "parent_id": {
      "type" : "employee",
      "id" : 1
    }
  }
}
# using Parent Keyword
GET /sample_4/_search
{
  "query": {
    "has_parent": {
      "parent_type": "department",
      #"has_child": {            ## Similar to the idea of Parent Searching this is Child based Searching
      #"type" : "employee",
      "score" : true,
      "query" : {
        "term" : {
          "name.keyword": "Development"
        }
      }
    }
  }
}
 ## Similar to the idea of Parent Searching this is Child based Searching
GET /sample_4/_search
{
  "query": {
    "has_child": {           
      "type" : "employee",
      "score_mode" : "sum", # This can be anything like sum, min, max, avg, none(Default) which decide the relevance score and its mode of calculations.
      "score" : true,
      "inner_hits": {}, # Get all the inner Hits when retrieving data 
      "query" : {
        "term" : {
          "name.keyword": "Development"
        }
      }
    }
  }
}
```
---
# Controlling Query Results
Getting Pretty results or YAML instead of JSON results output from Elasticsearch

```
GET /index_here/_search?format=yaml # Get results as YAML
GET /index_here/_search?pretty # Pretty the JSON Output
```
##### Controlling output using _source in nested Object with Size , Offsetand Pagination

```
GET /sample_index/_search 
{
  "size" : 10,
  "from" : 5
  "_source" : [
    "includes_" : "nested_1.*",
    "excludes_" : "nested_1.not_to_include"
  ],
  "query" : {
    ....
  },
  "sort" : [
    {
      "field_1" : "asc"
    },
    {
      "field_3" : "desc"
    }
  ]
}
```
---
# Aggregations
Aggregations much like in RDMS, we group the data and create statistics from it. There are 2 types of Agg. :

1. Single - Value Numeric Metric Aggregations
2. Multi - Value Numeric Metric Aggregations
Use global Aggregations when to Ignore filters or range.

```
# Some basic egg operations
GET /order/_search
{
  "size": 0,
  "aggs": {
    "total_sales": {  # Get Total Sum of the Field 
      "sum": {
        "field": "total_amount"
      }
    },
    "avg_sale": {     # Get average of the Field 
      "avg": {
        "field": "total_amount"
      }
    },
    "min_sale": {     # Get Minimum of the Field
      "min": {
        "field": "total_amount"
      }
    },
    "max_sale": {     # Get Maximum of the Field
      "max": {
        "field": "total_amount"
      }
    },
    "total_salesmen": {     # Get Unique Count of salesman ID in the Field
      "cardinality": {
        "field": "salesman.id"
      }
    },
    "values_count": {     # Get Count of the Amount 
      "value_count": {
        "field": "total_amount"
      }
    },
    "amount_stats": {     # Get All the stats from the field
     "stats": {
       "field": "total_amount"
     }
   },
  }
}
```
##### Now making Bucket in Search
Similar to value_counts in pandas, this return count of various values in searching. Note that each count is not very accurate and only approx due to shared nature of our Shards. 

```
GET /order/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "field_any",
        "size": 20, # Size of Max Bucket (Default is 10)
        "missing": "N/A", # Missing Default to considered
        "min_doc_count": 0, # Minimum value to count (Default is 1)
        "order": {
          "_key": "asc" # Order is Ascending rather than Descending
        },
	      {
          "aggs":{  ## This aggregation is ran in context to terms above, 
            "status_stats" : {
              "stats" : {
                "field" : "total_amount"
              }
            }
          }
        }
      }
    }
  }
}
```
##### You can also create different Filters / Ranges and create aggregations on top of each of them. 

```
# Like in this example we aggregate based on match we get after.
GET /recipe/_search
{
  "size": 0,
  "aggs": {
    "my_filter": {
      "filters": { # This filter will match with 2 Terms and create stats based on top of them
        "filters": {
          "pasta": {
            "match": {
              "title": "pasta"
            }
          },
          "spaghetti": {
            "match": {
              "title": "spaghetti"
            }
          }
        }
      },
      "aggs": {
        "avg_rating": {
          "avg": {
            "field": "ratings"
          }
        }
      }
    }
  }
}
# Creating a Range and Creating an stats on top of each Range
GET /order/_search
{
  "size": 0,
  "aggs": {
    "purchased_ranges": {
      "date_range": {
        "field": "purchased_at",
        "format": "yyyy-MM-dd", # Format to print Dates 
        "keyed": true, # Keys defined below will be used to display
        "ranges": [
          {
            "from": "2016-01-01",
            "to": "2016-01-01||+6M",
            "key": "first_half"
          },
          {
            "from": "2016-01-01||+6M",
            "to": "2016-01-01||+1y",
            "key": "second_half"
          }
        ]
      },
      "aggs": {
        "bucket_stats": {
          "stats": {
            "field": "total_amount" # Stats based on 2 parts of year
          }
        }
      }
    }
  }
}
```
##### Create Histogram Data over any field 

```
GET /order/_search
{
  "size": 0,
  "aggs": {
    "amount_distribution": {
      "histogram": {
        "field": "total_amount", # Create Histogram Data over this field
        "interval": 25, # Interval to considered when creating the Buckets 
        "min_doc_count": 0, # Minimum Document count to display
        "extended_bounds": { # Create False boundary to create Hist Data
          "min": 0,
          "max": 500
        }
      }
    }
  }
}
GET /order/_search
{
  "size": 0,
  "aggs": {
    "orders_over_time": {
      "date_histogram": { # Date Histogram 
        "field": "purchased_at", # Field to look into
        "calendar_interval": "month" # Interval (Can be month, day, year)
      }
    }
  }
}
```

---
# Improving the Searching Results

##### match_phrase
It is used to match the text in correct order. But if you want to allow say N number of words in middle, we can insert slob parameter that allow insertion of words in the middle.
This can also be combined in bool to change Relevance Score.

```
GET /proximity/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "phrase_1 phrase_2",
        "slop": 2 # Number of words allowed in the middle (But Relevance score will be decreased when words are far apart.)
      }
    }
  }
}
```

##### Fuzzy Searching
Based on Levenshen Distance Fuzzy Distance is used to find out similar words in dictionary 

```
GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "l0bster love", # Look for Fuzzy in each term
        "operator": "and", 
        "fuzziness": 1,
        "fuzzy_transpositions": false # Transpositions are switching 2 adjacent alphabet in word. This help 
      }
    }
  }
}

## Other Way
GET /products/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "lobster", # Note - This form of fuzzy don't get analysed before searching. So they can get affected by Lowercasing and symbols.
        "fuzziness": "auto"
      }
    }
  }
}
```
##### Elastic Search also allow Synonyms.
Synonyms are imp since they allow multiple searching without  symmetric meaning. They can be also added from TXT file in index which make searching easier.

[Check this] (https://github.com/codingexplained/complete-guide-to-elasticsearch/blob/master/Improving%20Search%20Results/adding-synonyms.md)

##### Highlighting The Search
The Searched term can be highlighted with default tag or Defined custom Tag. This is important when words are searched based on Synonyms and Stemming. 

```
GET /highlighting/_search
{
  "_source": false,
  "query": {
    "match": { "description": "Elasticsearch story" }
  },
  "highlight": {
    "pre_tags": [ "<strong>" ],
    "post_tags": [ "</strong>" ],
    "fields": {
      "description" : {}
    }
  }
}
```