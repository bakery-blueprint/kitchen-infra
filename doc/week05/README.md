# 인프라를 지탱하는 으용 이론

## 1. 캐시

Cache란 나중에 요청할 결과를 미리 저장해둔 후 빠르게 서비스 해주는 것을 의미한다.

매커니즘은 파레토 법칙이 있다. 

파레토 법칙 : 80% 결과는 20%의 원인으로 발생한다. 즉 많이 발생하는 20% 캐싱하면 극대화한다.

https://namu.wiki/w/%ED%8C%8C%EB%A0%88%ED%86%A0%20%EB%B2%95%EC%B9%99

### 로컬 vs 글로벌

- Local Cache
  - 서버마다 캐시를 따로 저장한다.
  - 다른 서버의 캐시를 참조하기 어렵다.
  - 서버 내에서 작동하기 때문에 속도가 빠르다.
  - 로컬 서버 장비의 Resource를 이용한다. (Memory, Disk)
  - 캐시에 저장된 데이터가 변경되는 경우:
    - 해당 서버를 제외한 모든 peer에 변경 사항 전달
    - All-to-All Replication
    - WAS 인스턴스가 늘어나고, 캐시 저장 데이터 크기가 커지면 성능이 저하되는 이유는 이 때문
- Global Cache
  - 여러 서버에서 캐시 서버에 접근하여 참조 할 수 있다.
  - 별도의 캐시 서버를 이용하기 때문에 서버 간 데이터 공유가 쉽다.
  - 네트워크 트래픽을 사용해야 해서 로컬 캐시보다는 느리다.
  - 데이터를 분산하여 저장 할 수 있다.
    - Replication: 두 개의 이상의 DBMS 시스템을 Mater / Slave 로 나눠서 동일한 데이터를 저장하는 방식
    - Sharding: 같은 테이블 스키마를 가진 데이터를 다수의 데이터베이스에 분산하여 저장하는 방법
  - 캐시에 저장된 데이터가 변경되는 경우:
    - 추가적인 작업 필요없음
    - 서비스 확장으로 WAS 인스턴스가 늘어나고, Cache 데이터 크기가 커질 수록 효과적인 이유



### 강대명 캐시의 모든것

- 애플리케이션 단에 메모리(로컬 캐시)랑 별도의 인메모리 DB쓰는 거랑 차이점?

  A : 로컬 캐시가 가장 빠르지만, 동기화가 어렵다. 

- 트랜잭션과 캐시

  A : 트랜잭션 개념을 구현하기 힘들다. 

- 캐시가 죽었을때 가정을 해봤을까..??

  A: 캐시가 죽으면 DB까지 죽을 경우를 대비해야한다. 	

https://youtu.be/zkbvFOwJFgA



## 2. 인터럽트

컴퓨터가 작업을 수행하던 도중 예기치 못한 특수한 상황이 발생하여 작업을 중단하고, 그 특수한 상황을 먼저 처리한 후, 원래의 작업으로 되돌아가 나머지 작업을 계속 수행하게 되는 일련의 과정이다. 

**하드웨어 인터럽트와 소프트웨어 인터럽트로 나뉜다. **

1. 하드웨어 인터럽트 : 컨트롤러, 하드웨어 장치가 cpu 인터럽트 라인에 세팅한다. 

2. 소프트웨어 인터럽트 : 소프트웨어가 수행

- cpu는 프로그램 카운터가 가리키는 곳에 있는 명령을 수행하는 일 밖에 없기 때문에 

    현재 수행 중인 프로세스로부터 cpu를 회수해 cpu가 다른 일을 수행하도록 하기 위해선 인터럽트 메커니즘이 필요하다. 

### 소프트웨어 인터럽트

통상적으로 인터럽트라고 말하면 하드웨어 인터럽트를 의미하고 (타이머, 입출력)

소프트웨어 인터럽트는 보통 트랩(trap)이라는 용어로 부른다. 예외상황(0으로 나누기)과 시스템콜이 있다.

인터럽트가 발생할 때만 운영 체제 코드 부분으로 cpu가 이양되어 인터럽트 처리를 하게 된다. 

운영체제가 직접 cpu 제어하는 경우는 인터럽트 말고는 없다. 

**"시스템 콜"**은 운영체제의 서비스를 받기 위해  운영체제에 정의된 함수(커널영역에 있는)를 호출하는 것을 말한다. 

사용자가 운영체제의 서비스(특권 명령, 즉 커널 함수를 호출하기 위해)를 요청하기 위해 커널의 함수를 호출하는 것.

- 시스템콜 처리 과정

  1. 디스크 파일을 읽어와야 할 경우 시스템 콜을 한다. 

  2. 인터럽트 라인에 명령을 세팅한다. 

  3. cpu는 다음 명령을 수행하기전 인터럽트 라인을 체크한다. 

  4. 수행 중인 프로그램을 멈추고 , 상태를 저장하고 , 인터럽트를 발생시킨다. 모드비트 1로 변경

  5. 인터럽트 백터를 통해 인터럽트 처리 루틴을 찾아 이동한다. 

  6. 데이터를 읽어오는 일을 컨트롤러, 로컬버퍼에게 맡기고, cpu 제어권을 다른 프로세스에게 넘긴다. 

  8. 작업이 완료되면 컨트롤러는 인터럽트 라인을 설정한다. 

  9. cpu는 명령어를 수행 하기전에 인터러트 라인을 체크하고 인터럽트가 발생한다. 하던 것을 멈추고 상태 저장

  10. 로컬버퍼에서 메모리로 얻어오고, 봉쇄당한 프로세스를 다시 준비 상태로 만들어 준비 큐에 넣는다.

  11. 다시 인터럽트 당했던 프로세스에게 넘어가 하던 작업을 수행한다.  

  12. 항상 명령어를 수행 하기전 모드비트를 검사한다. 

: 모드비트는 하드웨어 보안으로 사용자, 커널모드가 접근 구분 
