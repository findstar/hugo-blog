---
aliases:
    - /til/2018-05-13-TIL
categories:
- TIL
date: '2018-05-13'
group: til
permalink: /til/:year-:month-:day-TIL
tags:
- curl
- file upload
title: CURL 에서 파일 업로드 하기
url: /2018/05/13/upload-file-on-curl
type: post
---


# 5월 13일 (수) TIL

작성한 API를 테스트 하기 위해서 [Postman](https://www.getpostman.com/) 이나 [Paw](https://paw.cloud/)를 주로 사용하는데,
CURL을 사용해서 CLI에서 테스트해야 되는 경우도 종종 있다. 새롭게 API를 테스트 하던 중 `CURL`을 사용해서 파일 업로드 API를 테스트 하는 방법을 확인해 봤다.

<!--more-->

#### 기본 사용법

먼저 파일을 업로드 하는 `request`를 위해서는 `multipart/form-data` 형식으로 보내야 하는데 이를 위해서 사용할 CURL 옵션은 `-F(--form)` 이다. (대문자!!)
그리고 파일의 path를 지정해서 보내면 된다.

```
$ curl -F ‘file1=@/upload/file/path’ http://file.testApi.com
```

#### 여러개 파일 전송

여러개의 파일을 보내야 하는 경우에는 `-F` 옵션을 연속해서 사용하면 된다.

```
$ curl -F ‘file1=@/upload/file1/path’ -F ‘file2=@/upload/file2/path’ http://file.testApi.com
```

#### 파일변수가 배열인 경우

가끔씩 수신하는 서버의 파일파라미터가 배열연 경우도 있다 이 경우는 아래처럼 하면 된다.

```
$ curl -F ‘file[]=@/upload/file1/path’ -F ‘file[]=@/upload/file2/path’ http://file.testApi.com
```

#### 다른 변수값 함께 전달

파일과 함께 다른 인자를 같이 넘겨줘야 되는 경우도 있는데 이럴 때는 `path` 형태가 아닌(@ 가 없는) 형태로 보내면 된다.

```
$ curl -F ‘file1=@/upload/file/path’ -F 'userId=1' -F 'title=test' http://file.testApi.com
```

#### filename 지정

파일인자말고, 원래의 파일이름을 명시하기를 원할 수도 있다.

```
$ curl -F ‘file1=@/upload/file/path;filename=Profile1.png’ http://file.testApi.com
```


참고 : https://medium.com/@petehouston/upload-files-with-curl-93064dcccc76

