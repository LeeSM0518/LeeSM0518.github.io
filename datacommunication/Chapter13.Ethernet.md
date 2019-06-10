# Chapter 13. Ethernet

Wired Lan (Local Area Networks) : 유선 랜 (로컬 영역 네트워크), 근거리 네트워크



# 13.1. Ethernet Protocol

이더넷 프로토콜



## 13.1.1. IEEE Project 802

* 다양한 제조사 장치간 **상호 연결 가능한 표준**
* LAN 프로토콜의 **물리층과 데이터 링크층을 명세화**
* **LLC** : 데이터 링크 계층의 흐름 제어, 오류 제어 및 프레임 생성 기능, 서로 다른 LAN 연결성 제공

<img src="../capture/스크린샷 2019-06-09 오후 10.29.18.png">

---



## 13.1.2. Ethernet Evolution

**이더넷 발전**

<img src="../capture/스크린샷 2019-06-09 오후 11.24.05.png">

---



# 13.2. Standard ethernet

표준 이더넷

* **10Mbps** 속도
* 비연결, 비신뢰성 서비스 제공
* 접속 방법 : **1-지속성(매체가 이용 중 인지 계속 검사, 즉각 반응), 프레임 최소 길이 제한(64bytes)**

<img src="../capture/스크린샷 2019-06-09 오후 11.27.18.png">



## 13.2.1. Frame format

* **Preamble** : 프레임 도착 알림 및 동기화 제공
* **SFD (Start frame delimiter)** : 프레임의 시작을 알림 (10101011). 이 다음 필드(DA)가 목적지 주소임을 알린다.
* **DA (Destination address)** : 패킷을 수신하는 지국들의 물리 주소
* **SA (Source address)** : 패킷 송신자의 물리 주소

* **Type** : 패킷의 종류를 나타냄 (IP, ARP, OSPF)
* **Data** : 최소 46 ~ 최대 1500
* **CRC** : CRC-32 로 표준화된 오류 검출 정보



**Frame length**

* CSMA/CD 충돌을 감지할 수 있는 최소 길이 : **64 바이트**
* 최대 길이 1518 (헤더 18 + data 1500) 바이트

---



## 13.2.2. Addressing

* 각 지국은 자신의 **NIC (Network Interface card)** 를 갖고 있으며, 6 바이트의 **물리적 주소** 를 제공

<img src="../capture/스크린샷 2019-06-10 오전 1.11.42.png">

* **Unicast, Multicast, and Broadcast address** : 전송 방식

---



**표준 이더넷 전송은 항상 flooding (브로드 캐스팅)**

<img src="../capture/스크린샷 2019-06-10 오전 1.13.19.png">

---



**매체 접근 방법**

* **CSMA/CD 사용 (1-Persistency)**
  * 지국 A는 매체가 사용 중인지 확인
  * 신호 에너지가 없으면 매체가 사용 중 이지 않기 때문에 <u>자신의 프레임을 전송</u>
  * A는 512bit (64byte) 를 전송하고 <u>충돌이 감지되지 않으면</u> 잘 전송된 것으로 확신.

---



## 13.2.3. Implementations

<img src="../capture/스크린샷 2019-06-10 오전 1.23.34.png">

* **10** : 10Mbps, 속도
* **Base** : Baseband(digital), 디지털 전송 네트워크
* **5** : 500m, 전송 가능 거리 (T: TP 케이블, F: fiber(광) 케이블)

---



**10Base5** : Think Ethernet (두꺼운 이더넷)

* 잘 굽혀지지 않는다.
* **Bus topology,** 탭을 이용 동축 케이블 연결
* **Transceiver** 는 송수신 및 충돌 감지 수행

<img src="../capture/스크린샷 2019-06-10 오전 1.30.27.png">

---



**10Base2** : Thin Ethernet (얇은 이더넷)

* **Bus topology**
* 비용면에서 효율적, 설치가 쉽다.
* 신호 감쇠가 높다. (최대 전송 거리 185m)

<img src="../capture/스크린샷 2019-06-10 오전 1.32.48.png">

---



**10Base-T** : Twisted Pair Ethernet (2쌍, 4가닥)

* **Physical start topology**
* 지국들을 허브에 연결
* 허브 내부에서 충돌이 발생한다.
* 최대 길이 100m

<img src="../capture/스크린샷 2019-06-10 오전 1.34.49.png">

---



**10Base-FL** : Fiber Ethernet

* 2 개의 광섬유를 통해 지국들을 허브에 연결하는 **Star topology**

<img src="../capture/스크린샷 2019-06-10 오전 1.36.03.png">

---



## 13.2.4. Encoding in Standard Ethernet

**표준 이더넷 부호화**

10Mbps의 디지털 신호를 **맨체스터 기법** 으로 디지털 변환하여 전송.

<img src="../capture/스크린샷 2019-06-10 오전 1.37.52.png">

---



## 13.2.5. Change in the Standard

**Bridged Ethernet (브릿지형 이더넷)**

* 브릿지에 의한 <u>LAN의 분할</u>
* **대역폭 증가 (Raise the bandwidth)** : 브릿지를 통해 독립된 링크로 동작

* **충돌 영역의 분리 (Separate collision domain)** : 브릿지로 충돌 영역을 줄이면, 충돌 확률이 줄어든다.

<img src="../capture/스크린샷 2019-06-10 오전 1.47.16.png">

---



## 13.2.6. Changes in the Standard

**Switched Ethernet (교환형 이더넷)**

* 대역폭이 지국과 스위치 사이에만 공유되며 N개의 포트 스위치는 **충돌 영역이 N개의 영역으로 나누어짐**

<img src="../capture/스크린샷 2019-06-10 오후 5.01.22.png">

---



**Full-Duplex Ethernet (전이중 양방향 이더넷)**

* 지국과 스위치 사이에는 송신과 수신을 위한 **2개의 링크 사용**

<img src="../capture/스크린샷 2019-06-10 오후 5.05.20.png">

* 두 개의 독립된 링크로 송수신하므로 **충돌이 발생하지 않아 CSMA/CD 불필요**
* 지국과 스위치간 **점대점으로** 연결된 전용선

---



# 13.3. Fast Ethernet

**목적**

* 100Mbps
* 표준 이더넷과 호환성 유지
* 동일한 48비트 주소 체계 유지
* 동일한 프레임 형식 유지
* 동일한 **최소 / 최대 프레임 길이 유지**

---



**MAC Sublayer**

