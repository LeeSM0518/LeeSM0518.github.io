# Chapter 11. Data Link Control (DLC)

데이터 링크 제어



# 11.1. DLC Services

**Data Link Layer(데이터 링크 계층) 구성**

* **Data link control** : node to node 통신
* **Media access control** : 다중 접근 제어



**The data link control (DLC)** : 인접한 노드 사이의 통신을 위한 절차를 제공한다.



## 11.1.1. Framing

**Faming** : <u>메시지(비트들)를 프레임으로 구성하여</u> 각 프레임이 다른 프레임과 구분되도록 한다.

* 정보 단위에 따라 두 가지 프레임 방식 존재
  * **Character-oriented (Byte-oriented) framing** : 문자 지향
  * **Bit-oriented framing** : 비트 지향



**Flag** : 프레임의 <u>시작과 끝을 알리는</u> 특수한 8 비트 문자

<img src="../capture/스크린샷 2019-06-08 오전 1.14.28.png">

---



**Byte stuffing** : 플래그나 ESC(탈출문자)가 있을 때마다 <u>미리 정해진 비트 패턴을 추가하는</u> 방법.

<img src="../capture/스크린샷 2019-06-08 오전 1.19.06.png">

---



**If) 상위 계층의 데이터에 flag와 같은 비트 패턴이 존재한다면?**

<img src="../capture/스크린샷 2019-06-08 오전 1.24.32.png">

> 위와 같은 상황이면 에러가 발생하기 때문에 **Bit stuffing** 을 통해서 방지한다.

---



**Bit stuffing** : Flag 와 같은 패턴 데이터를 구분

* 다섯 개의 연속적인 1을 만나면 무조건 0을 추가한다.
* 수신측은 연속 5개의 1 뒤에 위치한 0을 제거한다.

<img src="../capture/스크린샷 2019-06-08 오전 1.27.47.png">

---



## 11.1.2. Flow and Error Control

흐름과 에러 제어



**Flow Control (흐름 제어)**

* DLC 의 목적

* 확인 응답을 받기 전에 전송할 수 있는 <u>데이터의 양을 조절하는 것</u>

  : 수신 장치는 데이터를 수신할 수 있는 양이 한정되기 때문에 수신 한계에 도달하기 전에 송신자에게 일시적 송신 중단을 요청한다.



**Error Control (에러 제어)**

* 에러 검출 및 수정 **(재전송)**
* 두 가지 방법
  * 프레임 손상 시, **폐기**
  * 프레임 손상되지 않았으면, **네트워크 계층으로 전달하거나 확인 응답을 송신자에게 전송**



## 11.1.3. Connection

**DLC(데이터 링크 컨트롤)** 은 비연결 지향이나 연결 지향이 될 수 있다.

* **Connectionless (비연결 지향 프로토콜)**

  : 프레임들이 서로 독립적

* **Connection-oriented (연결 지향 프로토콜)**

  : 두 노드 사이에 논리적인 **연결이 설정** , 프레임은 순서대로 보내지고 모든 프레임들이 전달되면 **연결이 종료**



# 11.2. Data link layer Protocols

데이터 링크 계층 프로토콜들

* **Simple Protocol**
* **Stop and Wait Protocol**
* **GO Back N Protocol**
* **Selective Repeat Protocol**



## 11.2.1. Simple Protocol

흐름 제어와 오류 제어를 하지 않는 **단순한 프로토콜**

* 수신자는 프레임을 수신하는 **즉시 처리함**
* 수신자가 수신하는 프레임은 **절대 초과되지 않는다는 과정**

<img src="../capture/스크린샷 2019-06-08 오후 2.06.47.png">

---



**FSM** : 프로토콜의 동작을 조건(Event) 및 수행 할 작업(Action)으로 정의

<img src="../capture/스크린샷 2019-06-08 오후 2.10.42.png">

---



**Simple Protocol 의 FSM**

<img src="../capture/스크린샷 2019-06-08 오후 2.17.26.png">

**송신자**

* 패킷을 네트워크 계층으로 부터 받고, 프레임을 만들어 보낸다.

**수신자**

* 프레임을 받고 네트워크 계층으로 전송하다.



**송신자는 수신자의 수신에 대한 고려없이 프레임을 보낸다.**

<img src="../capture/스크린샷 2019-06-08 오후 2.25.56.png">

---



## 11.2.2. Stop-and-Wait Protocol

이 프로토콜은 **흐름과 에러 제어 모두를 사용한다.**

<img src="../capture/스크린샷 2019-06-08 오후 2.28.17.png">

* **송신자** 는 프레임을 보내고 응답을 기다린다.
* **수신자** 는 프레임을 수신하고 프레임의 에러 상태에 따른 결과를 답장으로 송신자에게 보낸다.

---



**Stop-and-Wait Protocol 의 FSM**

<img src="../capture/스크린샷 2019-06-08 오후 2.30.24.png">

**송신자**

1. 네트워크 계층으로 부터 패킷을 받는다.
2. 프레임을 만들고 복사한 뒤, 수신자에게 보낸다.그 후 타이머를 시작한다.
3. 타이머가 타임 아웃되면 복사본을 다시 보낸다.
4. ACK 라는 응답이 손상됬으면 무시하고 다시 기다린다.
5. 에러가 없는 ACK 를 받았을 때는 복사본을 제거하고 타이머를 정지한다.



**수신자**

1. 손상된 프레임을 받게 되면 프레임을 무시한다.
2. 에러 없는 프레임을 받게 되면 ACK를 보내게 된다.

---



* **잘 전송되었을 때**

<img src="../capture/스크린샷 2019-06-08 오후 2.41.40.png">



* **전송하다가 프레임이 손상되었을 때**

<img src="../capture/스크린샷 2019-06-08 오후 2.42.39.png">



* **ACK가 손실되었을 때**

<img src="../capture/스크린샷 2019-06-08 오후 2.43.51.png">

> 수신측 보낸 ACK가 손상되면 같은 송신측에서 똑같은 프레임을 두번 보내는 오류가 발생한다.

---



### Example 11.4.

ACK가 손실되었을 때, 똑같은 프레임을 다시 보내는 오류를 방지하기 위해 **sequence numbers 와 acknoledgment numbers 추가한다.**

<img src="../capture/스크린샷 2019-06-08 오후 2.53.42.png">

> ACK 1 이 손상되어 송신측에서 타이머가 다되어 다시 Frame 0 을 보내지만, 수신측에서는 Frame 0 을 이미 받은 것을 인지하고 Frame 0 을 버리고 ACK 1을 다시 보낸다.

---



## 11.2.3. Piggybacking

* 이전 예시들은 **단방향 통신만을** 설명한다.
* **양방향 통신을 위해** Piggybacking 을 사용한다.



**Piggybacking** : 데이터를 전송할 때 확인응답을 함께 끼워 보내는 방법

* 데이터를 받고 확인 응답을 보낼 때, 보내야 하는 데이터와 확인 응답을 같이 보낸다.



# 11.3. HDLC

**High-level Data Link Control (HDLC)** : point-to-point 와 멀티 포인트 링크를 통한 통신의 비트 지향 프로토콜이다.

* **기능**
  * **흐름 제어** : 송수신 간에 전송 데이터를 위한 <u>버퍼를 두고 흐름을 제어한다.</u>
  * **에러 제어** : CRC 방식의 <u>손상 에러 체크.</u> 순서 제어에 의한 <u>전송 에러 검출 및 재전송</u>



## 11.3.1. Transfer Modes

전송 모드들

* **NRM (Normal response mode)**
  * 주로 **일 대 다** 경우에 사용
  * **Primary 만 명령을** 보내고 **secondary 는 응답만** 보낸다.

<img src="../capture/스크린샷 2019-06-08 오후 3.23.30.png">



* **ABM (Asynchronous balanced mode)**
  * 주로 **점 대 점** 경우에 사용
  * 어느 한쪽이 **허가 없이 데이터를 전송할** 수 있다.

<img src="../capture/스크린샷 2019-06-08 오후 3.28.42.png">



## 11.3.2. Framing

* **I-Frame** , Information frame

  : 사용자 데이터와 제어 정보 전송

* **S-Frame** , Supervisory frame

  : 흐름 제어와 오류 제어 정보를 전송

* **U -Frame** , Unnumbered frame

  : 링크 관리와 같은 시스템 관리 정보 전송

<img src="../capture/스크린샷 2019-06-08 오후 3.34.56.png">

---



**Frame**

* **Flag field**

  : 프레임의 시작과 끝

* **Address field**

  : 목적지 주소

* **Control field**

  : 흐름 제어 및 오류 제어

* **Information field**

  : 사용자 데이터나 네트워크 관리 정보

* **FCS (Frame Check Sequence field)**

  : HDLS의 오류 검출 필드

---



**I-Frame** : 사용자 데이터 전송

<img src="../capture/스크린샷 2019-06-08 오후 4.02.33.png">

* **제어 필드의 첫 번째 비트가 0**
* **N(S)** : Sequence number 즉, Frame 번호를 뜻한다.
* **P/F 비트**
  * **Poll** : primary => secondary, 1 일 때
  * **Final** : secondary => primary, 0 일 때

* **N(R)** : ACK, 프레임의 순차 번호

---



**S-Frame** : 확인 응답 및 흐름 제어와 오류 제어

<img src="../capture/스크린샷 2019-06-08 오후 4.08.25.png">

* **제어 필드의 처음 2비트가 10**
* Code (2bit)
  * **00 (수신준비, Receive Ready, RR)** : 정상적인 전송에 대한 확인 응답
  * **10 (수신 불가, Receive Not Ready, RNR)** : 지금까지 수신한 데이터에 대한 응답을 보내고, 더 이상의 프레임을 받을 수 없다는 것을 통보
  * **01 (거부, Reject, REJ)** : 에러가 발생한 프레임을 재전송 요청한다.
  * **11 (선택적 거부, Selective Reject, SREJ)** : 전송 에러가 난 프레임 N(R)에 대한 재전송 요청
* **N(R)** : ACK 또는 NAK 의 순차 번호

---



**U-Frame** : 세션 관리와 제어 정보를 교환

* 시스템 관리 정보를 포함
* 세션의 연결 설정/해제 정보 교환

<img src="../capture/스크린샷 2019-06-08 오후 4.49.15.png">



**asynchronous balanced mode (SABM) frame** : 연결 설정

<img src="../capture/스크린샷 2019-06-08 오후 4.53.32.png">

> Control
>
> : 11(U-Frame) + 11 100 (SABM)



**unnumbered acknowledgment (UA) frame** : 연결 응답

<img src="../capture/스크린샷 2019-06-08 오후 4.55.23.png">

> Control
>
> : 11(U-Frame) + 00 110 (UA)



**DISC (disconnect) frame** : 연결 해제

<img src="../capture/스크린샷 2019-06-08 오후 4.57.13.png">

> Control
>
> : 11(U-Frame) + 00 010 (DISC)

---



* **piggybacking 예시**

<img src="../capture/스크린샷 2019-06-08 오후 4.58.47.png">

**Sequence number** : Frame 번호

**Acknowledge number** : 응답 번호

---



# 11.4. PPP (Point-to-Point Protocol)

장치 가의 **점대점 연결을** 위한 프로토콜



