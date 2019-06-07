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

**Error Correction (정정)** : 오류 발생 여부와 오류 개수 및 위치를 판단 후 정정

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
* 

