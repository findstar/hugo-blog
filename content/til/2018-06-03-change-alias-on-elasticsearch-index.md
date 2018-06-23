---
aliases:
    - /til/2018-06-03-TIL
categories:
- TIL
date: '2018-06-03'
group: til
permalink: /til/:year-:month-:day-TIL
tags:
- ElasticSearch
- index alias
title: ElasticSearch 에서 Index Alias 변경하기
url: /2018/06/03/change-alias-on-elasticsearch-index
type: post
---


# 6월 3일 (일) TIL

ElasticSearch 에서 index 의 alias 를 변경하는 방법을 알아보았다.

<!--more-->

#### Alias 확인

먼저 ElasticSearch 클러스터에 `/_alias` 로 접속하면 현재 생성된 인덱스 들과 연결된 alias 를 확인할 수 있다.

{{< highlight JSON >}}
{
  "activity-log-index": {
    "aliases": {

    }
  },
  "my-contents-index-v1": {
    "aliases": {
      "contents": {

      }
    }
  },
  "my-contents-index-v2": {
    "aliases": {

    }
  }
}

{{< /highlight >}}

현재 `my-contents-index-v1` 이라는 인덱스에 `contents` 라는 *alias* 가 부여되어 있다. 이를 `my-contents-index-v2` 으로 교체해보겠다.

```
curl -XPOST 'http://my-elasticsearch-host:9200/_aliases?pretty' -d'
{
    "actions" : [
        { "remove" : { "index" : "my-contents-index-v1", "alias" : "contents" } },
        { "add" : { "index" : "my-contents-index-v2", "alias" : "contents" } }
    ]
}'
```

한번의 쿼리를 통해서 `my-contents-index-v1` 에서는 alias 가 삭제되고 `my-contents-index-v2` 에는 alias 가 추가되었다.

이후 다시 `/_alias` 로 확인해보면 변경된 결과를 확인가능하다


```
{
  "activity-log-index": {
    "aliases": {

    }
  },
  "my-contents-index-v1": {
    "aliases": {

    }
  },
  "my-contents-index-v2": {
    "aliases": {
      "contents": {

      }
    }
  }
}

```