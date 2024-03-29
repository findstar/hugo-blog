---
date: '2022-08-21 16:13:22 +09:00'
group: blog
image: /images/posts/aws/eks/aws-eks.jpg
tags: ["aws", "eks", "instance type"]
title: "EKS를 구축할 때 어떤 인스턴스 타입이 적합할까?"
url: /2022/08/21/which-instance-type-is-right-for-EKS
type: post
summary: "AWS에서 EKS 클러스터를 생성할 때 어떤 인스턴스 타입이 적당한지 고민이 되었는데, 인스턴스를 선택하는 과정을 정리해보았다."
---

# AWS EKS 에서 인스턴스 타입 선택하기

새로운 프로젝트에 AWS를 사용하기로 하면서 어떻게 인프라를 구성할까 고민했다. 우선 개발팀 멤버 모두가 K8S 환경에 익숙한 상태였기 때문에 제일먼저 EKS를 사용하기로 결정했다.
다른 AWS 서비스들에 대한 구상을 마치고, EKS 클러스터를 설정하기 위한 방법을 알아보았다. 그 중에서 노드를 어떻게 관리할지에 대한 내용을 정리해보았다.


## EKS에서 노드를 관리하는 3가지 방법 

EKS에서 노드를 관리하는 방법은 3가지가 있다.
1. **노드그룹을 통해서 사용하는 방법**은 노드(VM-EC2 Instance)를 그룹핑하여 EKS가 관리하게 하는 방법으로 인스턴스의 타입, 인스턴스의 갯수등을 지정하기만 하면 된다.
2. **Fargate를 사용하는 방법**은 Pod에 정의된 리소스 설정에 맞게 AWS에서 알아서 인스턴스를 배정해준다(1 pod / 1 instance)
   - 두 가지 방법을 섞어서 사용할 수도 있다.
3. **노드를 직접 관리하는 방법**은 잘 권장되지 않는데, 워커노드의 K8S 연결을 위한 kubelet, kube-proxy 를 고려해야하기 때문이다.

노드의 직접 관리를 제외하고 두 가지 방법중에 어떤 방법이 더 적합할지 고민하다가 결과적으로 **노드 그룹으로만 관리**하고 Fargate는 사용하지 않기로 하였는데 그 이유는 다음과 같다. 
 - Fargate는 Pod의 vCPU와 메모리 조합이 정해져 있다. 그래서 일반적인 애플리케이션 구동에는 문제가 없지만, CPU를 많이 사용하거나 메모리가 많이 필요한 애플리케이션을 구동하기에는 적절하지 않다.
 - Node를 직접관리하지 않기 때문에 편하다는 장점이 있지만, Node 에 DaemonSet을 설치하여 노드 자체의 모니터링을 수행하는 등의 작업을 수행할 수 없다. (보안관점에서 별도로 노드를 모니터링 하는 경우가 있다.)
 - DaemonSet 대신 사이드카를 사용할 수 있지만 사이드카 컨테이너가 Pod의 리소스(vCPU, Memory)를 일부 사용하게 된다.
 - 그리고 Fargate는 public subnet 을 사용할 수 없고, NLB을 사용할 수 없다는 점도 고려하였다.

따라서 노드 관리는 "**노드 그룹**"을 직접 생성하여 관리하기로 하였다.

## 노드 그룹을 통해서 노드를 관리할 때 인스턴스 타입 결정

노드 그룹을 통해서 노드를 관리하고자 결정한 다음에는 **어떤 인스턴스 타입을 적용해야할지 고민**되었다.
물론 서비스의 특성에 따라, 애플리케이션이 필요로 하는 성능에 따라, 예상 트래픽등 다양한 변수에 따라서 달라지겠지만, 기준점을 정해놓고 싶었다.
먼저 인스턴스의 유형을 다시한번 정리해보았다.

## AWS EC2 인스턴스 유형
EC2 인스턴스는 기능에 따라서, 그리고 가격에 따라 구분된다.

* 기능별 구분
: 인스턴스를 표현할 때 흔히 `t3.micro`, `m4.large`, `c2.medium` 와 같이 표현하는데 이런 표현은 기능별 유형을 나타낸다. 
예를들어 `c2.medium` 이라면 `c`는 인스턴스 family 라고 해서 어떤 특성을 가진 compute 자원인지 나타낸다.
`2` 는 몇세대 인스턴스인지 표시한다. 여기서는 2세대 인스턴스라는 의미가 된다. 시간이 지남에 따라 AWS에서는 새로운 세대의 인스턴스가 계속 출시된다.
  (새로운 세대가 나오면 성능이 향상되기 마련이라, 가성비가 좋아진다.) `medium`은 인스턴스의 사이즈를 나타낸다.
따라서 `c2.medium` 이라는 표현은 컴퓨팅에 최적화된 2세대 Medium 사이즈의 인스턴스 라는 의미가 된다.
  - family 구분
    - M : 범용 
    - C : 컴퓨팅 최적화
    - G : GPU 최적화
    - R : 메모리 최적화
    - .. 기타 여러가지 구분이 더 있다.
  - AWS 인스턴스 타입 비교 페이지(https://aws.amazon.com/ko/ec2/instance-types/)

* 가격별 구분
: 기능별 구분이외도 가격을 기반으로 구분할 수도 있다. 인스턴스를 기본 생성하면 온디멘드 유형의 인스턴스를 사용하게 된다. 
  - 온디멘드 인스턴스 : 필요할 때 사용하는 기본적인 인스턴스 유형
  - 리저브 인스턴스 : 핸드폰에 비유하면 약정을 걸고 사용하는 유형이다. 2년 혹은 3년 사용을 보장하고 좀 더 저렴한 비용으로 인스턴스를 사용한다. 오래 사용한다면 당연히 좋아보이지만, 새로운 세대의 인스턴스가 나오면 가성비가 떨어질 수 있다는 점을 고려해야한다. 
  - 스팟 인스턴스 : 온디멘드와 비교하여 최대 90% 저렴하게 사용할 수 있는 인스턴스. 
    AWS 클라우드 전체로보면 리전 내부에 사용하지 않는 인스턴스 자원이 있을 것이고 이 남는 인스턴스를 저렴하게 사용하는 형태가 바로 스팟 인스턴스 이다. 
    나의 인프라스트럭처를 모두 스팟 인스턴스로 구성하면 당연히 저렴한 비용으로 인프라를 구축할 수 있지만, 리전내부에 스팟인스턴스 자원이 남지 않는 경우도 있으니 주의가 필요하다.

## EKS에서 사용할 때의 인스턴스 고려사항

기능과 가격구분에 대해서 이해하였으니 이제 EKS에서 사용할 때의 고려사항을 정리해보았다.

- EKS 쿠버네티스의 Pod는 한 개 이상의 컨테이너를 구성하고
  같은 Node의 Network 스택을 공유한다.
- 그리고 클러스터 내부의 여러 Node에 걸쳐 생성된 Pod은
  Overlay Network를 통해 서로 통신한다.
- 이 말은 Pod는 AWS VPC의 네트워크를 차지한다는 뜻이 된다.
- 즉 하나의 EC2 인스턴스(노드) 가 차지하는 네트워크 IP가
  여러개가 될 수 있다.
- 그런데 EC2 인스턴스는 타입 사이즈에 따라서 연결할 수 있는 네트워크 인터페이스 (ENI)의 갯수가 정해져있고, 
  [매뉴얼 링크](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI)
- 사이즈에 따라 사용할 수 있는 네트워크 대역폭도 차이가 있다.
- 따라서 xlarge 이상의 사이즈를 선택해야 POD를 배포했을 때 무리 없이 활용할 수 있다.
  (너무 작은 사이즈는 EKS 노드로 활용하기 힘들다는 말이 된다. 실제 노드 그룹을 생성할 때 `t3.nano` 같은 타입은 선택 옵션에 나오지도 않는다.)

{{< imageFull src="/images/posts/aws/eks/instance-selection.png" class="large-width" title="eks 노드 그룹의 instance 선택" border="false" caption="eks 노드 그룹의 instance 선택">}}

CLI 에서 다음 명령어를 입력하면 테이블 형태로 CPU, Memory, ENI, IP 수를 확인할 수 있다. 
```shell
aws ec2 describe-instance-types --filters "Name=instance-type,Values=m6*" 
   --query "InstanceTypes[].{Type: InstanceType, MaxENI: NetworkInfo.MaximumNetworkInterfaces, IPv4addr: NetworkInfo.Ipv4AddressesPerInterface, VCpuInfo: VCpuInfo.DefaultVCpus, MemoryInfo: MemoryInfo.SizeInMiB}" 
   --output table


------------------------------------------------------------------
|                      DescribeInstanceTypes                     |
+----------+---------+-------------+----------------+------------+
| IPv4addr | MaxENI  | MemoryInfo  |     Type       | VCpuInfo   |
+----------+---------+-------------+----------------+------------+
|  30      |  8      |  131072     |  m6i.8xlarge   |  32        |
|  15      |  4      |  16384      |  m6g.xlarge    |  4         |
|  50      |  15     |  262144     |  m6g.metal     |  64        |
|  50      |  15     |  393216     |  m6i.24xlarge  |  96        |
|  10      |  3      |  8192       |  m6i.large     |  2         |
|  30      |  8      |  65536      |  m6g.4xlarge   |  16        |
|  30      |  8      |  65536      |  m6i.4xlarge   |  16        |
|  50      |  15     |  524288     |  m6i.metal     |  128       |
|  30      |  8      |  131072     |  m6g.8xlarge   |  32        |
|  10      |  3      |  8192       |  m6g.large     |  2         |
|  50      |  15     |  262144     |  m6i.16xlarge  |  64        |
|  50      |  15     |  524288     |  m6i.32xlarge  |  128       |
|  50      |  15     |  262144     |  m6g.16xlarge  |  64        |
|  30      |  8      |  196608     |  m6g.12xlarge  |  48        |
|  15      |  4      |  16384      |  m6i.xlarge    |  4         |
|  15      |  4      |  32768      |  m6g.2xlarge   |  8         |
|  30      |  8      |  196608     |  m6i.12xlarge  |  48        |
|  15      |  4      |  32768      |  m6i.2xlarge   |  8         |
|  4       |  2      |  4096       |  m6g.medium    |  1         |
+----------+---------+-------------+----------------+------------+

```

## EKS 에서 사용하기로한 기준 인스턴스 

결과적으로 우리 팀에서 사용하기로한 기준 인스턴스 타입은 `m6i.xlarge` - 온디멘드 유형으로 결정하였다. 
- 인스턴스 Family 중에서 범용타입인 `m` 타입으로 결정한다. (현재 애플리케이션 개발 단계라 `c`, `r` 타입보다는 범용성을 고려하기로 했다.)
- `m` 타입중에서 최신 세대인 6세대를 선택하기로 했다. (가성비가 제일 뛰어나다.)
- 6세대 `m` 타입은 다시 `m6i`, `m6a`, `m6g` 로 나뉘는데 각각 `intel`, `amd`, `arm(Graviton2)` cpu를 의미한다. 
- cpu 중에서는 가장 일반적인 `intel` 을 선택했다. (arm을 써볼까 했지만.. 선택안했다. )
- 사이즈는 `xlarge` 를 기준점으로 삼기로 했다. 

### 덧) arm 인스턴스(graviton)를 사용하지 않는 이유
AWS에서 자체적으로 만든 ARM 기반의 CPU 인스턴스 Graviton은 다른 intel 대비 가성비가 좋다고 설명하고 있다.
하지만 실제 produciton 쓰기에는 다음과 같은 우려점 때문에 선택하지 않았다.

- ARM 인스턴스는 별도의 빌드 과정이 필요했다. (우리는 Docker를 기반으로 배포전략을 구성했는데 자체 구축된 Docker build 시스템에서 ARM을 지원하지 않았다.)
- CPU, Memory 는 인텔 대비 동일했지만, 네트워크 대역폭 지원등이 상대적으로 부족했다. 

# 결론

1. EKS에서 POD도 VPC 내의 IP를 차지하고 
NODE (EC2 Instance) 가 가질 수 있는 네트워크 ENI는
사이즈에 따라 갯수 제한이 있어서 네트워크 관점에서 최소 xlarge 를 선택

2. 인스턴스 Family는 범용타입인 M 선택

3. CPU 유형은 제일 무난한 인텔

4. 최종 선택은 `m6i.xlarge` 

기준이 되는 인스턴스를 정했지만, 이 기준은 만들고자 하는 애플리케이션의 특성, 예상 트래픽 정도, 인프라의 구성에 따라 달라질 수 있다. 
만약 자신만의 인프라스트럭처를 고민하고 있다면 참고가 되길 바란다. 


# 참고링크 
- AWS 인스턴스 타입 비교 페이지(https://aws.amazon.com/ko/ec2/instance-types/)
- Graviton 소개 페이지 (https://aws.amazon.com/ko/ec2/graviton/) 
- 인스턴스 타입의 네트워크 퍼포먼스 소개 페이지 (https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/general-purpose-instances.html#general-purpose-network-performance) 
