---
date: '2018-07-07 22:14:53'
group: blog
tags:
- ElasticSearch
- reindex
title: ElasticSearch 에서 reindex 을 활용하는 방법
url: /2018/07/07/elasticsearch-reindex
type: post
---

그럴 것 같지 않지만, ElasticSearch 에서는 `reindex`를 수행할 일이 많이 발생한다. `reindex`를 실행할때 사용할 수 있는 옵션을 확인해 보았다.

<!--more-->

ElasticSearch 를 사용하면서

- 인덱스의 mapping 을 수정해야 하거나,
- 이전 버전 (2.X, 5.X - Rolling upgrade 를 사용한 버전 업그레이드를 지원하지 않는 ES) 에서 최신 버전으로 마이그레이션 해야 할 때,
- index의 이름 변경이 필요할 때 (rename) - 이경우는 주로 `alias` 를 잘 활용하지 않을 때 발생한다.

`reindex`를 실행할 일은 생각보다 자주 발생한다.

`reindex` 란 말 그대로 인덱싱을 새로 한다는 의미이다. 기존에 존재하는 index 에서 새로운 index로 데이터를 새롭게 색인하는 것을 의미한다.


나의 경우에는 쿼리를 추가하는 과정에서 과정에서 aggregation 을 수행해야 하는데, mapping이 잘못 설정 되어 있다던가,
이전 데이터의 마이그레이션 과정에서 source 필드 일부를 제외해야 하거나 하는 등의 케이스에서 `reindex` 가 필요했다.

### 기본적인 `reindex` 실행하기

다음은 가장 기본적인 `reindex` 실행방법이다. source 에 기존의 index, dest 에 새롭게 인덱싱할 index 이름을 지정한다.

```
curl -H 'Content-Type: application/json' -XPOST http://my-elasticsearch-host:9200/_reindex -d `{
  "source": {
    "index": "old-index-name"
  },
  "dest": {
      "index": "new-index-name"
  }
}`

```

이렇게 하면 데이터의 양에 따라서 시간이 걸리긴 하지만 정상적으로 완료되면 아래와 같은 응답을 확인할 수 있다.

```
{
  "took" : 147,   // 소요된 시간 (ms)
  "timed_out": false, // timeout 이 발생했는지 여부
  "created": 120, // 새롭게 생성된 document 의 갯수
  "updated": 0,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "total": 120, // 전체 처리된 document 갯수
  "failures" : [ ]
}
```


### 다른 ES 클러스터의 데이터를 `reindex` 하기

경우에 따라서는 신규 클러스터를 운영해야 해서, 크러스터 내부에서 새롭게 `reindex` 하는게 아니라, `remote` 의 ES 클러스터의 데이터를 `reindex` 해야 될 수도 있다.
이럴 때에는 "source" 에 "remote" 를 지정하면 된다.


```
curl -H 'Content-Type: application/json' -XPOST http://my-elasticsearch-host:9200/_reindex -d `{
  "source": {
    "index": "old-index-name",
    "remote": {
      "host": "http://my-old-es-cluster-host:9200",
      "username": "user",
      "password": "pass"
    },
  },
  "dest": {
      "index": "new-index-name"
  }
}`

```

* "remote" 의 index 를 `reindex` 하고자 한다면, 기존 클러스터의 host 가 whitelist 에 추가되어 있어야 한다. (elasticsearch.yml)

```
reindex.remote.whitelist: "otherhost:9200, another:9200, 127.0.10.*:9200, localhost:*"
```

### 기존 index 의 일부 필드만 `reindex` 하는 방법

일부 필드만 `reindex` 하거나(includes), 불필요한 필드는 제외시킬 수도 있다. (excludes)

```
curl -H 'Content-Type: application/json' -XPOST http://my-elasticsearch-host:9200/_reindex -d `{
  "source": {
    "index": "old-index-name",
    "_source": {
      "includes": ["field1", "field2"]
    }
  },
  "dest": {
      "index": "new-index-name"
  }
}`

```

### 전체 데이터가 아니라, 일부 데이터만 `reindex` 하는 방법

쿼리를 질의해서 기존 index 에서 일부 데이터만을 `reindex` 할 수도 있다. 일반적인 쿼리와 마찬가지로 size 를 지정할 수도 있다.

```
curl -H 'Content-Type: application/json' -XPOST http://my-elasticsearch-host:9200/_reindex -d `{
  "source": {
    "index": "old-index-name",
    "_source": {
      "includes": ["field1", "field2"]
    },
    "size": 10000,
    "query": {
      "bool": {
        "filter": [
          {
            "match_phrase": {
              "field1": "search_keyword"
            }
          },
          {
            "range": {
              "datetime": {
                "gte": "2018-07-05T05:05:54.000Z"
              }
            }
          }
        ]
      }
    }
  },
  "dest": {
      "index": "new-index-name"
  }
}`
```

`reindex` 를 실행하면 한꺼번에 많은 데이터를 새로 인덱싱 해야 하기 때문에 ES 클러스터의 부하가 높아진다. 이미 부하가 높은 클러스터라면,
성능에 여유가 있는지 잘 살펴서 실행하도록 하자. 그리고 이왕이면 언제 필요할지 모르는 `reindex` 의 부담을 최소화 하기 위해서라도 index 는 잘 쪼개서 `alias` 로 관리하는 습관을 들이도록 하자.


### 참고
 - https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html