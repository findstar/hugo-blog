---
aliases:
    - /til/2018-01-24-TIL
categories:
- TIL
date: '2018-01-24'
group: til
permalink: /til/:year-:month-:day-TIL
tags:
- ElasticSearch
- "null"
- empty string
- difference
title: ElasticSearch 에서 null 과 empty string 의 차이
url: /2018/01/24/difference-between-null-and-empty-string-elasticsearch
type: post
---


# 1월 24일 (수) TIL

ElasticSearch 에서 indexing 을 처리할 때, json 객체를 생성하면서 속성값이 때로는 `null` 또 어떨때는 빈 문자열 `""` 로 채워넣을 때가 있었는데
보다 명확하게 이해하고 처리하기 위해서 `null` 값과 `empty string("")` 의 차이를 알아보았다.

<!--more-->

> <span class="evidence">결론부터 말하자면 둘의 차이는 없다라고 이야기 할 수 있겠다.</span>

그 이유는 바로 inverted index 를 생각하면 알 수 있는 건데, `null` 과 `empty string` 은 둘다 analyzer 에 의해서 분석된 이후
inverted index를 하나이상 생성하지 못하기 떄문에 ES 입장에서는 둘의 차이가 없는 것이다.

그렇다면

* `null` 또는 `empty string` 과 다르게 아예 mapping field 가 없는 경우는 어떠한가? 이 또한 inverted index 가 생성되지 않기 때문에 동일하게 취급되는가?

> 이 경우에도 마찬가지다. inverted index 가 생성되지 않기 때문에, 두개는 동일하게 취급된다.

* `exist` 쿼리를 사용해도 마찬가지다. 다음의 5개 데이터가 있다고 가정해보자.

```
{ "tags" : ["search"]                } // id 1
{ "tags" : ["search", "open_source"] } // id 2
{ "other_field" : "some data"        } // id 3
{ "tags" : null                      } // id 4
{ "tags" : ["search", null]          } // id 5
```

아래와 같이 질의를 하면,
```
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "exists" : { "field" : "tags" }
            }
        }
    }
}
```

다음의 결과를 확인가능하다.
```
"hits" : [
    {
      "_id" :     "1",
      "_score" :  1.0,
      "_source" : { "tags" : ["search"] }
    },
    {
      "_id" :     "5",
      "_score" :  1.0,
      "_source" : { "tags" : ["search", null] }
    },
    {
      "_id" :     "2",
      "_score" :  1.0,
      "_source" : { "tags" : ["search", "open source"] }
    }
]
```

> 결과는 exist 쿼리를 질의해도 3, 4번 데이터가 hit 되지 않는다. mapping field 가 아예 존재하지 않는 경우와 null, empty string 인 경우 모두 동일하다.

* 그렇다면, 속성값이 `null` 인 document를 DSL 을 통해서 찾을 수 있을까?

아래와 같이 질의하면,
```
{
    "query" : {
        "constant_score" : {
            "filter": {
                "missing" : { "field" : "tags" }
            }
        }
    }
}
```

다음의 결과를 확인가능하다
```
"hits" : [
    {
      "_id" :     "3",
      "_score" :  1.0,
      "_source" : { "other_field" : "some data" }
    },
    {
      "_id" :     "4",
      "_score" :  1.0,
      "_source" : { "tags" : null }
    }
]
```

mapping field 상에서 존재하지 않는 경우와, value 가 null 인 경우 모두를 찾는다. 따라서 아예 field가 없는 경우만 딱 지정해서 찾을 수는 없다.


> 요약하자면, inverted index를 기준으로 모든 ES query 와 data를 생각하면 된다는 것이다. `null`, empty string(""), missing field 모두 동일하다.


참고 : https://www.elastic.co/guide/en/elasticsearch/guide/current/_dealing_with_null_values.html