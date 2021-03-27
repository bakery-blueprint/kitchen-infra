# 제 6장 시스템을 연결하는 네트워크 구조

## 6.1 네트워크

데이터는 네트워크를 경유해서 전달



## 6.2 계층 구조

계층 간 역할이 나누어지며, 자신이 담당하는 일만 책임. 다른 일은 다른 계층이 책임진다 (다른 계층이 어떤 방식으로 일 처리 하는지 모름)

독립적으로 동작함. 인터페이스만 바꾸지 않으면 각 계층의 내부 처리가 바뀌는것은 상관 X



대표적인 계층 모델 ```OSI 7계층 모델(OSI 참조 모델)```



### 6.3 프로토콜

컴퓨터가 서로 소통하기 위해 정한 규약

통신 시 사용하는 매체도 프로토콜. 통신 매체(전파, 전기신호) / 의미 부분(HTTP)

TCP/IP, USB 프로토콜, SCSI 프로토콜 등



### 6.4 TCP/IP를 이용하고 있는 현재의 네트워크

현재 네트워크를 지탱하는 것은 TCP/IP 및 관련 프로토콜이다. (모아서 TCP/IP 프로토콜 스위트 라고 함)

시초인 ARPANET https://namu.wiki/w/%EC%95%84%ED%8C%8C%EB%84%B7?from=ARPAnet



네트워크 제조사별로 독자 프로토콜을 사용하고 있어서 상호접속에 문제가 있었음.

국제 규격의 프로토콜을 만들자, OSI 프로토콜!

하지만 이미 TCP나 IP프로토콜이 보급되어 국제 규격으로 OSI 사용을 권장했지만 사장됨



TCP/IP는 TCP와 IP 두 개의 프로토콜을 주축으로 한 프로토콜 집합

![img](../img/tcp4layer.png)





### 6.5 [레이어7] 애플리케이션 계층의 프로토콜 HTTP

애플리케이션이 사용하는 프로토콜 ([HTTP](https://tools.ietf.org/html/rfc2616))

통신 자체는 OS(TCP/IP)에게 맡긴다.



1. 브라우저에 URL 입력
2. 요청이 전송 (to 웹서버)
3. 요청 내용 해석
4. 파일이나 이미지를 응답으로 반환
5. 파일 내부를 해석해서 이미지, 스크립트 등이 포함돼 있으면 다시 요청

클라이언트 <-> 웹서버 HTTP를 통해 몇 번이고 왕복하여 요청, 응답을 주고 받음



HTTP의 request / reponse에 포함되는 내용

request

* 명령 (GET, POST)
* 헤더 (브라우저, 쿠키 등)
* 바디 (브라우저에 입력한 내용)



response

* 요청에 대한 상태 결과
* 헤더
* 바디 (실제 데이터)



HTTP가 하위 계층(IP)이나 유선을 통해 명령을 보내거나 통신 제어를 하지 않기 때문에 매우 간단한 구조로 구성되어있음



애플리케이션 계층 프로토콜은 필요한 데이터를 소켓에 기록만 하며, 통신은 모두 TCP/IP에 위임한다.

HTTP 요청은 TCP/IP를 통해 상대 서버까지 전송됨





### 소켓 이하 처리는 어떻게?

애플리케이션은 커널에게 통신을 위해 회선을 열어달라고 의뢰(시스템 콜!)

접속 대상 서버의 IP주소와 TCP포트 정보가 필요함.

1. 커널은 소켓을 만들어 준다
2. TCP 포트, IP 주소 및 포트 번호 정보를 커널에 전달.

~~ 접속 대상 서버와의 연결 생성 ~~

3. 상대방 서버에도 소켓이 만들어지며 가상 경로가 생성됨 (virtual circuit)

   물리적인 개념으로는 통신 케이블을 통해 상대방에게 전달되지만 프로세스 관점에서는 소켓을 통한 가상경로로 설명



### 6.6 [레이어4] 전송 계층 프로토콜 TCP

애플리케이션이 보낸 데이터를 그 형태 그대로 상대방에게 확실하게 전달하는 것

서버가 송신할 때와 서버가 수신한 후 애플리케이션에게 전달하는 것까지만 담당. 상대 서버까지 전송하는 부분은 하위 계층(IP)에 위임

* 포트 번호를 이용해서 데이터 전송
* 연결 생성
* 데이터 보증과 재전송 제어
* 흐름 제어와 폭주 제어

제대로 전달됐는지 확인, 도착한 순서를 확인



소켓에 기록된 데이터는 큐를 경유해서 소켓 버퍼에서 처리됨.

애플리케이션 데이터에 TCP 헤더를 붙여서 TCP 세그먼트(데이터 단위)를 작성

MSS : 하나의 TCP 세그먼트로 전송할 수 있는 최대 데이터 크기

​	MSS를 초과한 데이터는 자동적으로 분할돼서 복수의 TCP 세그먼트가 생성됨

MTU : 링크 계층(네트워크 계층)에서 전송할 수 있는 최대 데이터 크기

![그림6.15](../img/그림6.15.jpeg)







상대 서버에 데이터가 도착한 후, 어떤 애플리케이션으로 전달할까?

TCP에서는 포트 번호를 사용해서 상대 서버에 데이터가 도착했을 때 어떤 애플리케이션에 데이터를 전달할지 판단한다. 범위 : 0~65535





위에서 잠깐 얘기나온 가상 경로(virtual circuit)! TCP는 연결형 프로토콜로 가상 경로를 생성한다.

TCP/IP의 3-way handshaking

SYN : synchronize sequence numbers

ACK : acknowledgment

![img](https://t1.daumcdn.net/cfile/tistory/225A964D52F1BB6917)



TCP 통신을 시작할 때 상대 서버에 포트 번호와 연결을 열어달라고 부탁만 할 뿐 다른 특별한 일은 하지 않는다.







TCP의 데이터가 확실히 전달되도록 보증하는 기능?



#### 데이터 손실을 방지하는 구조

ACK가 돌아오는 것을 보고 전송한 세그먼트가 무사히 도착했다는 것을 인지

재전송이 가능하도록 ACK가 돌아오기전까지는 소켓 버퍼에 세그먼트를 남겨둠



#### 데이터 순서를 보증하는 구조

시퀀스 넘버를 활용 (TCP 헤더에 기록). 해당 세그먼트가 가지고 있는 데이터가 전송 데이터 전체 중 몇 바이트째부터 시작하는 부분인지를 가르킴

ex. 3000바이트를 보내면 1460/1460/80 바이트로 쪼개어 세그먼트가 분할됨.(198페이지 참고)

각 시퀀스 번호는 1/1461/2921로 순서를 판별할 수 있음



#### TCP 재전송 제어

어느 시점에 재전송?

1. 타임아웃 - 일정 시간내에 ACK가 돌아오지 않으면 재전송
2. 중복 ACK (그림 6.18) Selective ACK

![image-20210323132228507](../img/tcp재전송제어.png)



---

#### 흐름 제어

TCP는 어느 정도의 세그먼트 수라면 ACK를 기다리지 않고 전송함 (윈도우)

수신 측에서 수신 윈도우와 송신 윈도우 중 작은 쪽을 송신 윈도우로 채택. 이 범위 내에서는 ACK를 기다리지 않고 전송

ACK가 오면 송신용 소켓 버퍼에서 삭제하고 송신 윈도우를 다음으로 이동함 - 슬라이딩 윈도우



#### 폭주 제어

송신 측 윈도우 크기는 네트워크 폭주 상태에 맞추어 변경됨.

네트워크가 혼잡하면 송신 윈도우(폭주 윈도우)를 작게 해서 전송 데이터 양을 줄임.

폭주 윈도우 크기는 통신 시작시 1세그먼트에 설정됨. 통신이 문제 없이 시작돼서 수신측에 도착하면 ACK 반환 시마다 폭주 윈도우 크기를 2, 4... 지수 함수적으로 늘려감.

슬로우 스타트(Slow Start)

어느 정도 증가하면 그 이후부터는 1세그먼트씩 크기를 늘려나감. 폭주를 감지하면 다시 윈도우 크기를 작게





## 6.7 [레이어3] 네트워크 계층의 프로토콜 IP

IPv4, IPv6

지정한 대상 서버까지 전달받은 데이터를 전해 주는 것

* IP 주소를 이용해서 최종 목적지에 데이터 전송
* 라우팅

최종 목적지가 적힌 IP 헤더(데이터 길이, 프로토콜 종류, 헤더 체크섬, TTL 등)를 TCP세그먼트에 추가해서 IP 패킷을 생성



네트워크부 + 호스트부

같은 네트워크는 네트워크부가 동일함.



가정이나 회사 내에서 사용하는 사설 네트워크.

인터넷상에서 통신이 가능한 공공 IP.



대상 서버가 다른 네트워크에 있는 경우 목적지를 알고 있는 라우터에 전송을 부탁함.

IP패킷을 받은 라우터는 헤더에서 목적지를 확인해서 어디로 보내야할지를 판별함. 이때 라우팅 테이블이라는 것을사용한다.

외부와 접속하는 네트워크는 보통 기본 게이트웨이 라는 라우터가 설치돼있음.

라우터가 잘못된 라우팅 테이블을 가지고 있어서 라우터 같에 패킷을 왕복하는 경우가 발생하면...?

​	-> 이런 상태를 방지하기 위해 IP 헤더에 TTL 정보를 갖고 있음

TTL은 라우터를 경유할때마다 1씩 차감하기 때문에, 체크섬을 재계산 해야함 (헤더 정보에 변경이 있으면 체크섬 재계산)

이 때문에 IPv6에서는 체크섬이 제외됨.





## 6.8 [레이어2] 데이터 링크 계층의 프로토콜 이더넷

대표적인 프로토콜 __이더넷__



동일 네트워크 내의 네트워크 장비까지 전달받은 데이터를 운반하는 역할을 함

MAC주소를 사용하여 자신이 포함된 링크 내(동일 네트워크)에서만 데이터를 전송할 수 있음



MAC주소와 IP주소의 대응 관계를 기록한 표 : ARP 테이블

IP패킷에 이더넷 헤더와 푸터를 붙여서 __이더넷 프레임__ 을 만듦



MTU : 링크 층에서 하나의 프레임으로 전송할 수 있는 최대 크기 (일반적으로 이더넷에서는 1500바이트)



1) IP 패킷의 최대 크기가 MTU보다 작은 경우

전달할 수 있는 데이터 크기가 작기 때문에 통신 횟수가 늘어남

2) IP 패킷의 최대 크기가 MTU보다 큰 경우

패킷 분할이 이루어짐. 





전체에 데이터를 전송하는 브로드캐스트 통신 같은 경우는 불필요한 트래픽을 증가시킴

물리 구성에 좌우되지 않고 설정만으로 네트워크를 나눌 수 있는 구조. __VLAN__

가상적으로 네트워크를 나누는 구조, VLAN ID라 불리는 숫자로 관리.



## 6.9 TCP/IP를 이용한 통신 이후

L2, L3 스위치를 경유해서 최종 목적지 서버에 이더넷 프레임이 도착하면!

1. NIC 수신 큐에 저장해서 인터럽트 or 폴링을 이용해서 커널 내에 프레임을 복사
2. 이더넷 헤더와 푸터를 제거하고 IP패킷을 꺼냄
3. IP주소를 확인해서 자신에게 보낸 패킷인지 확인
4. (나에게 온게 맞다면) IP 헤더를 제거하고 TCP 세그먼트 꺼내기
5. TCP 포트 번호를 확인해서 포트 번호에 대응하는 소켓에 데이터를 전달
6. TCP 헤더를 제거하고 애플리케이션 데이터를 재구성하여 포켓을 통해 애플리케이션으로 전달