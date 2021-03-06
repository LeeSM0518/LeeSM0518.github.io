# Wireless PAN

## IEEE WPAN - IEEE 802.15.4 (= Zigbee)

* **IEEE 802.15.4 Zigbee 규격의 특징**
  * 낮은 통신속도
  * 낮은 전력소비
  * 저비용

<br>

# Zigbee

## Zigbee 아키텍처 구성 및 기능

| 명칭                  | 약칭                                               | 주요 기능                                                    |
| --------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| 어플리케이션층        | APL<br />(Application)                             | 사용자의 통신 어플리케이션                                   |
| 어플리케이션 지원부층 | APS<br />(Application<br />Supplier<br />Sublayer) | 어플리케이션 각 논리적 통신경로 확립<br />어플리케이션 간 데이터 교환<br />어플리케이션 간의 메시지 착신확인 및<br />재전송 요구 |
| 네트워크층            | NWK<br />(Network)                                 | 네트워크 관리, 라우팅 관리<br />네트워크 내 메시지 전송      |
| 미디어액세스층        | MAC<br />(Medium<br />Access<br />Control)         | 1홉 내에서 메시지 전송<br />1홉 내에서 착신확인 및 재전송 요구 |
| 물리층                | PHY<br />(Physical)                                | 통신에 사용되는 주파수 채널, 변조방식,<br />프레임 구조      |

<br>

## 디바이스 타입

* **WPAN**
  * 근거리로 설치된 복수의 무선 노드로 구성되어 있는 네트워크
  * 1 개의 PAN 내의 노드는
    * 동시에 공통의 통신 채널 사용이 가능하다.
    * 네트워크 주소만으로 상호 인식이 가능하다
    * 공통 네트워크 식별자인 PAN ID를 가진다.

<br>

* **논리 디바이스 타입**

  | IEEE 802.15.4 명칭                   | PAN 코디네이터    | 코디네이터       | 네트워크 디바이스    |
  | ------------------------------------ | ----------------- | ---------------- | -------------------- |
  | ZigBee 명칭                          | ZigBee 코디네이터 | Zigbee 라우터    | ZigBee 엔드 디바이스 |
  | 네트워크 기동 기능                   | 유                | 무               | 무                   |
  | 라우터 기능                          | 유                | 유               | 무                   |
  | 비콘 발행                            | 가능              | 가능             | 불가                 |
  | 관리 범위                            | 전체 노드         | 자기의 자식 노드 | 자기 뿐              |
  | 가능 노드 수                         | 1 노드 뿐         | 복수 노드 가능   | 복수 노드 가능       |
  | 전력 소비                            | 대                | 대               | 소                   |
  | 요구되는 마이크로<br />프로세서 지원 | 대                | 중               | 소                   |

  * **Coordinator** : PAN을 전체적으로 관리하는 노드
  * **Router** : 전송 착신지 주소에 따라 적절한 통신 경로를 선택하는 기능을 가지는 노드
  * **End device** : router 혹은 coordinator와만 통신할 수 있는 제한된 기능을 가진 노드

<br>

* **물리 디바이스 타입**

  * **FFD(Full-function device)**

    : 라우터 기능 포함

  * **RFD(Reduced-function device)**

    : end device로 제한 (라우터 X, 엔드 디바이스만 존재)

  | 항목          | FFD  | RFD    |
  | ------------- | ---- | ------ |
  | 라우터 기능   | 유   | 무     |
  | 코디네이터    | 가능 | 불가능 |
  | 라우터        | 가능 | 불가능 |
  | 엔드 디바이스 | 가능 | 가능   |

<br>

## 프리미티브(Primitive)

: 네트워크의 각 층에서 제공되는 서비스의 인터페이스

<img src="../capture/스크린샷 2019-10-19 오후 5.14.25.png">

* 요구(Request)
* 통지(Indication)
* 응답(Response)
* 확인(Confirm)

<br>

# IEEE 802.15.4 PHY(물리층)

## 물리층 특징 및 개요

### *IEEE 802.15.4 물리층의 기본 기능*

* 통신 채널 주파수 설정
* 송수신 기능의 개시와 정지
* 간섭 전파 유무의 클리어 채널 판정
* 기본 통신 프레임의 송수신
* 통신경로의 링크 품질 측정

<br>

### *주파수 대역과 채널*

* 868/915MHz의 UHF 대역 채널은 2.4GHz에 비해 데이터 전송속도는 낮지만 장점이 있다.
  1. 실장이 쉽다.
  2. 통신거리가 길다.
  3. 간섭이 적다.
* 2.4GHz 대역의 채널 : **11(2405MHz) ~ 26(2480MHz)**
* 무선통신의 거리는 사용 환경에 크게 좌우된다.

<br>

## 물리층 서비스 - 데이터 서비스

| 프리미티브 | 요구 | 확인 | 통지 | 응답 | 설명        |
| ---------- | ---- | ---- | ---- | ---- | ----------- |
| PD-DATA    | O    | O    | O    |      | 데이터 통신 |

<img src="../capture/스크린샷 2019-10-19 오후 5.38.36.png">

* **PD-DATA indication** : <u>프레임 데이터를</u> 수신측 MAC에 전송함과 동시에 수신한 <u>프레임의 링크 품질 지수를</u> 통지한다.
  * **LQI** : 링크 품질 지수

<br>

## 물리층 서비스 - 관리 서비스

* **클리어 채널 판정 : PLME-CCA**

  * 다른 노드의 메시지와 충돌을 회피하기 위해 데이터를 전송하기 전에 채널이 일정시간 이상 계속해서 비어 있는 상태를 확인

    <img src="../capture/스크린샷 2019-10-19 오후 5.43.51.png" alt="image-20191019174402225" style="zoom:50%;" />

<br>

## 물리층 프레임 구조

<img src="../capture/스크린샷 2019-10-19 오후 5.46.21.png">

* **동기 헤더** : Preamble과 SFD(Start of Frame Delimiter)로 구성, 프레임의 시작을 알린다.
  * **SFD** : 동기 신호의 종료와 실제 송신 데이터의 개시를 수신측에 통지
* **길이** : 물리층 데이터 페이로드의 길이

* **프레임 전송에 필요한 최대 시간**
  * ( ( 5 + 1 + 127 ) x 8 ) / 250,000 = 4.256 msec

<br>

# IEEE 802.15.4 MAC

## MAC의 기본 기능

* **IEEE 802.15.4 MAC의 기본 기능**
  * CSMA/CA 방식에서 채널 액세스
  * 신뢰성 있는 데이터 통신 링크
  * 네트워크에 접속과 절단
  * 슈퍼프레임의 동기 신호인 비콘의 발신(FFD만 옵션)

<br>

## 액세스 제어방식

* **마스터/슬레이브 액세스 제어 방식**

  * 특권을 가진 <u>마스터 노드가 네트워크 전체의 액세스권을 제어한다.</u>

  <img src="../capture/스크린샷 2019-10-19 오후 6.05.32.png" alt="image-20191019180535245" style="zoom:80%;" />

  * 관리하기 쉽지만 네트워크 통신대역 <u>자원을 효과적으로 이용할 수 없다.</u>

<br>

* **CSMA 액세스 제어 방식**

  * **CSMA(Carrier Sense Multiple Access)**
  * 송신할 때 우선 전송로의 상태를 확인하고 <u>전송로가 비어 있으면 송신을 시작한다.</u>

  <img src="../capture/스크린샷 2019-10-19 오후 6.07.01.png">

  * 네트워크의 <u>통신대역 자원을 최대한으로 이용할 수 있지만 통신 지연시간을 제어하기 어렵다.</u>

<br>

## 슈퍼 프레임

: IEEE 802.15.4 네트워크의 <u>노드들끼리 상호 협조하여 동작 가능한 구조(슈퍼 프레임과 비콘 기술의 사용)</u>

* 네트워크 상의 노드는 임의의 시간이 아니라 <u>일정한 시간에서만 송수신이 가능하다.</u>
* 슈퍼 프레임을 이용하기 위하여 각 노드는 서로 <u>비콘을 통해</u> 서로 동기화된다.
* 시간을 16개로 나눔

<img src="../capture/스크린샷 2019-10-19 오후 10.04.16.png">

* **비콘** : 슬롯의 시작, 통신이 가능한지 확인
* **CAP** : Contention Access Preiod(경쟁 접속 구간), 8 ~ 15 슬롯, 8개 보장
* **CFP** : Contention Free Period(무 경쟁 구간), 0 ~ 7 슬롯
* **GTS** : Guaranteed Time Slot(보장된 타임 슬롯)

<br>

* **슈퍼 프레임의 주기**

  <img src="../capture/스크린샷 2019-10-19 오후 10.13.03.png">

  * **Active** : 통신하는 상태
  * **Inactive** : 전체가 유휴 상태

<br>

## 데이터 전송 모델

### *비콘 네트워크의 데이터 전송 모델*

* 스타 토폴리지 네트워크에서만 가능

  <img src="https://t1.daumcdn.net/cfile/tistory/2245E8505668EC8904">

* 부모노드는 데이터 교환이 가능하나, 자식 노드들간 직접적인 데이터 교환은 불가능하다.

* CSMA/CA 방식의 송신 개시 시간은 슬롯의 시간에 맞추어야 한다.

<img src="../capture/스크린샷 2019-10-19 오후 10.30.49.png">

<br>

### *비 비콘 네트워크의 데이터 전송 모델*

* <u>상시적으로 동작하는</u> 노드들끼리의 데이터 전송 모델
* <u>슬립 가능한</u> 자식 노드로부터 부모 노드로의 전송 모델

<img src="../capture/스크린샷 2019-10-19 오후 10.34.10.png">

<br>

## MAC 층 서비스 - 데이터 서비스

| 프리미티브 | 요구 | 확인 | 통지 | 응답 | 설명             |
| ---------- | ---- | ---- | ---- | ---- | ---------------- |
| MCPS-DATA  | O    | O    | O    |      | 데이터 설명      |
| MCPS-PURGE | ▵    | ▵    |      |      | 송신 데이터 취소 |

<img src="../capture/스크린샷 2019-10-20 오후 2.08.40.png" alt="image-20191020140859299" style="zoom:80%;" />

<br>

## MAC 층 서비스 - 관리 서비스

| 프리미티브        | 요구 | 확인 | 통지 | 응답 | 설명                   |
| ----------------- | ---- | ---- | ---- | ---- | ---------------------- |
| MLME-ASSOCIATE    | O    | O    | ▵    | ▵    | 네트워크 접속          |
| MLME-DISASSOCIATE | O    | O    | O    |      | 네트워크 접속의 절단   |
| MLME-SCAN         | O    | O    |      |      | 채널 스캔              |
| MLME-GTS          | ▵    | ▵    | ▵    |      | 보증시간 슬롯 GTS 관리 |

<br>

### *네트워크 접속 : MLME-ASSOCIATE*

: 부모 노드와 접속이 필요할 때 MAC의 상위층인 <u>네트워크 층은 MLME-ASSOCIATE.request 로 MAC 층에 요구</u>

<img src="../capture/스크린샷 2019-10-20 오후 2.18.10.png" alt="image-20191020141812505" style="zoom:80%;" />

<br>

### *네트워크 접속 절단 : MLME-DISASSOCIATE*

* Coordinator가 연결된 장치가 PAN(Personal Area Network, 근거리 무선 네트워크로 사용)을 나가기를 원할 때
  * 간접 전송을 통해 disassociation notification 명령을 보냄
* 연결된 장치가 PAN을 나가기를 원할 때
  * Coordinator에게 disassociation notification 명령을 보냄

<img src="../capture/스크린샷 2019-10-20 오후 2.23.44.png" alt="image-20191020142347639" style="zoom:80%;" />

<br>

### *채널 스캔 : MLME-SCAN*

* 모든 장치는 지정된 채널 목록에 대해 <u>passive 또는 orphan 스캔을</u> 수행할 수 있다.

<img src="../capture/스크린샷 2019-10-20 오후 2.26.58.png" alt="image-20191020142701051" style="zoom:80%;" />

| 타입          | FFD  | RFD  | 설명                                                         |
| ------------- | ---- | ---- | ------------------------------------------------------------ |
| 전계강도 스캔 | O    | X    | ED 스캔, 수신경로 링크의 품질을 측정하기 위해<br />채널의 전계강도를 검지한다. |
| 액티브 스캔   | O    | X    | 주변 노드를 탐색하기 위해 비콘 요구 커맨드를 발행한다.       |
| 패시브 스캔   | O    | O    | 주변 노드로부터 비콘을 탐지한다.                             |
| 고립 스캔     | O    | O    | 고립 노드는 코디네이터로부터 재가입 응답<br />메시지를 기다린다. |

* **O** : 필수 기능
* **X** : 지원 안 됨.
* **FFD(Full-function device)** : 라우터 기능 포함 (코디네이터)
* **RFD(Reduced-function device)** : end device로 제한 (라우터 X, 엔드 디바이스만 존재)

<br>

### *보증시칸 슬롯 GTS의 관리 : MLME-GTS*

: 시간보증 슬롯 GTS(Guaranteed Time Slot)의 요구 혹은 사용중인 GTS를 취소하는 경우

<img src="../capture/스크린샷 2019-10-20 오후 2.46.12.png" alt="image-20191020144614707" style="zoom:80%;" />

<img src="../capture/스크린샷 2019-10-20 오후 2.50.06.png" alt="image-20191020145008855" style="zoom:80%;" />

<br>

## MAC 층 서비스 - 보안 서비스

* **MAC 층의 3가지의 보안 모드**
  * **무 보안 모드** : 보안기능X
  * **액세스 컨트롤 리스트 모드** : ACL(Access Control List)에 들어 있는 노드와 통신을 허가한다.
  * **보안 모드** : ACL의 기능과 AES(Advanced Encryption Standard) 암호화 알고리즘 기능이 적용됨.

<br>

## MAC 층 프레임 구조

<img src="../capture/스크린샷 2019-10-20 오후 3.04.23.png" alt="image-20191020152611955" style="zoom:80%;" />

<br>

## MAC 층 프레임 구조 - 비콘 프레임

<img src="../capture/스크린샷 2019-10-20 오후 3.14.21.png" alt="image-20191020153744884" style="zoom:80%;" />

* **비콘 프레임(Beacon frame)**
  * 비콘 네트워크에서 <u>부모 노드가 비콘 프레임을 자식 노드에게 주기적으로 브로드캐스트함으로써</u> 슈퍼 프레임과의 동기 및 펜딩 데이터를 통지한다.
  * 비콘 프레임은 <u>노드가 네트워크에 참가하고자 하는 경우에 액티브 스캔으로</u> 비콘 요구 명령을 주변 노드에 브로드캐스트할 때도 사용된다.

<br>

## MAC 층 프레임 구조 - 데이터 프레임

<img src="../capture/스크린샷 2019-10-20 오후 3.22.46.png" alt="image-20191020154259728" style="zoom:80%;" />

* **데이터 프레임 (Data frame)**
  * 각 층의 데이터 프레임은 상위 층의 데이터를 이용 가능한 유일한 프레임이다.

<br>

## MAC 층 프레임 구조 - ACK 프레임

<img src="../capture/스크린샷 2019-10-20 오후 3.26.09.png" alt="image-20191020154459942" style="zoom:80%;" />

* **단순한 구조로 설계된 이유**
  * RF칩의 하드웨어로 자동적으로 발생하도록 실장이 가능하고 송신에 필요한 시간이 매우 단축된다.
  * 즉, ACK(응답)을 빨리 보내기 위해

<br>

## MAC 층 프레임 구조 - 커맨트 프레임

<img src="../capture/스크린샷 2019-10-20 오후 3.36.59.png" alt="image-20191020154951403" style="zoom:80%;" />

* 접속 요구, 접속 응답, 접속절단의 통지, 데이터 요구, 비콘 요구, GTS 요구 등 으로 커맨드 ID 설정

<br>

## 견고성

: 굳고 단단한 성징.

* LR-WPAN은 데이터 전송의 **견고성(robustness)을 보장하기 위해** 다양한 메커니즘을 사용한다
  * CSMA-CA
  * Frame acknowledgement (프레임 확인)
  * Data verification (데이터 검증)

<br>

### *CSMA-CA 메커니즘*

* 비 비컨 네트워크는 **unslotted CSMA-CA** 채널 액세스 메커니즘 사용
  * Ack 프레임은 CSMA-CA 메커니즘을 사용하지 않고 전송됨.
* 비콘 네트워크는 **slotted CSMA-CA** 채널 액세스 메커니즘을 사용
  * Ack, 비콘 프레임은 CSMA-CA 메커니즘을 사용하지 않고 전송됨.

<br>

**slotted CSMA-CA mechanism**

<img src="../capture/스크린샷 2019-10-20 오후 3.37.42.png" alt="image-20191020155847600" style="zoom:150%;" />

<br>

**Unslotted CSMA**

<img src="../capture/스크린샷 2019-10-20 오후 3.42.56.png" alt="image-20191020160011623" style="zoom:80%;" />

<br>

### *Inter-frame spacing*

<img src="../capture/스크린샷 2019-10-20 오후 3.44.57.png" alt="image-20191020160452514" style="zoom:150%;" />

<br>

### *Frame acknowledgement*

* Ack를 통해 데이터 또는 MAC 명령 프레임의 성공적인 수신 및 유효성 검사

<br>

### *Data verification*

비트 오류를 검출하기 위해 프레임 검사 시퀀스 (FCS) 메커니즘을 사용

* 16비트 ITU-T CRC를 사용

<br>

# Zigbee NWK

직비 네트워크 계층

<br>

## 네트워크층의 기능

* **멀티 홉이** 가능한 메시지 통신
* **네트워크 주소 관리**
* 네트워크의 **참가와 이탈**
* 1홉 범위에서의 **인접 노드 발견**
* **경로** 발견과 관리

<br>

## 네트워크 토폴로지

: 네트워크의 접속 형태

* **스타 토폴로지**
  * 단말 노드들끼리는 직접통신이 불가능하고 중심이 되는 노드를 반드시 개입시켜서 서로 접속할 수 있는 방식
* **P2P 토폴로지**
  * 노드끼리 서로 직접 데이터를 교환할 수 있는 접속 방식
* **메쉬 네트워크**
  * 각 노드가 각각 메시지의 중계 기능을 가지고 P2P에 접속하는 메쉬 형의 네트워크

<br>

## 네트워크 토폴로지 - 스타 토폴로지

<img src="../capture/스크린샷 2019-10-20 오후 3.49.45.png" alt="image-20191020163023886" style="zoom:80%;" />

**장점**

* 구성이 <u>단순하다.</u>
* 단말 노드는 평소에는 동작하지 않아도 된다(무전력의 슬립. 상태, 저비용 RFD).
* 1홉 통신이므로 데이터전송의 최대 지연 시간을 일정 범위로 제어

**단점**

* 통신거리에는 제한이 있다.

<br>

## 네트워크 토폴로지 - 메쉬 토폴로지

<img src="../capture/스크린샷 2019-10-20 오후 3.58.45.png" alt="image-20191020163617918" style="zoom:80%;" />

**장점**

* 멀티 홉 통신
* 통신의 신뢰성 향상, 통신거리의 연장 가능
  * 별도의 통신경로로 우회하여 메시지 도달 가능

**단점**

* 실장 비용이 높고, 무전력 실장이 어렵다.

<br>

## 네트워크 토폴로지 - 클러스터 트리 토폴로지

<img src="../capture/스크린샷 2019-10-20 오후 4.00.09.png" alt="image-20191020163807945" style="zoom:80%;" />

**장점**

* 멀티 홉 지원
* 통신 경로를 확정하여 데이터 전송 시간을 일정범위로 보증
* 관리하기 쉬운 아키텍처(구조)이며 대규모 네트워크도 구성 가능

**단점**

* 통신 경로의 유연성 결여

<br>

## 네트워크 토폴로지 - 토폴로지 선택(비교)

| 토폴로지                  | 스타 | 메쉬          | 클러스터 트리 |
| ------------------------- | ---- | ------------- | ------------- |
| 소비전력                  | 소   | 대            | 중            |
| 실장 가격                 | 저   | 고            | 중            |
| 통신 거리                 | 짧다 | 길다          | 길다          |
| 1네트워크 당 최대 노드 수 | 적다 | 많다          | 많다          |
| 신뢰성                    | 낮다 | 높다          | 낮다          |
| 여분 메시지               | 적다 | 많다          | 중간          |
| 지연 시간                 | 짧다 | 예측이 어렵다 | 예측 가능     |

<br>

### *하이브리드형 토폴로지*

<img src="../capture/스크린샷 2019-10-20 오후 4.04.50.png" alt="image-20191020164431818" style="zoom:80%;" />

* 스타 + 메쉬 + 클러스터 트리 => 최적의 네트워크 구성

<br>

## PAN ID와 주소

* **PAN ID(Personal Area Netwrok ID)** : 16비트
  * 특정 PAN을 식별하기 위해 PAN ID 사용
* **IEEE 802.15.4 주소**
  * <u>IEEE 확장 주소 : 64 비트</u>
    * 전세계의 IEEE 802.15.4에 준하는 RF칩에 유일한 번호 할당
  * <u>단축 주소 : 16비트</u>
    * PAN 내부에서만 사용되는 주소
* **네트워크 주소**
  * Zigbee NWK 층에서 <u>단축 주소의 정의 방법을 정하고</u> 이 주소를 네트워크 주소라 한다.

<br>

## PAN ID와 주소 - 주소 용량

* **주소 용량**

  * 네트워크에 참가하는 <u>자식 노드에 지정 가능한 노드의 최대수</u>

    <img src="../capture/스크린샷 2019-10-20 오후 4.30.22.png" alt="image-20191020165844232" style="zoom:80%;" />

<br>

* **주소 할당**

  <img src="../capture/스크린샷 2019-10-20 오후 4.36.15.png" alt="image-20191020165906208" style="zoom:80%;" />

<br>

<img src="../capture/스크린샷 2019-10-20 오후 4.38.05.png" alt="image-20191020165922767" style="zoom:80%;" />

<br>

## 네트워크 참가

* **Zigbee 네트워크 참가** : 부모 노드로부터 네트워크 주소를 지정 받아서 네트워크의 멤버로서 다른 노드와의 통신을 가능하게 하는 프로세스

<br>

* **네트워크에 참가하는 두 가지 방법**
  * **직접 참가**
    * 참가 가능한 <u>노드의 주소를 사전에 코디네이터 또는 라우터의 EEPROM에 기록한다.</u>
    * 참가한 노드의 주소를 EEPROM에 기록하고 다시 기당할 때에는 <u>참가 신청을 경유하지 않고 EEPROM의 정보에 근거하여 네트워크에 참가</u>
  * **참가 신청**
    * EEPROM 에 접속 상대의 정보가 존재하지 않는 경우에 <u>MAC 층의 접속 요구를 네트워크에 참가하는 방법</u>

<br>

<img src="../capture/스크린샷 2019-10-20 오후 4.44.29.png" alt="image-20191021003106987" style="zoom:80%;" />

<br>

## 라우팅

* **Zigbee의 라우팅**

  * 적절한 데이터 전송 경로의 선택
  * 경로의 발견
  * 라우팅의 유지보수

* **트리 경로와 P2P 경로**

  <img src="../capture/스크린샷 2019-10-20 오후 4.58.40.png" alt="image-20191021003235949" style="zoom:80%;" />

  * 트리 경로
    * 라우팅 테이블에 의존하지 않고, 네트워크 주소만으로 라우팅이 가능한 방법

<br>

## 라우팅 - Zigbee 경로 정보

* **경로 비용 (Route Cost)**
  * 가능한 한 <u>적은 홉 수와</u> 동시에 <u>링크 품질이 높은 경로로</u> 데이터를 전송해야 한다.
  * **링크 비용 (Link Cost)**
    * 1홉으로 링크된 2개의 노드 간의 비용

<br>

### *인접 테이블 (Neighbor Table)*

네트워크의 참가, 라우팅, 브로드캐스트 등을 수행하는 경우에 <u>1홉 범위에 있는 주변 노드 정보를 파악하기 위하여</u> 매우 중요한 역할 담당

<br>

### *라우팅 능력*

* **라우팅 테이블** : 최종 목적지로 가는 <u>다음 홉만을 기술</u>
  * Destination : 최종 목적 네트워크 주소
  * Status : 라우팅 상태
  * Next-hop address : 다음 홉의 직접 송신 목적지 주소
* **경로 발견 테이블** : 경로를 발견할 때 사용
  * 경로 발견 프로세스가 완료되면 <u>메모리로부터 자동적으로 제거된다.</u>

<br>

## 라우팅 - AODV 라우팅 알고리즘

### *메쉬 네트워크의 라우팅 기술*

가능한 한 적은 홉 수의 최적 경로를 자동적으로 탐색

<br>

### *Zigbee 규격에서의 라우팅 알고리즘*

* 경로 정보에 기초하여 메쉬 네트워크의 **AODV 라우팅 알고리즘 채용**
* Zigbee 사양에서는 기존의 AODV 알고리즘을 간소화하여 **메쉬형 무선 센서 네트워크의** 라우팅 알고리즘으로 채용하고 있다.

<br>

### *AODV(Ad hoc On-demand Distance Vector) 라우팅 알고리즘*

* **온디멘드형의 라우팅 방식**
  * 송신 경로 탐색이 필요할 때에 경로 발견을 수행하지만, 경로가 확정되면 데이터 송신 이외에 라우팅을 확인하기 위한 메시지는 네트워크상에 흐르지 않는다.
* **AODV에서 송신원은 다음 홉의 송신 목적지 정보만 갖는다.**
  * 라우팅 정보는 경로를 경유하는 모든 노드에 분산되어 보존된다.

<br>

### *알고리즘 동작 원리*

* 경로 발견 요구노드로부터 <u>경로비용 정보가 포함되어 있는 RREQ의 경로 요구 메시지가 네트워크에 브로드캐스트된다.</u>
* 중간 노드는 수신된 RREQ 메시지의 <u>경로비용에 앞 홉의 링크 비용을 가산하여 새로운 RREQ 메시지를 브로드캐스트로 릴레이한다.</u>
* 가장 낮은 비용의 RREQ 메시지가 최종 목적노드에 도착하면 <u>RREQ 응답 메시지를 유니캐스트로 답신하고</u> 경로 발견 요구를 송신한 노드까지 거슬러 올라가 발견된 경로를 확립한다.

<br>

![image-20191028110635818](../capture/image-20191028110635818.png)

![image-20191028110649983](../capture/image-20191028110649983.png)

![image-20191028110717385](../capture/image-20191028110717385.png)

<br>

### *경로 발견 응답: RREP*

* 발견 요구 시퀀스 번호
* 라우팅 발견을 요구하는 노드의 주소
* 경로 발견을 응답하는 노드의 주소
* 잔류 비용

<br>

![image-20191028111717652](../capture/image-20191028111717652.png)

![image-20191028111729860](../capture/image-20191028111729860.png)

![image-20191028111740498](../capture/image-20191028111740498.png)

![image-20191028111751644](../capture/image-20191028111751644.png)

<br>

## 브로드캐스트

: 모든 노드에 동시에 데이터를 전송하는 수단

* 목적 주소를 **"0xFFFF"** 로 하고 주소모드를 **브로드캐스트 모드(0x02)**로 설정하면 브로드캐스트를 할 수 있다.
* 브로드캐스트 반경(메시지 전송이 가능한 최대 홉의 수)을 설정할 수 있다.

![image-20191028112208600](../capture/image-20191028112208600.png)

