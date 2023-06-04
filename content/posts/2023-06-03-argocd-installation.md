---
date: '2023-06-04 19:24:13 +09:00'
group: blog
image: /images/posts/argocd/argocd-logo.png
tags: ["argocd" ]
title: "ArgoCD 설치"
url: /2023/06/04/argocd-installation
type: post
summary: "k8s 클러스터를 운영하면서 애플리케이션을 배포하는 방법은 각각의 상황에 따라 다양하게 존재한다. 많은 배포 방식중에서 ArgoCD에 대해서 알아보고 설치 방법을 정리해보았다."
목적 : "ArgoCD의 설치 방법및 Rollout 방식의 셋팅 방법을 설명한다."
대상독자 : "k8s 에서 배포를 위해서 argocd 설치 방법을 알고자 하는 사람"
---

# ArgoCD 

최근 K8S 클러스터를 정비할 일이 있어서 새롭게 구성을 마치고 배포를 위해서 ArgoCD를 재설치 해보았다.

ArgoCD는 **GitOps**를 기반으로 하는 CD 도구로, k8s 애플리케이션의 CD(지속적인 배포)를 지원한다. 
> GitOps라는 건 ArgoCD가 Git Repository와 연동하여 애플리케이션 배포를 자동화하고, 롤백과 같은 작업을 수행한다는 걸 의미한다.
> 따라서 애플리케이션이 빌드되고 컨테이너 이미지가 생성되는 과정에서 Git Manifest의 내용을 수정하면 K8S 클러스터에 자동으로 애플리케이션이 배포된다.

도식화 하면 다음과 같은 순서가 된다.

{{< imageFull src="/images/posts/argocd/argocd-diagram.png" title="ArgoCD 도식화" border="false" >}}

* 쉽게 말해서 ArgoCD 는 K8S 클러스터를 위한 **애플리케이션 배포 도구**이고 **Gitops** 방식으로 배포한다는 특징을 가지고 있다.

K8S를 운영하면 애플리케이션을 어떻게 배포할지는 사실 케이스별로 각각 다른 도구를 사용할 수도 있다. 각각의 상황에 따라 사용 가능한 도구가 제한될 수도 있다. (사내 도구를 사용해야한다던가, 이미 운영중인 도구가 있을 수도 있다.)
그만큼 다양한 배포 방식이 존재하고, 다양한 도구들이 있기 때문에 어느 것이 더 좋다고 말하기는 어렵다. 나의 경우에는 **ArgoCD**를 가장 익숙하게 사용하고 있다.


## 설치 방법

### 준비사항

* 먼저 ArgoCD가 설치될 K8S 클러스터를 준비한다. 
  - 나의 경우에는 ArgoCD가 설치되어 동작할 별도의 K8S 클러스터를 준비했다. 
  - 그 이유는 ArgoCD의 동작이 실제 Production 애플리케이션이 구동되는 클러스터에 영향을 주지 않도록 하고 싶었기 때문이다.
  - 참고로 애플리케이션은 dev, cbt, prod 를 위한 클러스터가 독립적으로 존재했다.

* ArgoCD 설치 가이드 확인 
  - 설치 시점에 2.6을 사용했다. (그 사이 2.7이 릴리즈 되었다.) 
  - https://argo-cd.readthedocs.io/en/release-2.6/getting_started/

* ArgoCD CLI 설치
  - 설치도중 사용할 CLI 명령어를 준비한다. 
  ```bash
  brew install argocd
  ```

### 설치 순서

1. 공식 가이드 문서를 참고하여 클러스터에 ArgoCD를 설치한다.
    ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```

2. 도메인부여
    - 각각 환경이 달라서 세부 설정은 생략한다.
    - 다만 Ingress 를 설정하여 http 접근으로 ArgoCD에 접근가능하게 하였다.
    ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: argocd-server-http-ingress
     namespace: argocd
     annotations:
       kubernetes.io/ingress.class: "nginx"
       nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
       nginx.ingress.kubernetes.io/backend-protocol: "https"
       nginx.ingress.kubernetes.io/ssl-passthrough: "true"
   spec:
     rules:
     - http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: argocd-server
               port:
                 name: http
       host: my-argocd.my-domain.com
     tls:
     - hosts:
       - 'my-argocd.my-domain.com'
       secretName: my-domain-secret
   ```

3. 접속확인
    - Ingress 에서 설정한 "my-argocd.my-domain.com" 으로 접속해본다.
    - 로그인 창이 표시되면 정상적으로 설치된 것이다. 
    - 기본 계정은 `admin` 이고 패스워드가 필요한데, 패스워드는 ArgoCD가 설치될 때 임의의 값으로 생성된다. 
    - ArgoCD가 생성한 임의의 패스워드를 확인하기 위해서 다음과 같이 명령어를 입력해서 확인한다.
    ```bash
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath=``"{.data.password}" | base64 -d; echo
    ```
   {{< imageFull src="/images/posts/argocd/argocd-login.png" title="ArgoCD 로그인" border="false" >}}
   
4. Github 연결 (optional)
    - admin 사용자로 ArgoCD를 사용할 수 있지만, 팀에서 운영한다면 개별 계정을 사용해서 접근하는 것이 권한관리도 쉽고 관리 이력상 좋다. 
    - 계정연동 방법은 여러방법이 있지만, 개발자라면 모두 Github 계정은 있을테니 Github 과 연동해서 로그인해보도록 설정해보았다.
    - 신규 OAuth App 등록
        - "https://github.com/organizations/{my-org}/settings/applications" 에 접속하여 새로운 OAuth 앱을 생성하자 
        - 생성한 OAuth App 의 ClientId, ClientSecret 를 복사해놓는다.
    - ArgoCD ConfigMap 등록
        - 다음과 같이 ArgoCD가 설치된 클러스터에 ConfigMap 을 등록한다.
        ```yaml
        apiVersion: v1
        kind: ConfigMap
        metadata:
          annotations:
          labels:
            app.kubernetes.io/name: argocd-cm
            app.kubernetes.io/part-of: argocd
          name: argocd-cm
          namespace: argocd
        data:
          url: https://my-argocd.my-domain.com
          dex.config: |
            connectors:
              - type: github
                id: github
                name: github
                config:
                  hostName: github.com
                  clientId: "깃헙에서 복사한 client Id"
                  clientSecret: "깃헙에서 복사한 client secret"
                  orgs:
                    - name: "github organization"
       ```
   - Rbac ConfigMap 등록
       - rbac 를 위한 ConfigMap 도 등록한다.
       ```yaml
       apiVersion: v1
       kind: ConfigMap
       metadata:
         name: argocd-rbac-cm
         namespace: argocd
       data:
       policy.default: role:admin
       ```
   - 이제 다시 ArgoCD 웹 UI에 접속하면 Github 계정을 사용해서 로그인 할 수 있다. 
   - {{< imageFull src="/images/posts/argocd/argocd-login-with-github.png" title="ArgoCD Github 로그인" border="false" >}}

5. Gitops Manifest Repo 연결
    - ArgoCD에서 변경을 감지하고 K8S 클러스터의 형상을 동기화할 Gitops Manifest Repo를 연결한다. 
    - {{< imageFull src="/images/posts/argocd/argocd-repo-setting.png" title="ArgoCD Repo Setting" border="false" >}}

6. 타겟 K8S 클러스터 등록
    - ArgoCD 가 애플리케이션을 배포할 대상이 되는 Target K8S Cluster 를 등록해야한다. 
    - 이 과정은 Web UI에서 불가능하기 때문에 앞서 준비과정에서 설치한 ArgoCD CLI를 사용한다.
    ```bash
    argocd login my-argocd.my-domain.com --username admin (패스워드입력필요)
    argocd cluster add my-prod-target-cluster-context
    ```
    - context 를 추가하는 부분에서 알 수 있듯이 cli 를 실행하는 시점에 context 정보가 로컬에 있어야 한다 

7. 프로젝트 등록
    - 이제 프로젝트를 등록해야한다. 
    - 프로젝트는 일종의 애플리케이션의 그룹을 지정한다고 이해하면 된다. 

8. 애플리케이션 등록
    - 마지막으로 애플리케이션을 등록해보자. 입력할 항목이 많은데 다음의 부분만 입력하면 된다.
        - GENERAL - Application Name (애플리케이션 이름)
        - GENERAL - Project Name (소속될 프로젝트 이름)
        - GENERAL - SYNC POLICY (Gitops Repo 에 변경이 일어 났을 시 자동으로 클러스터에 동기화 할것인지, 수동으로 할것인지 결정)
        - SOURCE - Repository URL (Gitops Repo 선택)
        - SOURCE - Path (애플리케이션이 참조할 YAML Path 선택)
        - DESTINATION - Cluster URL (클러스터 등록이 잘되었다면 대상 K8S 클러스터의 Master Node 주소가 나온다.)
        - DESTINATION - Namespace (애플리케이션을 배포할 네임스페이스 지정)
    - 나머지 부분은 그대로 두고 등록하면 된다. 

### 동기화가 진행된 애플리케이션 샘플

동기화가 진행되면 다음과 같이 애플리케이션의 상태를 확인할 수 있다.
{{< imageFull src="/images/posts/argocd/argocd-app-detail.png" title="ArgoCD Application Sample" border="false" >}}

### 기타 

- 필요시 [ArgoCD Notification](https://argocd-notifications.readthedocs.io/en/stable/) 을 설치하여 알림을 받을 수 있도록 한다.

## 정리

1. K8S 클러스터를 운영할 때 애플리케이션 배포를 위해서 ArgoCD를 운영할 수 있다.
2. ArgoCD는 Prod 클러스터에 영향을 주지 않기 위해서 별도의 클러스터에서 운영하기로 하였다.
3. 로그인을 쉽게 하기 위해서 Github OAuth 연동을 하였다
4. 클러스터 추가는 CLI에서만 가능하다.
5. 애플리케이션을 추가하여 Sync 를 확인한다.
6. 필요시 Notification 을 추가로 설치한다.

## 참고자료 

- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/)
- [ArgoCD Notification](https://argocd-notifications.readthedocs.io/en/stable/) 