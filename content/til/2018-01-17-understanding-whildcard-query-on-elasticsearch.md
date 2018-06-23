---
aliases:
    - /til/2018-01-17-TIL
categories:
- TIL
date: '2018-01-17'
group: til
permalink: /til/:year-:month-:day-TIL
tags:
- ElasticSearch
- wildcard
- query DSL
- inverted index
title: ElasticSearch 에서 wildcard 쿼리에 대한 이해
url: /2018/01/17/understanding-whildcard-query-on-elasticsearch
type: post
---


# 1월 17일 (수) TIL

ElasticSearch 에서 쿼리를 작성하던 중 wildcard 쿼리의 결과가 내가 생각했던 것 과는 달라서 내용을 정리해본다.
[wildcard query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-wildcard-query.html)를 작성할 때 기대한 것은
RDBMS 의 like '%keyword%' 와 같은 형태가 가능할 것으로 기대했는데, 막상 쿼리 결과를 확인해 보니 원하는 형태가 아니었다.

<!--more-->

original text 가 *"여러개의 물건들"*이고, 내가 시도한 쿼리는 다음과 같았다.
```
{
  "query": {
    "bool": {
      "must": [
        {
          "wildcard": {
            "title": "*여러개*"
          }
        }
      ]
    }
  },
  "size": 100
}
```

그런데 결과는 아래와 같이 hits count 가 0 이었다.

```
{
  "took": 243,
  "timed_out": false,
  "hits": {
    "total": 0,
    "max_score": null,
    "hits": []
  }
}
```

원인은 바로 `wildcard query` 가 term level query 이기 때문이다.

[term level query](https://www.elastic.co/guide/en/elasticsearch/reference/current/term-level-queries.html) 문서를 확인해 보자. (진작에 좀 읽었어야 하는데..)

term 즉 inverted index 를 기준으로 결과를 찾는다는 의미다. 다시 말해 analyzed 된 term keyword 가 있어야 하며,
문서를 색인할 때 토크나이징된 단어가 아니라면 inverted index에 들어 있지 않기 때문에, 아예 비교 대상에 포함되지 않는 것이다.

내가 테스트한 custom analyzer의 토크나이징은 아래와 같았으니,

```
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -H 'Content-Type: application/json' -d'
{
  "analyzer": "my_custom_analyzer",
  "text": "여러개의 물건들"
}


{
  "tokens" : [
    {
      "token" : "여러",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "MM",
      "position" : 0
    },
    {
      "token" : "개",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "NNB",
      "position" : 1
    },
    {
      "token" : "물건",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "NNG",
      "position" : 2
    },
    {
      "token" : "물건들",
      "start_offset" : 5,
      "end_offset" : 8,
      "type" : "NNG",
      "position" : 2
    }
  ]
}
```

토크나이징된 결과에 *"여러개"* 라는 단어는 없기 때문에, 처음 질의한 wildcard 결과가 hits가 0 일 수 밖에 없었던 것이다.

아 문서를 좀 더 잘 읽자.. 오늘의 교훈 *"RTFM"*