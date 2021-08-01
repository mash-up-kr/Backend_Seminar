# OSI 7계층

## OSI 7계층이란?

> Open Systems Interconnecrtoin 의 약자로서, ISO(국제표준화기구)에서 컴퓨터 네트워크 프로토콜 디자인과 통신을 계층으로 나눠 설명한 모델

> 네트워크 구성 요소를 표준화 → 시스템 간의 상호연결성 부여

## 7계층 : 응용 계층(Application Layer)

- 실제로 사용자가 사용하는 UI 및 I/O 작업
- 응용 프로세스 간의 정보 교환, 전자 메일, 파일 전송 등의 서비스를 제공

- 데이터 단위 : Message
- 프로토콜
    - **HTTP**
    - **SMTP**
    - **FTP**

## 6계층 : 표현 계층(Presentatoin Layer)

- 인코딩, 암호화

- 데이터 단위 : Message

## 5계층 : 세션 계층(Session Layer)

- 통신을 관리
    - duplex, half-duplex, full dupelx
    - 통신을 하기 위한 세션을 확립/유지/중단 (운영체제가 해줌)

- 데이터 단위 : Message
- 프로토콜
    - SSL, TLS

## 4계층 : 전송 계층(Transport Layer)

- 유저가 데이터를 주고 받을 수 있도록 함
- 데이터의 신뢰성 검증(시퀀스 넘버 기반)
    - 오류검출 및 복구
    - 흐름제어
    - 중복검사

- 데이터 단위 : Segment/Datagram
- 프로토콜
    - TCP, UDP, **QUIC**

## 3계층 : 네트워크 계층(Network Layer)

- 라우팅
    - 여러개의 노드를 거칠때마다 IP를 사용하여경로를 찾아주는 역할
- 포워딩
    - 다음 라우터로 넘김
- 흐름 제어, 세그멘테이션(segmentation/desegmentation), 오류 제어, 인터네트워킹

- 데이터 단위 : Packet
- 프로토콜
    - IP, ICMP, ARP, RARP...

## 2계층 : 데이터 링크 계층(Data Link Layer)

- 하드웨어와 소프트웨어의 연결점
- 데이터 단위 : frame
- 직접 연결된 두 point 간의 데이터 전송 담당

## 1계층 : 물리 계층(Physical Layer)

- 100110 과 같은 이진 데이터 송수신 (Digital ↔ Analog)
- 데이터 단위 : bit

---

# TCP/IP 4계층

## TCP/IP 4계층이란?

- 데이터

## 4계층 : 응용 계층(Application Layer)

- 사용자와 가장 가까운 계층
- 데이터를 교환하기 위해 사용되는 프로토콜들 존재

- 데이터 단위 : Data
- 프로토콜

    DNS, TLS/SSH, HTTP, SMTP..

## 3계층 : 전송 계층(Transport Layer)

- 연결 제어 및 데이터 송수신
- 세그먼트 : 데이터 전송을 위해 데이터를 일정 크기로 나눈 것

- 데이터 단위 : Segment
- 전송 주소 : Port
- 프로토콜

    TCP, UDP,QUIC...

## 2계층 : 인터넷 계층(Internet Layer)

- 정확한 연결 제공 및 라우팅

- 데이터 단위 : Packet
- 전송 주소 : IP
- 프로토콜

    IP, ARP, ICMP...

## 1계층 : 네트워크 연결 계층(Network Access Layer)

- 데이터가 네트워크를 통해 어떻게 전송되는지를 물리적으로 정의

    → 논리주소(IP)가 아닌 물리 주소 (MAC) 사용

- 데이터 단위 : 프레임
- 전송 주소 : MAC
- 프로토콜

    이더넷, Wi-Fi...

---

# 비교

![OSI%207%20Layer%205cd7c5ae04a247c0bedb3e39dd698f5d/Untitled.png](OSI%207%20Layer%205cd7c5ae04a247c0bedb3e39dd698f5d/Untitled.png)

### 공통점

- 계층 구조이다.

[차이점](https://www.notion.so/7b22691814d84d2ab627e1d84373e5c3)

---

# 어떻게 쓰이고 있을까?

**검색창에 `www.google.com`을 입력했을 때 생기는 일.** ~~근데 이제 네트워크를 곁들인. *구글이 뜬다.*~~

1. URL 검사
    1. URL 파싱
    2. HSTS 리스트 확인
2. 나의 IP 찾기
    1. 같은 서브넷에 DHCP 서버를 찾는 `DHCP Discover` 메세지 Broadcast
    2. DHCP가 응답 → Client의 IP, 가까운 Router IP, 가까운 DNS 서버 IP
    3. IP 충돌 여부 확인
        - Client : 할당받은 IP에 ARP Request 전송 ⇒ 응답 없으면 할당
        - Server : 할당하려는 IP에 ICMP Echo Request ⇒ 응답 없으면 할당
3. `www.google.com` 에 해당하는 IP 찾기
    1. 브라우저 캐시 참조
    2. 로컬 캐시 참조(hosts파일)
    3. 위의 두 경우 모두 없으면 OS에게 DNS서버에 요청 지시
4. 서버와 연결
    1. `socket`  시스템 라이브러리 호출 → 네트워크 계층을 타고 내려가며 헤더들이 덧씌워짐
    2. 3-way handshake
        - HTTPS면 TLS(Transport Layer Security) handshake 과정으로 세션키 획득
5. 서버와 통신
    - HTTP Request
    - 세션이 유지되는 동안 서버에게 요청&응답
6. 브라우저 렌더링
7. 연결종료
    - 서버와 세션이 종료되면 4-way handshake로 연결 종료

![OSI%207%20Layer%205cd7c5ae04a247c0bedb3e39dd698f5d/Untitled%201.png](OSI%207%20Layer%205cd7c5ae04a247c0bedb3e39dd698f5d/Untitled%201.png)

![OSI%207%20Layer%205cd7c5ae04a247c0bedb3e39dd698f5d/Untitled%202.png](OSI%207%20Layer%205cd7c5ae04a247c0bedb3e39dd698f5d/Untitled%202.png)

---

**Reference**

[https://ko.wikipedia.org/wiki/OSI_모형](https://ko.wikipedia.org/wiki/OSI_%EB%AA%A8%ED%98%95)

[https://ko.wikipedia.org/wiki/인터넷_프로토콜_스위트](https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%84%B0%EB%84%B7_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C_%EC%8A%A4%EC%9C%84%ED%8A%B8)

[https://reakwon.tistory.com/59](https://reakwon.tistory.com/59)

[https://donologue.tistory.com/380](https://donologue.tistory.com/380)

[https://owlgwang.tistory.com/1](https://owlgwang.tistory.com/1)

[https://goitgo.tistory.com/25](https://goitgo.tistory.com/25)

[https://velog.io/@jehjong/개발자-인터뷰-TCPIP-4계층](https://velog.io/@jehjong/%EA%B0%9C%EB%B0%9C%EC%9E%90-%EC%9D%B8%ED%84%B0%EB%B7%B0-TCPIP-4%EA%B3%84%EC%B8%B5)

[https://ko.gadget-info.com/difference-between-tcp-ip](https://ko.gadget-info.com/difference-between-tcp-ip)

[https://m.blog.naver.com/tkdldjs35/221977544533](https://m.blog.naver.com/tkdldjs35/221977544533)

[https://twitter.com/kamranahmedse/status/1297131414190776320/photo/1](https://twitter.com/kamranahmedse/status/1297131414190776320/photo/1)

[http://tcpschool.com/webbasic/works](http://tcpschool.com/webbasic/works)
