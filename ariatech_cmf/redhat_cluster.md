# Red Hat Customer Portal - PCS COMMAND

## PCS 명령 인터페이스

일반적인 pcs command interface는 다음과 같다

```
# pcs [-f 파일] [-h] [명령]...
```



__PC command__

- cluster
  - 클러스터 옵션 및 노드를 구성, pcs cluster명령은 4장에서 자세히 다룬다
- resource
  - 클러스터 리소스를 만들고 관리, pcs cluster 명령에 대한 내용은 6,8,9장 참조
- stonith
  - Pacemaker와 함께 사용할 차단장치를 구성, 5장참조
- constraint
  - 리소스 제약 관리 pcs constraint는 7장 참조
- property
  - Pacemaker 속성을 설정 12장 참조
- status
  - 현재 클러스터 및 리소스 상태를 확인 3.5절 참조
- config
  - 사용자가 읽을 수 있는 형식으로 전체 클러스터 구성 표시 3.6장 참조



```
# pcs status [resources, groups, cluster, nodes ...]
## 클러스터 및 클러스터 리소스의 상태를 표시
```
> 
```
# pcs config
## 현재 클러스터 구성의 전체 표시
```
>
---

## 클러스터 생성 및 관리

__클러스터 생성__

1. pcs 클러스터의 각 노드에서 시작한다
2. 클러스터를 구성할 노드를 인증한다
3. 클러스터 노드를 구성하고 동기화한다
4. 클러스터 노드에서 클러스터 서비스를 시작한다

__pcsd데몬 시작__

```
## 서비스를 시작하고 시스템 시작시pcsd활성화, 이 명령은 클러스터의 각 노드에서 실행해야 한다
# systemctl start pcsd.service systemctl enable pcsd.service
```

__클러스터 노드 인증__

```
## 다음 명령은 클러스터의 노드에서 데몬을 인증한다
## pcs 관리자의 사용자 이름은 hacluster의 모든 노드에 있어야 한다
## hacluster 사용자의 암호는 각 노드에서 동일한것이 좋다
## username또는 password를 지정하지 않으면 명령을 실행할 때 시스템에서 각 노드에 대한 해당 매개변수를 묻는 메시지를 표시한다
## 노드를 지정하지 않으면 이전에 해당 명령을 실행한 경우 이 명령이 해당 명령 pcs로 지정된 노드에서 인증한다
# pcs cluster auth [...] [-u username] [-p password]
## 예를들어 아래 명령은 두 노드에 사용자 암호를 묻는 메시지를 표시한다
# pcs cluster auth z1.example.com z2.example.com
```

__클러스터 노드 구성 및 시작__

```
## 다음 명령은 클러스터 구성 파일을 구성하고 구성을 지정된 노드와 동기화한다
## --start명령은 지정된 녿에서 클러스터 서비스를 시작한다
## 필요한 경우 pcs cluster start명령으로 클러스터를 시작할 수 있다
## 하지만 작업전 pcs cluster status명령을 사용하여 클러스터가 이미 실행중인지 확인하는 것이 좋다
# pcs cluster setup [--start] --name cluster_name node1 [node2
# pcs cluster start [--all] [node] [...]
```

__중복링 프로토콜(RRP)구성(REDUNDANT RING PROTOCOL)__

```
# pcs cluster setup --name my_rrp_cluster(클러스터이름) nodeA-0,nodeA-1, nodeB-0,nodeB-1
```



 ## 클러스터 리소스 관리

__리소스 생성__

```
# pcs resource create resource_id [standard:[provider:]]type [resource_options] [op operation_action operation_options [operation_action operation options]...] [meta meta_options...] [clone [clone_options] | master [master_options] | --group group_name [--before resource_id | --after resource_id] | [bundle bundle_id] [--disabled] [--wait[=n]]

```

옵션을 지정하면 --group 리소스가 명명된 시소스 그룹에 추가된다

그룹이 없으면 그룹이 생성되고 리소스가 추가된다

--before, --after 옵션은 그룹에 이미 존재하는 리소스를 기준으로 추가된 리소스의 위치를 지정한다

옵션을 --disabled하면 리소스가 자동으로 시작되지 않음을 나타낸다

```
## 다음 명령은 이름virtual가 standard, ocf provider heartbeat 타입 리소스를 만든다. 이 리소스의 유동주소는 192.168.0.120이며 시스템 리소스는 30초마다 실행 중인지 확인한다
# pcs resource create VirtualIP ocf:heartbeat:IPaddr2 ip=192.168.0.120 cidr_netmask=24 op monitor interval=30s
```

```
## 또는 standard, provider를 생략하고 아래처럼 사용할수 있으며 이는 standard, ocf provider로 설정된다
# pcs resource create VirtualIP IPaddr2 ip=192.168.0.120 cidr_netmask=24 op monitor interval=30s
```

```
## 다음 명령은 구성된 리소스를 삭제하며 두번째는 리소스id가 virtualip인 리소스를 삭제한다
# pcs resource delete resource_id
# pcs resource delete VirtualIP
```

>

- resource_id = 리소스의 이름

- standard = 스크립트가 준수하는 표준. ocf, service, upstart, systemd, lsb, stonith를 포함

- type = 사용하려는 리소스 에이전트 이름, IPaddr 또는 Filesystem

- provider = ocf사양을 통해 여러 공급업체가 동일한 리소스 에이전트를 제공할 수 있다. 대부분의 에이전트heartbeat는 공급자로 사용

```
## 사용 가능한 모든 리소스 목록을 표시
# pcs resource list
## 사용 가능한 리소스 에이전트 표준 목록을 표시
# pcs resource standards
## 사용 가능한 리소스 에이전트 공급자 목록을 표시
# pcs resource provider

```

> ![image-20220119151339381](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20220119151339381.png)








