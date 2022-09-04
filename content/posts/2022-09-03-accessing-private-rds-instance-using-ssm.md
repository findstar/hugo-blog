---
date: '2022-09-03 19:51:32 +09:00'
group: blog
image: /images/posts/aws/rds/aws_rds_with_ssm.png
tags: ["aws", "rds", "system manager"]
title: "gossm 을 사용하여 Private RDS 인스턴스에 접근하기"
url: /2022/09/03/accessing-private-rds-instance-using-ssm
type: post
summary: "Private 서브넷에 있는 RDS 인스턴스는 외부에서 직접적으로 접근이 불가능하다. 이럴 때 데이터베이스에 접근하기 위한 SSM (gossm)사용법을 알아보았다."
---

# SSM을 사용하여 Private RDS 인스턴스에 접근하기

## Private RDS

AWS에서 RDS(데이터베이스) 인스턴스를 생성하면 외부에서 접근이 되지 않는 VPC 내부의 Private subnet 에 위치시키는 것이 일반적이다. RDS 인스턴스가 public subnet에 존재하여 public IP를 통해서
외부에서 접근할 수 있게 되면 보안입장에서 좋지 않기 때문이다. 따라서 간단한 웹 애플리케이션을 구동한다고 가정하면 RDS 인스턴스는 다음의 그림처럼 private subnet에 들어간다.

{{< imageFull src="/images/posts/aws/rds/aws_rds_with_private_subnet.png" class="middle-width" title="rds instance in private subnet" border="false" caption="rds instance in private subnet">}}

## 로컬에서의 접근 불가 문제

보안을 위해서 private subnet 안에 RDS 인스턴스를 생성한 것 까지는 좋은데, 한 가지 불편한 점이 있다. 바로 **나의 로컬 환경에서 RDS 데이터베이스에 접근이 안된다는 점이다.** public subnet에 있는 인스턴스에서만 RDS에 접근이 가능하기 때문에
나의 로컬 환경(노트북)에서는 private subnet 안에 있는 RDS 데이터베이스에 한번에 접근이 안되는 것이다. 생각해보면 당연한 것이지만, 개발환경에서 또는 일부 작업을 위해서 종종 RDS 데이터베이스에 직접 접속해야할 필요가 있는데 이 경우 매번
EC2 인스턴스를 거쳐서 접근해야하기 때문에 불편하다.

{{< imageFull src="/images/posts/aws/rds/can_not_access_direct_rds.png" class="middle-width" title="can not access direct to rds" border="false" caption="로컬 머신에서는 접근이 불가능하다.">}}

## AWS System Manager - AWS SSM

이런 문제를 좀 더 쉽게 해결하기 위해서 AWS에서는 `AWS System Manager(SSM)`(https://aws.amazon.com/ko/systems-manager/) 이라는 기능을 제공한다. (AWS System Manager 의 줄임말 인데 ASM 이 아니라 SSM인지는 모르겠다.) 
SSM 에는 기능이 여러가지가 있지만 그 중에서 `Session Manager`(https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html) 기능을 활용할 수 있다. 
쉽게 요약하자면, public subnet에 있는 EC2 인스턴스에 SSH를 통해서 접근한 다음에 RDS에 접근하는 방법을 간략하게 실행할 수 있게 만들어 놓은 편의 기능정도로 이해할 수 있다. (다른 기능들도 많지만, 이게 제일 많이 사용되는 것 같다.)

### SSM을 사용하기 위한 사전 조건

SSM을 사용하기 위해서는 먼저 다음의 준비사항들이 필요하다.

1. SSM-Agent 가 설치된 인스턴스가 있어야 한다.
   - amazone linux image를 기반으로 인스턴스를 생성했다면 이미 설치되어 있다.([링크](https://docs.aws.amazon.com/systems-manager/latest/userguide/ami-preinstalled-agent.html))
   - 따라서 대부분의 경우에는 특별한 작업이 필요없다. 
   - 필요하다면 SSM 을 위한 전용 인스턴스를 생성하기도 한다. (t3.micro 도 충분하다)
2. EC2 인스턴스에 부여된 Role(역할)에 SSM 사용을 위한 `AmazonSSMManagedInstanceCore` Policy 가 존재해야한다.
   {{< imageFull src="/images/posts/aws/rds/attach_ssm_policy_to_ec2_role.png" class="large-width" title="ssm policy for ec2 role" border="true" caption="EC2 인스턴스에 부여된 Role 에 SSM Policy를 추가한다.">}}
   - policy를 추가한 다음에는 ssm agent 재시작이 필요하다. 
   - amazone linux 라면 `sudo systemctl restart amazon-ssm-agent`
   - 다른 인스턴스 이미지라면 [매뉴얼](https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/ssm-agent-status-and-restart.html) 참고
3. AWS CLI 가 필요하다.
   - https://aws.amazon.com/ko/cli/
   - aws cli 사용을 위한 `aws configure` 는 미리 완료해놓아야 한다. 

### SSM 사용하기

SSM을 사용하려면 다음과 같이 입력하면 된다. 뒤에 붙는 target 은 SSM Agent 가 설치된 인스턴스 ID이다. 

```shell
> aws ssm start-session --target i-aabbccddeeffgg

Starting session with SessionId: aws-cli-user-hhiijjkkllmmnn
$ whoami
ssm-user
$ sudo su -l ubuntu
ubuntu@ip-10-232-193-202:~$
```

SSM을 사용하면 이렇게 ssh를 위한 key 가 없이도 EC2 인스턴스에 접근할 수 있다. 이후에 private subnet에 있는 인스턴스에 접근할 수 있다. 

### SSM 에서 리모트 포트포워딩 기능 사용하기

SSM 을 사용해서 에이전트가 설치된 EC2 인스턴스에 접근하는 것만으로는 충분하지 않기 때문에 포트포워딩 기능을 사용할 수 있다. 이 기능을 사용하면 다음과 같이 SSM Agent 가 설치된 인스턴스가 트래픽을 리모트 호스트로 포트포워딩해준다. 

```shell
aws ssm start-session --target $INSTANCE_ID \
                       --document-name AWS-StartPortForwardingSessionToRemoteHost \
                       --parameters '{"portNumber":["3306"],"localPortNumber":["3306"],"host":["aws-remote-rds-instance.aabbccddee.ap-northeast-2.rds.amazonaws.com"]}'
                       
Starting session with SessionId: aws-cli-user-0c2856a4240a4e548
Port 3306 opened for sessionId aws-cli-user-0c2856a4240a4e548.
Waiting for connections... 
```

### 데이터베이스 커넥션 테스트

이제 로컬 머신의 3306 포트에 연결하면 private subnet 에 있는 RDS 인스턴스(`aws-remote-rds-instance.aabbccddee.ap-northeast-2.rds.amazonaws.com`)의 3306 포트로 연결된다.

{{< imageFull src="/images/posts/aws/rds/datagrip-add-datasource.png" class="small-width" title="datagrip add datasource" border="false" caption="datagrip 에서 datasource 추가">}}
{{< imageFull src="/images/posts/aws/rds/datagrip-add-remote-rds-instance-datasource.png" class="middle-width" title="datagrip add remote rds instance datasource.png" border="false" caption="localhost 3306으로 연결하지만 실제로는 remote rds에 연결된다.">}}
{{< imageFull src="/images/posts/aws/rds/datagrip-test-connection.png" class="middle-width" title="test connection for localhost to remote host" border="false" caption="로컬 커넥션 테스트 OK (실제로 remote로 연결된다.)">}}

### 트러블슈팅

cli 명령어를 사용해 리모트 호스트로 포트포워딩하려는데 다음과 같이 `doesn't support port forwarding to remote hosts` 에러가 나오는 경우가 있다.

```shell
aws ssm start-session --target $INSTANCE_ID \
                       --document-name AWS-StartPortForwardingSessionToRemoteHost \
                       --parameters '{"portNumber":["3306"],"localPortNumber":["3306"],"host":["aws-remote-rds-instance.aabbccddee.ap-northeast-2.rds.amazonaws.com"]}'

An error occurred (BadRequest) when calling the StartSession operation: The version of the SSM agent installed on this instance doesn't support port forwarding to remote hosts. Install the latest version of the SSM agent to use port forwarding sessions to remote hosts.
```

이 에러의 원인은 SSM Agent 의 버전이 너무 낮아서 발생하는 문제이다. "AWS Systems Manager > 노드 관리 > 플릿 관리자" 에서 버전을 확인할 수 있다.

{{< imageFull src="/images/posts/aws/rds/ssm_manager_version_list.png" class="large-width" title="SSM Manager agent version list" border="false" caption="설치된 SSM Agent 들의 버전 목록을 확인할 수 있다.">}}

에이전트 버전이 **3.1.1374.0** 이상이어야만 리모트 호스트로의 포트포워딩이 가능하다. 에이전트의 버전을 올려보자. 제일 쉬운 방법은 수동으로 업데이트를 사용하는 방법이다.

{{< imageFull src="/images/posts/aws/rds/ssm_agent_instance_command_execute.png" class="middle-width" title="ssm instance command execution" border="false" caption="ssm agent 가 설치된 instance 에 명령을 실행한다.">}}
{{< imageFull src="/images/posts/aws/rds/update_ssm_agent_command.png" class="middle-width" title="update agent command" border="false" caption="ssm agent 가 설치된 instance 에 명령을 실행한다.">}}
{{< imageFull src="/images/posts/aws/rds/updated_ssm_agent_version.png" class="large-width" title="updated agent version" border="false" caption="ssm agent 버전이 올라갔다.">}}


## gossm 활용하기

aws cli 의 ssm 명령어를 사용하면 리모트 호스트로 접속이 가능하지만, 매번 인스턴스의 ID를 확인하고 명령어도 복잡하고 길어서 불편한점이 있다. 이를 간편하게 해결해주는 `gossm` 을 사용해보자.

### gossm 설치방법

```shell
$ brew tap gjbae1212/gossm
$ brew install gossm
```

### gossm 사용방법

다음과 같이 `gossm fwdrem` 명령어를 입력하면 ssm agent 가 설치된 인스턴스 목록이 표시된다. 

```shell
$ gossm fwdrem

region (ap-northeast-2)
? Choose a target in AWS:  [Use arrows to move, type to filter]
> my-ssm-agent-micro-instance	(i-aabbccddeeffgg)
some-ec2-instance-a	(i-aabbccddeeffgg)
some-ec2-instance-b	(i-abcedfghijklmn)
some-ec2-instance-c	(i-abcedfghijklmo)

```

이 다음에 ssm agent 로 사용할 인스턴스를 선택하면 포트포워딩을 연결할 포트번호(리모트, 로컬), 그리고 연결할 리모트 호스트를 입력받는다. 

```shell
$ gossm fwdrem

region (ap-northeast-2)
? Choose a target in AWS: my-ssm-agent-micro-instance	(i-aabbccddeeffgg)
? Remote port to access: 3306
? Local port number to forward: 3306
? Type your host address you want to forward to: aws-remote-rds-instance.aabbccddee.ap-northeast-2.rds.amazonaws.com
[start-port-forwarding 3306 -> 3306] region: ap-northeast-2, target: i-aabbccddeeffgg

Starting session with SessionId: aws-cli-user-0c2856a4240a4e548
Port 3306 opened for sessionId aws-cli-user-0c2856a4240a4e548.
Waiting for connections...

```

그 다음은 `aws ssm` 명령어와 동일하게 로컬에서 데이터베이스 연결을 수행할 수 있다.

> aws 명령과 gossm 모두 aws configure 가 설정된 상태에서 사용이 가능하다.

## 결론

결론적으로 ssm (gossm)을 사용하면 private subnet 에 있는 RDS 인스턴스에 접근하여 데이터베이스 작업을 수행할 수 있다.

1. ssm을 사용하면 기존의 복잡한 ec2 인스턴스를 통한 ssh 연결을 대체하여 손쉽게 인스턴스를 관리할 수 있다
2. ssh 키를 관리하지 않아도 된다.
3. 리모트 포트포워딩 기능을 사용하여 Private RDS 인스턴스에 접근할 수도 있다.
4. ssm 명령어가 복잡하다면 gossm 을 활용할 수 있다.


## 참고자료 
 - https://musma.github.io/2019/11/29/about-aws-ssm.html#%EC%8B%A4%EC%8A%B5-aws-system-manager-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0
 - ssm 포트포워딩 지원 발표(https://aws.amazon.com/ko/about-aws/whats-new/2022/05/aws-systems-manager-support-port-forwarding-remote-hosts-using-session-manager/)
 - [gossm](https://github.com/gjbae1212/gossm)
 - [AWS SSM Agent 버전 확인 방법](https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/ssm-agent-get-version.html) 
