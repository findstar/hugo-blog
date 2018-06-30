---
date: '2018-06-29 20:53:20'
group: til
tags:
- ElasticSearch
- template
- mapping
title: ElasticSearch 에서 template 을 활용하는 방법
url: /2018/06/29/elasticsearch-template
type: post
---

ElasticSearch를 로그 분석용으로 사용할 때 인덱스의 `mapping`은 `template`을 사용해서 생성되도록 설정하면 편리하다.

<!--more-->

로그 분석용으로 ElasticSearch 를 사용하면 주로 인덱스는 daily/weekly/monthly 중에 하나로 구성하는게 일반적이다.
동일한 유형의 인덱스를 계속해서 생성하게 되는데, 예를 들자면 nginx log의 인덱스를 다음과 같이 생성한다고 해보자.

- nginx-access-log-2018.06.20
- nginx-access-log-2018.06.21
- nginx-access-log-2018.06.22
- nginx-access-log-2018.06.23
- ...

위와 같은 인덱스를 자동으로 생성할 때 사용 할 `mapping`을 미리 지정할 수 있는데 이 기능이 바로 `template` 이다.


### 템플릿 생성하기

다음은 `nginx-access-log` 라는 템플릿을 생성하는 것으로, 인덱스가 `nginx-access-*` 에 해당하는 패턴이라면, 지정된 `mapping` 으로 인덱스를 생성한다.

```
## nginx-access-log 템플릿 생성
curl -X "PUT" "http://my-elasticsearch-server-host:9200/_template/nginx-access-log" \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "index_patterns": [
     "nginx-access-*"
  ],
  "mappings": {
    "log": { // type name
      "properties": {
        "ip": {
          "type": "text"
        },
        "host": {
          "type": "keyword"
        },
        "uri": {
          "type": "text"
        },
        "datetime": {
          "type": "date"
        },
        "@timestamp": {
          "type": "date"
        }
      }
    }
    ........
  }
}'

```

### `setting` 설정하기

`mapping` 뿐만 아니라 `setting` 도 지정할 수 있다.

```
curl -X "PUT" "http://my-elasticsearch-server-host:9200/_template/new-template-name" \
     -d $'{
  "index_patterns": [
       "nginx-access-*"
  ],
  "settings": { // setting
    "number_of_shards": 1
  },
  "mappings": {
    "type1": {
      "properties": {
        "host_name": {
          "type": "keyword"
        },
        "created_at": {
          "type": "date",
          "format": "EEE MMM dd HH:mm:ss Z YYYY"
        }
      }
    }
  }
}'

```

### 여러개의 패턴 지정하기

인덱스 패턴을 여러개를 지정하고자 할 때는 배열 안에 계속 추가하면 된다.

```
curl -X "PUT" "http://my-elasticsearch-server-host:9200/_template/new-template-name" \
     -d $'{
  "index_patterns": [
       "pattern1*",
       "pattern2*",
       "pattern3*",
       "pattern4*"
  ],
  "settings": { // setting
    "number_of_shards": 1
  },
  "mappings": {
    "type1": {
      "properties": {
        "host_name": {
          "type": "keyword"
        },
        "created_at": {
          "type": "date",
          "format": "EEE MMM dd HH:mm:ss Z YYYY"
        }
      }
    }
  }
}'

```

### 참고
 - https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-templates.html