---
date: '2018-07-14 10:02:32 +09:00'
group: blog
tags:
- ElasticSearch
- wildcard
- ngram
- "Partial Matching"
- match_phrase
title: ElasticSearch 에서 wildcard 쿼리 대신 ngram을 활용하는 방법
url: /2018/07/14/elasticsearch-wildcard-to-ngram
type: post
---

ElasticSearch를 사용하면서 DSL 을 구성할 때, RDBMS 의 `like "%keyword%"` 와 같은 쿼리를 대체하기 위해서 `wildcard` 를 사용하는 경우를 몇번 목격하였다.
이 경우 원하는 결과를 제대로 얻을 수도 없을 뿐더러, 성능의 문제가 발생하기 쉬운데, 이를 `ngram` 으로 대체하여 원하는 결과를 얻는 방법을 확인해 보았다.

<!--more-->

## 콘텐츠 검색의 경우

시의 내용을 DB 와 ElasticSearch 에 저장하고 쿼리를 통해서 원하는 시의 본문 전체 내용을 찾는 방법을 가정해 보자. 저장된 내용은 **김소월의 진달래꽃** 을 예시로 가정해봤다.

```
진달래꽃

나 보기가 역겨워
가실 때에는
말없이 고이 보내 드리우리다

영변에 약산
진달래꽃
아름 따다 가실 길에 뿌리우리다

가시는 걸음 걸음
놓인 그 꽃을
사뿐히 즈려밟고 가시옵소서

나 보기가 역겨워
가실 때에는
죽어도 아니 눈물 흘리우리다
```


### RDBMS 의 경우

먼저 RDBMS 의 `like "%keyword%"` 쿼리를 생각해 보자, `걸음` 이라는 텍스트가 포함되어 있는 row 를 찾으려면 아래처럼 쿼리를 하게 되겠다.
이런 쿼리를 사용하는 순간 full text searching 을 하기 때문에 가급적이면 사용을 지양해야할 쿼리이긴 하지만, 어쩔 수 없이 사용한다고 가정하자.

```
SELECT * FROM poems WHERE contents LIKE '%걸음%'
```

```
id | contents
1 "진달래꽃 나 보기가 역겨워 가실 때에는 말없이 고이 보내 드리우리다 영변에 약산 진달래꽃 아름 따다 가실 길에 뿌리우리다 가시는 걸음 걸음 놓인 그 꽃을 사뿐히 즈려밟고 가시옵소서 나 보기가 역겨워 가실 때에는 죽어도 아니 눈물 흘리우리다"
```

### ElasticSearch 의 경우

RDBMS 에서 처럼 full text searching 이 가능할것으로 기대하고, `wildcard` 쿼리를 사용해보자.

```
curl -s http://my-elastic-cluster-host:9200/index-name/_search?pretty -XPOST -d '
{
  "query" : {
      "bool": {
        "must": {
          "wildcard": {
            "contents": "*걸음*"
          }
        }
      }
  }
}'
```

결과는 아래와 같은 형태가 된다.

```
{
  "took": 230,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": null,
    "hits": [
      {
        "_index": "index-name",
        "_type": "poems",
        "_id": "1-1",
        "_score": null,
        "_source": {
          "contents": "진달래꽃 나 보기가 역겨워 가실 때에는 말없이 고이 보내 드리우리다 영변에 약산 진달래꽃 아름 따다 가실 길에 뿌리우리다 가시는 걸음 걸음 놓인 그 꽃을 사뿐히 즈려밟고 가시옵소서 나 보기가 역겨워 가실 때에는 죽어도 아니 눈물 흘리우리다"
        }
      }
    ]
  }
}
```

원하는대로 **진달래꽃**을 잘 찾았다. 그럼 이번에는 키워드를 바꿔서 `나 보기가 역겨워`라는 문장에서 `기가` 라는 단어를 기준으로 검색해 보자.

* RDBMS

```
SELECT * FROM poems WHERE contents LIKE '%기가%'
```

여전히 아래와 같이 잘 찾는다.

```
id | contents
1 "진달래꽃 나 보기가 역겨워 가실 때에는 말없이 고이 보내 드리우리다 영변에 약산 진달래꽃 아름 따다 가실 길에 뿌리우리다 가시는 걸음 걸음 놓인 그 꽃을 사뿐히 즈려밟고 가시옵소서 나 보기가 역겨워 가실 때에는 죽어도 아니 눈물 흘리우리다"
```


* ElasticSearch

```
curl -s http://my-elastic-cluster-host:9200/index-name/_search?pretty -XPOST -d '
{
  "query" : {
      "bool": {
        "must": {
          "wildcard": {
            "contents": "*기가*"
          }
        }
      }
  }
}'
```

그런데 이번에는 원하는 결과를 찾을 수가 없다.

```
{
  "took": 230,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "hits": {
    "total": 0,
    "max_score": null,
    "hits": []
  }
}
```

### 왜 그럴까?

이 차이는 바로 `wildcard` 쿼리가 `term level query` 이기 때문이다. ElasticSearch는 실제로 저장된 `document` 의 원문을 검사하는 것이 아니라, `inverted index` 의 목록 중에서
`term(token)` 중에 쿼리에서 질의한 `keyword`를 찾기 때문이다. (이건 아주 기본적인 사항이지만, 우리는 이를 금방 까먹는다...)

그럼 **진달래꽃**은 어떤 token 으로 구성되어 있는지 확인해 보자.

```
curl -s http://my-elastic-cluster-host:9200/index-name/_analyze?pretty -XPOST -d '
  "text": "진달래꽃 나 보기가 역겨워 가실 때에는 말없이 고이 보내 드리우리다 영변에 약산 진달래꽃 아름 따다 가실 길에 뿌리우리다 가시는 걸음 걸음 놓인 그 꽃을 사뿐히 즈려밟고 가시옵소서 나 보기가 역겨워 가실 때에는 죽어도 아니 눈물 흘리우리다",
  "analyzer": "my_custom_analyzer"
}'

```

token 의 결과를 보면 "\*기가\*" 와 매칭되는 token 이 존재하지 않는다. **보기가** 라는 단어는 다음과 같이 **보기** 의 단어(word)로 tokenize 되기 때문이다. (어떤 analyzer 를 사용하느냐는 여기서 언급하지 않겠다.)

```
{
  "tokens": [
    .......
    {
      "token": "보기",
      "start_offset": 0,
      "end_offset": 2,
      "type": "VX",
      "position": 7
    },
    .......
  ]
}
```

따라서 처음에 생각한 `걸음` 이라는 단어는 검색이 되지만, `기가` 라는 단어는 검색이 되지 않는다.

### 그렇다면 이를 어떻게 해결할 수 있을까?

> 이런 경우 `ngram`을 사용 하면 된다.


mapping 에 nested 구조로 ngram 이라는 필드를 추가해보자

```
mappings
......
  "mappings": {
    "poems": { // type name
      "properties": {
        "contents": {
          "type": "text",
          "fields": {
            "ngram": {
              "type": "text",
              "analyzer": "my_customer_ngram_analyzer"
            }
          }
        }
        ....
      }
    }
    ....
```

```
settings
......
  "analyzer": {
    "my_customer_ngram_analyzer": {
      "tokenizer": "my_customer_ngram_tokenizer"
    }
  },
  "tokenizer": {
    "my_customer_ngram_tokenizer": {
      "type": "ngram",
      "min_gram": "2",
      "max_gram": "5"
    }
  }
......
```

이제 새롭게 document를 추가하면 contents.ngram 이라는 필드에 token 이 다르게 나온다. 다시 analyzer 결과를 확인해 보자.


```
curl -s http://my-elastic-cluster-host:9200/index-name/_analyze?pretty -XPOST -d '
  "text": "진달래꽃 나 보기가 역겨워 가실 때에는 말없이 고이 보내 드리우리다 영변에 약산 진달래꽃 아름 따다 가실 길에 뿌리우리다 가시는 걸음 걸음 놓인 그 꽃을 사뿐히 즈려밟고 가시옵소서 나 보기가 역겨워 가실 때에는 죽어도 아니 눈물 흘리우리다",
  "analyzer": "my_customer_ngram_analyzer"
}'

```

이번에는 단어를 하나하나 쪼개서 길이 2 ~ 길이 5사이의 token 으로 나뉘어진다. `보기가` 라는 단어는 `보기`, `보기가`, `기가` 라는 세개의 토큰으로 나뉘어 졌다.

```
{
  "tokens": [
    ......
    {
      "token": "보기",
      "start_offset": 0,
      "end_offset": 2,
      "type": "word",
      "position": 7
    },
    {
      "token": "보기가",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 7
    },
    {
      "token": "기가",
      "start_offset": 1,
      "end_offset": 3,
      "type": "word",
      "position": 8
    },
    ......
}
```

이렇게 `ngram` 을 설정하면 **min**, **max** 의 설정에 따라서 나뉘는 기준이 달라진다. **사랑합니다** 라는 단어를 ngram 의 length 에 따라서 나뉘어 보면 다음과 같다.

- Length 1 (unigram): [ 사, 랑, 합, 니, 다 ]
- Length 2 (bigram): [ 사랑, 랑합, 합니, 니다 ]
- Length 3 (trigram): [ 사랑합, 랑합니, 합니다 ]
- Length 4 (four-gram): [ 사랑합니, 랑합니 ]
- Length 5 (five-gram): [ 사랑합니다 ]

`보기가` 라는 단어는 단어의 길이가 3 이기 때문에 총 6개의 token 으로 나뉘어 졌다. 이렇게 단어의 일부분을 `ngram` 으로 잘라내서 tokenize 해서 매칭 시키는 방식을 `partial matching` 이라고 한다.

이렇게 `ngram` 을 사용하도록 변경하였으니, 쿼리를 수정하자. `wildcard` 쿼리는 빼고, `term` 쿼리를 사용하자.

```
curl -s http://my-elastic-cluster-host:9200/index-name/_search?pretty -XPOST -d '
{
  "query" : {
      "bool": {
        "should" : [
            {"term" : { "contents" : "기가" } },
            {"match_phrase" : { "contents.ngram" : "기가" } }
        ],
        "minimum_should_match" : 1
      }
  }
}'
```

위의 쿼리는 `contents` 의 token 과 `contents.ngram` 의 token을 찾게 된다. 쿼리 결과는 아래와 같이 정상적으로 원하는 문서를 찾을 수 있다.

```
{
  "took": 8,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": null,
    "hits": [
      {
        "_index": "index-name",
        "_type": "poems",
        "_id": "1-1",
        "_score": null,
        "_source": {
          "contents": "진달래꽃 나 보기가 역겨워 가실 때에는 말없이 고이 보내 드리우리다 영변에 약산 진달래꽃 아름 따다 가실 길에 뿌리우리다 가시는 걸음 걸음 놓인 그 꽃을 사뿐히 즈려밟고 가시옵소서 나 보기가 역겨워 가실 때에는 죽어도 아니 눈물 흘리우리다"
        }
      }
    ]
  }
}
```


**와일드카드 쿼리를 사용했을 때는 결과가 230 ms 였다면, ngram 을 사용하여 조정한 뒤에는 8 ms 가 걸렸다.**


## 로그 분석의 경우

nginx access log 를 ElasticSearch 를 저장하고 조회 & 분석하는 경우라면 URL 에 숫자와 알파벳이 조합된, 특수한 ID 값 같은 식별자가 붙는 경우가 많다.
예를 들면 ex) `?user=3FGAZS1032&key2=ssssss` 라고 가정해보자.. 이 경우 user 가 `3FGAZS1032` 인 경우를 찾는다면 문제 없겠지만, user 값에 `3FGAZS`가 들어가는 경우를 찾는다면,
원하는 결과를 얻기가 어렵다. 마찬가지로 이 경우도 `ngram` 을 통한 tokenize 를 구성했다면 `partial mathching` 이 가능해져 원하는 문서를 찾을 수 있다.

`3FGAZS1032` 라는 단어가 analyze 가 어떻게 되는지 살펴보자

```
curl -s http://my-log-es-cluster-host:9200/log-2018.07.14/_analyze?pretty -XPOST -d '
{
  "text": "3FGAZS1032",
  "analyzer": "log_analyzer"
}
```

token 이 하나만 나온다.

```
{
  "tokens": [
    {
      "token": "3FGAZS1032",
      "start_offset": 0,
      "end_offset": 10,
      "type": "<ALPHANUM>",
      "position": 0
    }
  ]
}
```

이 상황에서 `\*3FGAZS\*` 라는 와일드카드 쿼리를 사용할 수 있지만 응답속도가 느리다. 그렇지만 `ngram` 을 구성하면 훨씬 빠르다.

```
curl -s http://my-log-es-cluster-host:9200/log-2018.07.14/_search?pretty -XPOST -d '
{
  "_source": ["user","path"],
  "query" : {
      "bool": {
        "must": [
          "term": {
            "user.ngram": "3FGAZS"
          },
          "term": {
            "path.keyword": "/user/playground"
          }
        ]
      }
  }
}'
```

결과가 1ms 가 나왔다.

```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 39,
    "max_score": null,
    "hits": [
      {
        "_index": "log-2018.07.14",
        "_type": "log",
        "_id": "3078923",
        "_score": null,
        "_source": {
          "user": "3FGAZS1032",
          "path": "/user/playground"
        }
      }
      ......
    ]
  }
}
```

### wildcard 대신 ngram 을 도입하여 얻은 효과

- 먼저 token 을 기준으로 wildcard 에서 못찾는 document를 찾을 수 있다.
- 반응 속도가 엄청 빨라진다. (1ms 라니..)
- 대신 token 의 갯수가 많아지기 때문에 inverted index 사이즈가 늘어난다.


## 결론

ElasticSearch 쿼리를 작성하면서 wildcard 를 사용할 때 RDBMS 의 `like "%keyword%"` 와 같이 사용하고자 한다면 `ngram` 을 사용한 tokenize 를 고려해보는게 좋다.
wildcard 는 term(token) level query 이고, 이 차이점을 잘 알아 둘 필요가 있다. 더불어 `ngram` 과 `match_phrase` 쿼리를 사용하면 긴 단어나 문장을 검색 키워드로 조회하더라도
원하는 문서를 잘 찾을 수 있다(콘텐츠 검색의 경우).
