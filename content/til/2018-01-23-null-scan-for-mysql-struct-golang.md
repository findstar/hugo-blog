---
aliases:
    - /til/2018-01-23-TIL
categories:
- TIL
date: '2018-01-23'
group: til
permalink: /til/:year-:month-:day-TIL
tags:
- Golang
- mysql
- struct
- json
- "null"
title: Golang 에서 Mysql 데이터를 Struct 에 Scan 할 때 Null 처리 경험기
url: /2018/01/23/null-scan-for-mysql-struct-golang
type: post
---


# 1월 23일 (화) TIL

최근 ElasticSearch 에 bulk insert 를 하기 위한 golang 기반의 간단한 툴을 만들고 있다.
mysql 에서 데이터를 가져온 다음에 이를 JSON으로 변환해서 ElasticSearch로 bulk insert 하는 구조인데, 이 과정에서 mysql rows 를 golang 에서 scan 하면서 알게된 내용을 정리해보았다.

<!--more-->

먼저 golang 툴을 만들면서 사용한 패키지는 아래와 같다.

```
	"gopkg.in/olivere/elastic.v5"
	"database/sql"
	_ "github.com/go-sql-driver/mysql"
	"encoding/json"

```

처음 생성한 `Post`라는 이름의 struct는 다음과 같다.

```
type Post struct {
	PostId         int64          `json:"postId"`
	AuthorId       int64          `json:"authorId"`
	Category       int64          `json:"categoryId"`
	Title          string         `json:"title"`
	Content        string         `json:"content"`
	Created        string         `json:"created"`
	Ip             string         `json:"-"`
	Ip1            int64          `json:"ip1"`
	Ip2            int64          `json:"ip2"`
	Ip3            int64          `json:"ip3"`
	Ip4            int64          `json:"ip4"`
}
```

처음에는 쿼리를 해온 다음에 `rows.Scan` 와 같이 Post type 의 속성에 값을 넣으면 되겠지라고 생각했었는데, 실제로는 값이 `null` 인경우 에러가 발생한다
(여기서는 category 값이 `null` 인 경우가 존재했다.)

```
sql: Scan error on column index 3: unsupported driver -> Scan pair: <nil> -> *int64
```

따라서 이경우는 sql 패키지에서 제공하는 `sql.NullString`을 사용해야 한다.

NullString 은 다음과 같이 `valid` 를 확인해서 그 값을 이용해야 한다.

```
if post.RawCategoryId.Valid  {
    post.CategoryId = post.RawCategoryId.Int64
}
```

최종적으로 struct 는 다음처럼 구성했다.

```
type Post struct {
	PostId         int64          `json:"postId"`
	AuthorId       int64          `json:"authorId"`
	RawCategory    sql.NullInt64  `json:"-"`
	Category       int64          `json:"categoryId"`
	Title          string         `json:"title"`
	Content        string         `json:"content"`
	Created        string         `json:"created"`
	Ip             string         `json:"-"`
	Ip1            int64          `json:"ip1"`
	Ip2            int64          `json:"ip2"`
	Ip3            int64          `json:"ip3"`
	Ip4            int64          `json:"ip4"`
}
```
sql 패키지에서 제공하는 타입은 당연하겠지만. `NullString`, `NullBool`, `NullInt64`, `NullFloat64` 가 있다.


이후에 이 post struct 를 이용해서 JSON을 만들면 된다.
내 경우에는 추가적으로 IP를 ip1, ip2, ip3, ip4 로 나눴는데 이는 ES 쿼리를 편리하게 하기 위해서다.

참고로 JSON으로 `json.Marshal(post)` 할 때에는 struct 에서 `json:"-"` 으로 선언한 속성은 Marshal 결과에 포함되지 않는다.


참고 : https://medium.com/aubergine-solutions/how-i-handled-null-possible-values-from-database-rows-in-golang-521fb0ee267