# Chapter 12. Media Access Control (MAC)

공유 링크에 대한 링크 접근을 조율하기 위해 **다중 접근 프로토콜(Media Access Control)** 이 필요하다.

> 하나의 링크로 여럿이 데이터를 보낼 때 에러가 발생하는 것을 방지한다.



**Taxonomy of multiple-access protocols** : 다중 접속 프로토콜의 분류

<img src="../capture/스크린샷 2019-06-09 오후 5.57.00.png">

* **Random-access protocols** : 규칙만 있고 순서 없이 링크에 접근
* **Controlled-access protocols** : 순서를 정해서 링크에 접근
* **Channelization protocols** : 링크를 채널화



# 12.1. Random accessr

random-access / contention 방식은 접근 제어를 하지 않는다.

* **전송 매체의 상태에 따라서 전송한다.**
* 각 지국(station)은 **합의된 전송 절차(규칙O, 접근 제어X)** 에 따라 매체에 접근하여 전송한다.
* 각 지국은 **다른 지국의 간섭을 받지 않고 매체 접속의 권리** 를 갖는다.
* 지국이 동시에 접근할 때 **충돌(collision)** 발생



## 12.1.1. ALOHA

* 프레임을 언제든지 전송
* 하나의 채널을 여러 지국이 사용하므로 **프레임간 충돌 발생**
* ACK 가 **타임아웃까지 오지 않으면 재전송**
* 충돌시 데이터를 버려야 한다.

<img src="../capture/스크린샷 2019-06-09 오후 6.30.12.png">

---



**ALOHA protocol 절차**

<img src="../capture/스크린샷 2019-06-09 오후 6.32.34.png">

1. 지국에서 프레임을 만든다. (K: 재전송 시도 횟수)
2. 프레임을 보내고 (2 x 최대 지연 시간) 만큼의 타임 아웃 시간을 잰다.
3. ACK가 도착했는가?
   1. 도착하면 성공
   2. 도착하지 않으면, 재전송 시도 횟수를 올리고 최대 재전송 시도 횟수를 넘었는지 확인
      1. 넘었으면 전송을 중단한다.
      2. 넘지 않았으면 임의 숫자를 만들고 임의 숫자와 최대 지연 시간을 곱한 만큼을 기다린다.

---



**Vulnerable time** : 충돌 위험이 있는 <u>취약 시간</u>

* **지국 B의 취약 시간**

<img src="../capture/스크린샷 2019-06-09 오후 6.42.11.png">

> T(fr) = 하나의 frame 을 전송하는데 걸리는 시간

**Vulnerable time** = T(fr) 의 두 배

---



### Example 12.2.

A pure ALOHA network 는 200-bit frames 을 공유하는 200kbps 채널을 사용한다. 충돌이 생기지 않는 조건은 무엇인가?

 **T(fr) = 200bits / 200kbps X 2 = 2ms**



**취약 시간 회피**

* 지국이 전송하기 **1ms 이전부터** 전송 금지
* 전송이 시작된 후 **1ms 지나기 전** 전송 금지

---



### ALOHA - Slotted ALOHA

정해진 time slot 에 프레임 전송

* 시간을 T(fr) 의 slot으로 나누어 지국은 **매 시간 slot이 시작할 때만** 전송하도록 규제

  > 시간 슬롯이 시작되는 타이밍을 놓치면 다음 시작 시점까지 대기

<img src="../capture/스크린샷 2019-06-09 오후 6.48.26.png">



* **Slotted ALOHA 프로토콜의 취약 시간** : T(fr)
* **pure ALOHA** 의 절반으로 감소

<img src="../capture/스크린샷 2019-06-09 오후 6.49.41.png">

---



## 12.1.2. CSMA

**Carrier sense multiple access (CSMA)** : 캐리어 감지 다중 액세스로서, 채널의 상태를 먼저 확인한다.

* 각 지국이 전송전에 먼저 **매체의 상태를 확인**
* 충돌을 완전히 없앨 수는 없다.

<img src="../capture/스크린샷 2019-06-09 오후 7.46.20.png">

---



**CSMA의 취약 시간** : 전파 시간 (Propagation time) T(p)

<img src="../capture/스크린샷 2019-06-09 오후 7.48.06.png">

> 취약 시간 = 전파 시작 시간 + 지연 시간 = t(1) + T(p)

---



**CSMA-Persistence methods (지속 방식)**

* **채널이 사용 중일 때의 동작 방식**
  * Non-persistent method (비지속적 방식)
  * Persistent method (지속적 방식)
    * 1-persistent method
    * P-persistent method

---



**Nonpersistent strategy (비지속적 전략)**

* 회선이 사용 중이면 **임의의 시간 동안 기다리다가** 다시 회선 감지 (동시에 전송할 확률을 낮춘다)
* 희선이 사용 중이 아니면 **즉시 전송**
* 임의의 시간을 기다리므로, **네트워크의 효율성이 줄어든다.**

<img src="../capture/스크린샷 2019-06-09 오후 8.54.47.png">

---



**1-persistent method**

* 회선이 휴지 상태이면 지국이 즉시 프레임을 전송
* 충돌 가능성 증가. **(기다리는 임의의 시간이 없다)**

<img src="../capture/스크린샷 2019-06-09 오후 8.55.22.png">

---



**p-persistent** : 채널이 최대 전파 지연 시간과 같거나 그보다 큰 시간 슬롯을 사용하는 경우에 채택

1. 확률 p로 프레임 전송

2. 확률 q = 1 - p 로 다음 슬롯 시작까지 기다린 후 다시 회선 감지

3. 만약 회선이 사용 중이면 충돌로 간주하고 대기 절차

4. 회선이 휴지 상태면 1번 단계로 간다.

<img src="../capture/스크린샷 2019-06-09 오후 9.00.46.png">

---



## 12.1.3. CSMA/CD

**Carrier sense multiple access with collision detection (CSMA/CD)** : CSMA 방식에 충돌 처리 절차 추가

* 프레임 전송 후 충돌 감지되면 **재전송**
* 이더넷 프로토콜에 사용

<img src="../capture/스크린샷 2019-06-09 오후 9.04.58.png">

---



<img src="../capture/스크린샷 2019-06-09 오후 9.07.37.png">

* 매우 짧은 프레임은 전송완료 전에 **충돌 감지 안됨.**
* 2 x T(p) 이후에도 전송중이여야 충돌을 감지할 수 있다.
* **최소 프레임 크기**
  * 최소 프레임 전송 시간 T(fr) 은 **최대 전파 시간 T(p) 의 2배** 가 되어야 한다.

---



### Example 12.5

CSMA/CD 를 사용하는 네트워크 bandwidth 가 10Mbps 이다. 최대 전파 시간이 25.6 us 일 때, 프레임의 최소 길이는?

 최소 프레임 전송 시간은 전파 시간의 2배 이상이어야 하므로, **T(fr) = 2 X T(p) = 51.2 us** 이다.

따라서, 충돌 감지를 위해 최소 51.2 us 동안 기다려야 하고 네트워크에서 51.2 us 동안 전송되는 데이터는 

 **10Mbps x 51.2us = 512bits or 64bytes** 이다. 

그러므로, 최소 64byte 이상의 프레임 크기를 가져야 한다.

---



**CSMA/CD 와 ALOHA의 차이**

1. **지속 과정 추가** : 프레임 전송 전 채널 감지
2. **프레임 전송** : 프레임 전송 중 충돌 및 종료 여부 감지
3. **충돌 신호 전송** : 모든 지국에게 충돌 신호 전송

<img src="../capture/스크린샷 2019-06-09 오후 9.16.02.png">

1. 재전송 횟수 = 0
2. 지속성 방식 중 하나를 적용
3. 충돌이 났는가?
   * **예** : 충돌을 감지 했는가?
   * **아니요** : 송수신한다.
4. 충돌을 감지 했는가?
   * **예 :** 충돌이 났음을 보낸다.
   * **아니요 :** 성공
5. 재전송 횟수 증가
6. 재전송 횟수가 15 보다 낮은가?
   * **예 :** 랜던 시간을 기다린다.
   * **아니요 :** 중단한다.

---



## 12.1.4. CSMA/CA

**Carrier sense multiple access with collision avoidance (CSMA/CA)** : 무선 네트워크에 사용된다.

* 회선이 사용되지 않으면 **IFG (Interfame gap)** 시간 만큼 대기
* 다시 **임의의 시간(contention window)을 대기 후,** 프레임을 전송하고 타이머 설정
* 타입 아웃 전에 **확인 응답(Acknowledgement)** 을 받으면 전송 성공

<img src="../capture/스크린샷 2019-06-09 오후 9.24.52.png">

---



**CSMA/CA 의 3 가지 전략**

* **IFS (Interframe Space)**
  * 전송을 **늦추어** 충돌 회피
  * 멀리 떨어진 지국에서 보낸 신호가 도달할 수 있는 **여유를 둔다**
  * 지국이나 프레임의 **우선 순위에도 사용**
* **Contention window (경쟁 구간)**
  * 시간 슬롯으로 나뉘어져 있는 일정 시간
  * 휴지 채널을 발견하지 못하면 두 배 식 슬롯을 늘려 나간다
* **Acknowledgement (응답)**
  * 긍정 응답과 타임 아웃을 사용하여 수신자가 **프레임을 잘 받았다는 것을 보장**

---



**CSMA/CA 흐름도**

<img src="../capture/스크린샷 2019-06-09 오후 9.31.32.png">

1. 채널이 휴지 상태인가?
2. IFS 시간 만큼 쉰다
3. 랜덤 숫자를 생성한다.
4. RTS 를 보낸다
5. 타이머를 설정한다.
6. CTS(RTS에 대한 응답)가 Time out 되었는지 확인
7. IFS 만큼 다시 기다린다.
8. 프레임을 전송한다.
9. 타이머를 설정한다.
10. ACK가 Time out 되었는지 확인 후 성공시 종료하거나 실패시 재전송 횟수를 증가시키고 재전송을 요청한다.

---



### CSMA/CA and NAV

**프레임 교환 타임라인**

<img src="../capture/스크린샷 2019-06-09 오후 9.35.57.png">

> **NAV** 시간 동안 잠시 쉰다.

---



# 12.2. Controlled Access

제어되는 접근

* 다른 지국에 의해 **전송 권리를 인정 받을때만 전송한다.**
  * **Reservation**
  * **Polling**
  * **Token passing**

---



## 12.2.1. Reservation

예약

* **reservation mehotd (예약 방식)** : 각 지국은 데이터를 전송하기 전에 예약한다.
  * 자신의 슬롯에 예약한 지국은 예약 프레임 뒤에 데이터 프레임을 전송

<img src="../capture/스크린샷 2019-06-09 오후 9.44.06.png">



## 12.2.2. Polling

대표 지국을 **primary station(주국),** 나머지를 **secondary station(종국)으로** 지정하는 동작 방법

* 모든 데이터는 **주국을** 거쳐 전송됨
* **Select** : 주국이 데이터를 보낼 종국 선택
* **Poll** : 데이터를 보낼 종국이 있는지 문의

<img src="../capture/스크린샷 2019-06-09 오후 10.10.58.png">



## 12.2.3. Token Passing

Ring 형태의 topology에서 동작하며 **token** 이라는 특별한 프레임이 Ring을 따라 회전한다.

<img src="../capture/스크린샷 2019-06-09 오후 10.12.28.png">

* **Token Passing 종류**

<img src="../capture/스크린샷 2019-06-09 오후 10.13.07.png">



# 12.3. Channelization

**Channelization** : 링크의 가용 대역폭을 시간, 주파수, 코드 등을 통해 **다중 접근 하는 방법**

=> 가역폭 안에서 여러 개의 통신 채널을 만든다.

* **FDMA (Frequency Division Multiple Access)** : 주파수 분할
* **TDMA (Time Division Multiple Access)** : 시분할
* **CDMA (Code Division Multiple Access)** : 코드 분할

---



**FDMA**

<img src="../capture/스크린샷 2019-06-09 오후 10.16.46.png">

---



**TDMA**

<img src="../capture/스크린샷 2019-06-09 오후 10.17.14.png">

---



**CDMA**

* 각 지국에 **chip** 으로 부르는 숫자의 순서로 된 코드가 할당되어 있다.

<img src="../capture/스크린샷 2019-06-09 오후 10.18.13.png">

* **부호화 규칙**
  * Data bit 0 => -1
  * Data bit 1 => +1
  * Slience => 0

---



**Multiplexer**

<img src="../capture/스크린샷 2019-06-09 오후 10.19.39.png">

---



**Demultiplexer**

<img src="../capture/스크린샷 2019-06-09 오후 10.20.07.png">

