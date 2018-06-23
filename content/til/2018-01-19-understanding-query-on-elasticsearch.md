---
aliases:
    - /til/2018-01-19-TIL
categories:
- TIL
date: '2018-01-19'
group: til
permalink: /til/:year-:month-:day-TIL
tags:
- ElasticSearch
- term
- match
- match_phrase
- query DSL
title: ElasticSearch 에서 term, match, match_phrase 쿼리에 대한 이해
url: /2018/01/19/understanding-query-on-elasticsearch
type: post
---


# 1월 19일 (금) TIL

작성한 ElasticSearch 쿼리를 테스트 해보다가 정확하게 이해하지 못한 상태고 작성하고 있는 키워드를 발견했다.
작성한 쿼리에는 `term`, `match`, `match_phrase`가 무분별하게 사용되고 있었는데, 다시금 문서를 확인해보고, 잊어먹기 전에 정리해봤다.

<!--more-->

### `term`

사전적 의미로 보자면 *용어* 쯤 되겠다. 해당 content 의 inverted index 에 저장되는 token 들 중에서 쿼리의 키워드와 일치하는 녀석이 있는지 찾아준다.

original text 가 *"여러개의 물건들"*이고, 다음처럼 tokenize 된다면,

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

`term` 쿼리를 통해서 검색 결과에 포함하려면 다음의 4가지 쿼리가 가능하다.


```
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "content": "여러"
          }
        }
      ]
    }
  }
}
```

```
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "content": "개"
          }
        }
      ]
    }
  }
}
```

```
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "content": "물건"
          }
        }
      ]
    }
  }
}
```

```
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "content": "물건들"
          }
        }
      ]
    }
  }
}
```

### `match`

`term`과 마찬가지로 inverted index 에 저장되는 token 들 중에서 일치하는 녀석이 있는지 찾아주는데, 차이점은 바로 검색하는 키워드를 analyze 한다는 것이다.
이 analyze 한 결과의 token 들 중에서 하나라도 일치하면 결과 document 에 포함된다.

따라서 original text 가 위와 같을 때 가능한 쿼리는 다음과 같다.

```
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "content": "여러개"
          }
        }
      ]
    }
  }
}
```

```
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "content": "여러사"
          }
        }
      ]
    }
  }
}
```

```
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "content": "나의 물건들"
          }
        }
      ]
    }
  }
}
```

그 밖에 더 있을 수 있다...


### `match_phrase`

*phrase* 단어는 사전적 의미는 *구* 인데, 흔히 영어 문법 공부할 때 들어보았던 명사구, 부사, 전치사구... 의 *구* 이다.
`둘 또는 그 이상의 어절로 이루어져 한 덩어리로써 절이나 문장의 성분이 되는 동일한 말의 단위` 라고 한다. 여기서 포인트는 *둘 이상!*, *이루어져!* 이다.
즉 `match` 가 token 들 중에 일치하는 keyword 가 하나라도 존재한다면 결과 document 에 포함된다면,
`match_phrase` 가 검색 `match` 처럼 keyword를 analyze 하는 것은 동일하나 그 결과 token 들이 모두 존재하고,
순서도 순차적으로 동일한 document 만을 검색 결과에 포함한다는 차이가 있다.

따라서 `match` 와 같이 "나의 물건들" 이라는 검색 키워드를 넣는다면, `나` 라는 token 이 들어 있지도 않고 순서도 맞지 않으니. `여러가지 물건들` 이라는 document 는 결과에 포함되지 않는다.

이 때 가능한 쿼리는 다음과 같다.

```
{
  "query": {
    "bool": {
      "must": [
        {
          "match_phrase": {
            "content": "여러개"
          }
        }
      ]
    }
  }
}
```

```
{
  "query": {
    "bool": {
      "must": [
        {
          "match_phrase": {
            "content": "여러개의 물건들"
          }
        }
      ]
    }
  }
}
```

순서만 바꿔도 안된다

```
{
  "query": {
    "bool": {
      "must": [
        {
          "match_phrase": {
            "content": "물건들 여러개"
          }
        }
      ]
    }
  }
}
```

### 사용예

- `term` 의 경우에는 문서의 tag 를 검색할 때 사용할 수 있겠다.
- 본문 검색에서는 `match_phrase` 를 사용하는게 `match` 보다는 적합하다고 생각된다. (물론 nested should 와 함께 더 넣어야한다.)


오늘도 또 한번 느낀다. RTFM. 문서를 잘 읽자.