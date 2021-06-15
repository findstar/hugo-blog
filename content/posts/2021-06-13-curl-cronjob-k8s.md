---
date: '2021-06-13 22:21:09 +09:00'
group: blog
image: /images/posts/curl/curl_cronjob_k8s.png
tags:
- "curl"
- "k8s"
- "cronjob"
title: "curl 을 사용하여 K8S(Kubernetes) 에서 cronjob 등록하기"
url: /2021/06/13/curl-cronjob-k8s
type: post
summary: "종종 쿠버네티스 클러스터에서 cronjob 을 활용할 일이 있는데 간단하게 curl 명령어 한번만 실행하면 되는 경우가 있다.  
  이럴 때 사용하기 위한 alpine 기반의 curl 이미지를 사용하는 방법에 대해서 알아보았다. "
---
# k8s 에서 curl 을 사용하여 cronjob 등록하기

## 배경

종종 쿠버네티스 클러스터에서 cronjob 을 활용할 일이 있는데 간단하게 curl 명령어 한번만 실행하면 되는 경우가 있다.  
이럴 때 사용하기 위한 alpine 기반의 curl 이미지를 사용하는 방법에 대해서 알아보았다. 

### k8s cronjob

쿠버네티스의 오브젝트 중에서 `cronjob` 오브젝트는 yml 에 정의된 스케줄에 맞게 지정한 이미지를 기반으로 명령어를 실행하는 오브젝트다. 
나는 다음과 같은 형식으로 사용하고 있다. 

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: my-cronjob-name
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: my-cronjob-name-pod-name
              image: curlimages/curl:7.77.0
              imagePullPolicy: IfNotPresent
              command:
                - "/bin/sh"
                - "-c"
                - |
                  curl -X "POST" "somehost" \
                       -H 'header-key: value' \
                       -H 'Content-Type: application/json; charset=utf-8'
          restartPolicy: Never
```

## 설명

1. 먼저 apiVersion 에 맞게 `CronJob` 을 등록한다. 

2. schedule 는 리눅스의 cronjob 을 등록하는 포맷과 동일한데 위의 예제에서는 5분 마다 실행하라고 정의하였다. 

3. image 는 `curlimages/curl:7.77.0` 을 사용하였는데 필요하다면 alpine 기반으로 `apk --no-cache add curl` 을 추가하여 직접 이미지를 생성하여 사용해도 된다. 

4. command 부분에서 `|` 로 이어지는 부분으로 연결하면 curl 명령어를 raw 하게 정의할 수 있다. 콘솔에서 입력하는 형태 그대로 정의하자.

## 실행 기록

`kubectl get pods` 로 확인해보면 다음과 같이 언제 완료되었는지 확인이 가능하고 필요하다면 실행 결과를 로그로 확인할수도 있다. 
```shell
$ kubectl get pods
my-cronjob-name-1623488400-tb4zs                   0/1     Completed           0          16s

$ kubectl logs my-cronjob-name-1623488400-tb4zs
Response OK 
```
## 참고

https://nieldw.medium.com/using-a-kubernetes-cronjob-the-team-city-api-to-trigger-a-regular-backup-17dfb076d83a
