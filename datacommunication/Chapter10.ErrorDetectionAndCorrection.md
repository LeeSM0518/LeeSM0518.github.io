# Chapter 10. Error Detection and Correction (에러 검출과 회복)



# 10.1. Background (배경)

## 10.1.1. Types of Errors (오류의 유형들)

**간섭(interference)** 에 의해 신호 형태가 바뀔 수 있다.

* 한 비트씩 에러
* 두 개 이상의 비트 에러



## 10.1.2. Redundancy (중복)

오류를 검출하거나 정정하기 위해 데이터 외에 **추가 비트를 첨가하는 기법**

* 송신자는 비트를 추가하고 수신자는 추가된 비트를 제거한다.



## 10.1.3. Detection versus Correction (검출 대 회복)

**Error Detection (검출)** : 오류 발생 탐지

* 오류 **개수 및 위치 고려 X**

**Error Correction (정정)** : 오류 발생 여부와 오류 개수 및 위치를 **판단 후 정정**

* 검출에 비해 매우 어렵다.
* 고려사항이 많아 추가 비트가 많이 소요됨.



# 10.2. Block Coding 

정해진 k 개의 bit 로 잘라서 r 개의 비트를 붙여서 오류를 처리

* **k :** 보낼 데이터
* **n = k + r** : 오류를 검출할 수 있는 상태

<img src="../capture/스크린샷 2019-06-07 오후 5.37.56.png">



## 10.2.1. Error Detection (에러 검출)

* 수신자가 **코드워드 리스트를** 가지고 있는데, 수신자가 데이터를 해석했을 때 코드워드가 무효 코드워드 인 것을 **발견하게 되면** 데이터가 손상되었음을 감지한다.



### Example 10.1.

<img src="../capture/스크린샷 2019-06-07 오후 5.44.41.png">

1. 011 데이터를 받은 경우에는 **에러가 없는 것이므로** 01 데이터를 가져온다.
2. 111 데이터를 받은 경우에는 **에러를 검출하게 되고** 데이터가 버려진다.
3. **But,** 두 비트가 에러가 났을 경우에는 에러를 검출하지 못한다.



### Example 10.2.

**Hamming distance** : 서로 다른 비트의 개수

* 전송 중에 **오류가 생긴 비트들의 수**
* 두 워드에 **XOR 연산한 결과의 1의 개수 (서로 다른 비트의 수)**



**아래 두 워드 사이의 해밍 거리는?**

* **d(000, 011)**
  * 000 XOR 011 = 011
  * Hamming distance = 2
* **d(10101, 11110)** 
  * 10101 XOR 11110 = 01011
  * Hamming distance = 3



<img src="../capture/스크린샷 2019-06-07 오후 6.00.25.png">

* 000 XOR 101 = 101
  * Hamming distance = 2
* 101 XOR 011 = 110
  * Hamming distance = 2



## 10.2.2. Minimum Hamming distance (최소 해밍 거리)

모든 가능한 코드 워드 쌍들 사이의 **가장 작은 해밍 거리**

* 코드가 s개의 오류를 검출해 내기 위해서는 **최소 해밍 거리가 s+1 이 되어야 한다.**

<img src="../capture/스크린샷 2019-06-07 오후 6.21.27.png" width="500">

### Example 10.3.

최소 해밍 거리가 4일 때 몇 비트까지 에러 검출을 할 수 있는가?

> **3 bit**



# 10.3. Detection methods (검출 방법들)

* **Detection methods**
  * Parity check (패리티 검사)
  * Cyclic redundancy check (순환 중복 검사)
  * Checksum (합계 검사)



## 10.3.1. Parity check (패리티 검사)

**Simple parity check**

* Parity bit 를 덧붙여서 **1의 개수가 짝수(혹은 홀수)** 가 되도록 함.



### Example 10.7.

1011 이라는 데이터를 10111(짝수 패리티) 를 전송한다.

1. 10111 의 1의 개수는 4개 이고 2로 mod 연산했을 때, 0 이므로 에러가 아니다.

2. 10011 의 1의 개수는 3개 이고 2로 mod 연산했을 때, 1 이므로 에러 검출

3. 00110 은 2개에 에러가 발생했는데도 불구하고, 2로 mod 연산했을 때 **에러를 감지할 수 없다.**

   > 한 비트의 에러만 감지 가능



**Simple parity check 특성**

* 단일 비트 오류를 검출
* 총 개수가 홀수인 burst 오류도 검출 가능
* **짝수 오류를 검출하지 못함**



## 10.3.2. Cyclic Redundancy Check (순환 중복 검사)

* **Cyclic codes (순환 코드)** : 순환되는 성질이 있는 선형 블록 코드(linear block codes)
  * Linear block codes : 코드간 XOR 연산을 하면 다른 코드를 얻을 수 있는 블록코드
* **데이터 오류 감지를 위해 3 개의 bit를 추가로 붙인다.**



**CRC (Cyclic Redundancy Check 순환 중복 검사)**

* 2진 나눗셈 기반

<img src="../capture/스크린샷 2019-06-07 오후 9.43.35.png">

> 무엇으로 나눌지 송수신기가 공유한다.
>
> 수신기에서 나뉜 나머지 값이 0 인지 확인



* **CRC Encoder (CRC 부호화기)**

<img src="../capture/스크린샷 2019-06-07 오후 9.46.48.png">



* **CRC Decodet (CRC 복호화기)**

<img src="../capture/스크린샷 2019-06-07 오후 9.47.47.png">



### Polynomials (다항식)

**비트 패턴을 다항식 (Polynomials)로** 표현하는 방식

<img src="../capture/스크린샷 2019-06-07 오후 9.51.45.png">



* **CRC division using polynomials**

표현 및 계산 절차가 간단하다.

<img src="../capture/스크린샷 2019-06-07 오후 9.53.51.png">



<img src="../capture/스크린샷 2019-06-07 오후 9.56.14.png">

> 이처럼 순환 중복을 다항식으로 검사하게 되면 단일 비트 에러, 더블 비트 에러, 홀수, 짝수 에러 등 **굉장히 많은 에러가 검출 가능하다. 하지만 회복시킬 수는 없다.**



## 10.3.3. Checksum (합계 검출)

이 오류 검출 방식은 메시지의 오류를 검출하는데 사용된다.

<img src="../capture/스크린샷 2019-06-07 오후 10.01.08.png">

* 전송해야할 데이터 뿐만 아니라 전송할 데이터들의 전체 합도 함께 전송한다.
* 수신자는 데이터들을 더한 결과를 전달받은 마지막 값과 비교한다.
* 두 값이 **같으면 오류X, 다르면 오류O**



### Example 10.12.

(7, 11, 12, 0, 6) 을 다 더하면 36인데 36을 2진수로 변환하면 100100 이다. 하지만 이것은 4-bit가 넘어 **overflow가 발생하기 때문에** 100100 을 **4-bit 씩 나눠 10 + 0100 으로 변환하여 0110 = 6** 으로 만든다.

 그 후 보수화를 하면 **0110이 1001 = 9** 가 된다. 그리고 이 보수화 값을 데이터에 덧붙여서 수신측으로 보내게 된다.

수신측에서는 **데이터들의 합계와 보수화 값을 더한다.** 그러면 36 + 9 = 45 이고 45 = 101101 이므로 overflow 가 발생하게 되어 **1101 + 10 = 1111** 이 된다. 마지막으로 이 값을 **보수화한 값이 0000** 이 되면 오류가 없다는 것을 알게 된다.

<img src="../capture/스크린샷 2019-06-08 오전 12.18.33.png">

* **Checksum, 송신자**
  * 합을 1의 보수 취하여 검사합으로 정의
  * 데이터와 검사합을 전송
* **Checksum, 수신자**
  * 합을 1의 보수 취하여 새로운 검사합으로 정의
  * 0이면 오류 없으므로 메시지 수신, 아니면 폐기