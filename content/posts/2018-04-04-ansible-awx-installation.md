---
date: '2018-04-04'
group: blog
image: /images/posts/ansible-awx/ansible-awx.png
tags:
- ansible
- awx
- installation
- ansible tower
title: Ansible AWX 를 설치해보기
url: /2018/04/04/ansible-awx-installation
type: post
---


ansible을 팀에서 사용하면서, 몇가지 `이런게 있었으면` 하는 점이 있었는데, 그 중하나가 바로 `GUI 환경`이다.
물론 CLI 콘솔 상에서 playbook을 실행하는데는 아무 문제가 없고 잘 사용하고 있지만, GUI에서도 보기 쉽게 playbook을 실행하고 또한 누가 언제 playbook을 실행했는지 기록되면 좋겠다는 바램이 있었다.
`Ansible Tower`는 이러한 바램을 해소시켜 줄 수 있는 대안이라고 생각이 되지만, 라이센스 비용이 비싸기로 유명(?) 해서, 포기하고 있었는데.
Redhat에서 [Ansible Tower 의 오픈소스 버전으로 Ansible AWX를 발표했다!](https://www.ansible.com/products/awx-project) (대인배...Redhat.)

<!--more-->

AWX(Towner의 오픈소스 버전)는 stanalone으로 구동하며, job(playbook) 에 대한 수행 history, 사용자별 권한제어, GUI, 스케줄링을 통한 실행 기능등을 제공한다.

  - 접근 권한 제어
    : 사용자 및 팀을 구성하고 권한을 설정하여 엑세스를 제한할 수 있다.
  - 스케줄링
    : 작업 일정을 예약하고 반복 옵션을 설정할 수 있다 .
  - 가시성
    : 동작중인 job 과 job의 상태, 내역을 확인할 수 있다.
  - 인벤토리
    : 대상 서버들의 구성 및 dynamic inventory 를 사용하여 클라우드에서 동적으로 생성되고 삭제되는 서버들의 list를 관리할 수 있다.

단일 시스템으로 구동되는데 사용된 기술은 다음과 같다
 - Django
 - Angular JS 1.*
 - Pgsql
 - RabbitMQ
 - memcached

초기 발표 때 부터 [Github Repo](https://github.com/ansible/awx)를 보면서 버전업과 이슈들을 살펴보았는데, 초기에는 install guide가 부실해서 삽질이 많았다.
1.0.3 부터 docker를 통한 설치가 좀 더 간편해졌기 때문에, 이 버전부터는 그냥 docker 구성으로 설치를 진행했다.
AWX에서는 완성된 `docker-compose.yml` 파일을 제공해주지 않기 때문에,
각종 설정을 구성하고, 자체적으로 제공되는 playbook을 실행하면 docker-compose 파일이 생성되는 구조이다. 다음은 `Centos 7.4` 에서 AWX의 설치 과정을 기록한 것이다.

* 필요한 package 설치
```
$ sudo yum -y install epel-release
$ sudo yum -y install git gettext ansible docker nodejs npm gcc-c++ bzip2
$ sudo yum -y install python-docker-py
```

* docker 데몬 시작
```
$ sudo systemctl start docker
$ sudo systemctl enable docker
```

* awx clone
```
$ git clone https://github.com/ansible/awx.git
$ cd awx/installer/
```

* inventory 편집 (핵심!!!)
  - 설치하려는 환경이 proxy를 통해서 외부에 엑세스 한다면 : `host_port`, `http_proxy`, `https_proxy`, `no_proxy`  설정을 자체 환경에 맞게 변경한다.
  - `postgres_data_dir` 위치 변경 : 초기에 `/tmp` 로 잡혀 있는데 이를 그대로 두고 설치하면, 처음에는 잘 뜨지만, OS 가 임시디렉토리를 정리해버리면 DB Data가 날아가 작업한 데이터를 잃어버리는 사태가..
  - `use_docker_compose=true` : installer playbook 을 실행하여 나의 환경에 맞춰진 docker-compose.yml 파일을 생성하는것을 enable 하는 옵션. 이후에는 docker-compose를 통해서 컨테이너를 제어하자.
  - `docker_compose_dir=/var/lib/awx` : docker-compose를 사용하기로 했다면, `docker-compose.yml` 파일이 생성될 디렉토리를 지정해야한다.
  - `dockerhub_version` : docker hub 에서 받아올 awx 버전이다. latest 으로 지정되어 있는데. 아직은 안정화가 덜되어 있는것 같아서 가갑적 태그를 지정해서 쓴다. (현재 최신은 [Docker Hub](https://hub.docker.com/r/ansible/awx_web/tags/)에서 확인)

* 다음은 내가 설정한 내역이다.
```
dockerhub_version=1.0.4.83
postgres_data_dir=/var/awx/pgdocker
use_docker_compose=true
docker_compose_dir=/var/lib/awx
```

* 이제 playbook을 실행해서 AWX를 설치한다.
```
ansible-playbook -i inventory install.yml
```

* docker_compose_dir 로 이동해보면 `docker-compose.yml` 파일이 생성되었다.
```
docker-compose up -d
```

* 이제 웹 브라우저로 접근해보자
{% include image_caption.html imageurl="/images/posts/ansible-awx/awx_welcome_screen.png" title="welcome awx" caption="welcome" %}

* 초기 접속 계정은 `admin/password` 이다.
{% include image_caption.html imageurl="/images/posts/ansible-awx/awx-main-dashboard.png" title="awx dashboard" caption="dashboard" %}

여기까지하면, AWX는 설치가 완료된것이다. 이제 project를 설정해보자.

 * credential 추가

| 항목        | 설정값           |
| ------------- |:-------------:|
| name			   | Ansible Control Node Credential|
| description   | playbook 을 실행하기 위해서 control node 에 접속하기 위한 credential|
| organization | Default |
| type | machine |
| username | ssh 계정명 |
| password | ssh 패스워드|
| PRIVILEGE ESCALATION | sudo |



 * project 추가

| 항목        | 설정값           |
| ------------- |:-------------:|
| name			   | Project Name|
| description   | 프로젝트 설명|
| organization | Default |
| SCM type | git |
| SCM Url | ansible playbook github repo 주소 |
| SCM Branch | |
| SCM Update Options | Clean, Update on Launch 체크 |

 * inventory 추가

   ansible 을 CLI 에서 다룰 때는 hosts 파일을 지정하면 되었지만, awx 에서는 target node들을 db로 관리한다. 따라서 연결된 SCM 에서 hosts 파일을 읽어 오거나 (매번 playbook 이 실행되기 전에 hosts 내역을 업데이트 한다)/dynamic inventory(클라우드와 같이 target node 들이 유연하게 생성/삭제되어 변경되는 경우)/ 또는 수동으로 직접 관리할 수 있다..


 * job template 추가

| 항목        | 설정값           |
| ------------- |:-------------:|
| name			   | 실행할 playbook 의 제목|
| description   | 실행할 playbook 의 설명 |
| job type | run  (check 인경우에는 dry run 만 수행) |
| inventory | UI 상에서 추가한 inventory 연결 |
| project | Project Name|
| playbook | 프로젝트에 연결된 SCM(git)에서 playbook 리스트를 자동으로 불러와 그 중 하나를 선택한다. |
| machen Credential | Ansible Control Node Credential : playbook 을 실행하기 위한 control node 에 접속하기 위한 계정정보|
| Options | Enable Privilges Escalation (sudo 필요시 체크)  |


 * run job template


   template 에서 등록한 job을 run(로켓 모양 아이콘 클릭) 하면 된다. 이렇게 되면 jobs 메뉴에 실행 이력이 추가된다.

```
Identity added: /tmp/awx_40_wPmxLi/credential_6 (/tmp/awx_40_wPmxLi/credential_6)
Using /etc/ansible/ansible.cfg as config file


PLAY [all] *********************************************************************

TASK [delete project directory before update] **********************************
skipping: [localhost]

TASK [check repo using git] ****************************************************
skipping: [localhost]

TASK [update project using git] ************************************************
changed: [localhost]

TASK [Set the git repository version] ******************************************
ok: [localhost]

TASK [update project using hg] *************************************************
skipping: [localhost]

TASK [Set the hg repository version] *******************************************
skipping: [localhost]

TASK [parse hg version string properly] ****************************************
skipping: [localhost]

TASK [update project using svn] ************************************************
skipping: [localhost]

TASK [Set the svn repository version] ******************************************
skipping: [localhost]

TASK [parse subversion version string properly] ********************************
skipping: [localhost]

TASK [Ensure the project directory is present] *********************************
skipping: [localhost]

TASK [Fetch Insights Playbook(s)] **********************************************
skipping: [localhost]

TASK [Save Insights Version] ***************************************************
skipping: [localhost]

TASK [Repository Version] ******************************************************
ok: [localhost] => {
    "msg": "Repository Version fafb252e642065bff7dfa4349adeade7e085825d"
}

TASK [Write Repository Version] ************************************************
changed: [localhost]

PLAY [all] *********************************************************************

TASK [detect requirements.yml] *************************************************
skipping: [localhost]

TASK [fetch galaxy roles from requirements.yml] ********************************
skipping: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=4    changed=2    unreachable=0    failed=0

```

이제 필요한 만큼 job template(playbook)을 연결하고 실행하면 된다.
요약하자면, awx clone => inventory 파일 설정 => docker-compose up 이다.
보다 자세한 설치는 [install 가이드](https://github.com/ansible/awx/blob/devel/INSTALL.md)를 확인하자.
다음에는 Kubernetes 에서 설치해봐야 겠다.


#### 추가 사항

 * dynamic inventory 설정
  : AWS or GCP or OpenStack 과 같이 인스턴스가 dynamic 하게 생성/삭제되는 경우에 target node를 특정할 수 없다. 이런경우 Cloud insfra 에서 지원되는 API를 통해서 inventory target node(hosts)를 질의해오는 방법이 있는데 이를 dynamic inventory라고 한다.

 * AWX 는 standalone 으로 API 서버로 동작이 가능하다. 따라서 jenkins 나 다른 툴/시스템에서 API를 호출하여 job template을 실행할 수 있다. https://github.com/ansible/tower-cli 를 참고.


### 참고
 * 예제 - https://github.com/ansible/ansible-examples
 * Best priatices - http://docs.ansible.com/ansible/latest/playbooks_best_practices.html
 * Ansible Essential - https://www.ansible.com/blog/ansible-best-practices-essentials
 * AWX 사용 - https://steemit.com/utopian-io/@adson/ansible-open-sources-ansible-tower-with-awx
