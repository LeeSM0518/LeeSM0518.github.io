# Thread (스레드)

- CPU 이용의 기본 단위인 스레드를 소개한다.
  - 개요 (Overview)
  - 다중 코어 프로그래밍 (Multicore Programming)
  - 다중 스레드 모델 (Multithreading Models)
  - 암묵적 스레딩 (Implicit Threading)
  - 스레드와 관련된 문제들 (Threading Issues)



# 01. Overview (개요)

- **Thread = CPU 이용의 기본 단위**

  - 구성 : 스레드 ID, PC(프로그램 카운터), 레지스터 집합, 스택
  - 같은 프로세스에 속한 다른 스레드의 운영체제 자원들을 공유한다.

  ![img](../capture/스크린샷 2019-04-11 오전 2.57.12.png)



## 동기(1)

- **모든 소프트웨어 응용들은 다중 스레드를 이용한다.**
  - 하나의 응용은 몇 개의 실행 흐름을 가진 독립적인 프로세스로 구현된다.
- Web browser
  - ex) 크롬
- Word processor
  - ex) 한글파일, PDF



## 동기(2)

- **하나의 응용이 여러 개의 비슷한 작업들을 실행할** 필요가 있는 상황이 있다.
- **Web server**
  - 서버는 클라이언트로부터 **요청을 받는다.**
  - 여러 개의 클라이언트들이 **동시에 접근하게 된다면?**
    - 클라이언트의 요청을 받는 **별도의 스레드를 생성**
    - 요청이 들어오면 다른 **프로세스를 생성하지 않고** 요청을 서비스하는 또 다른 스레드를 생성.



## 동기(3)

 다수의 **운영체제 커널은 현재 다중화되어** 있다.

- **Solaris** : 스레드 집합 생성
- **Linux** : 커널 스레드 사용



## 장점

- **응답성 (Responsiveness)**

  : 다중 스레드화하면 응용 프로그램의 일부분이 봉쇄되거나 긴 작업을 실행하더라도 **프로그램의 수행이 계속되는 것을 허용한다.**

  - 사용자에 대한 **응답성 증가** , 사용자 인터페이스 설계에 유용

- **자원 공유 (Resource Sharing)**

  - 스레드는 자신이 속한 프로세스의 **자원과 메모리를 공유한다.**
  - 같은 주소 공간 내에 **여러 개의 다른 작업을 하는 스레드를** 가질 수 있다.

- **경제성 (Economy)**

  - 스레드는 자신이 속한 프로세스의 **자원을 공유**
  - **문맥 교환하는** 것이 보다 더 경제적이다.

- **규모 적응성 (Scalability)**

  - 각각의 스레드가 **다른 처리기에서도** 병렬로 실행될 수 있다. 즉, CPU 개수가 증가해도 적응을 한다.



## 다중 코어 프로그래밍

다중 코어를 더 효율적으로 사용할 수 있고 병행 실행을 더 향상시킬 수 있는 기법을 제공한다.

- 단일 코어 시스템에서의 **병행 실행(concurrent execution)**

  : 하나의 코어는 한번에 하나의 스레드만 실행 가능하기 때문에, **스레드의 실행이 시간에 따라 교대로 실행한다.**

  ![img](../capture/../capture/스크린샷 2019-04-23 오후 9.51.01.png)



- 다중 코어 시스템에서의 **병렬 실행(parallel execution)**

  : 여러 코어를 가진 시스템에서는 개별 스레드를 각 코어에 배정할 수 있기 때문에, 스레드들이 **병렬적으로 실행될 수 있다.**

  ![img](../capture/../capture/스크린샷 2019-04-23 오후 9.54.05.png)



## 프로그래밍 도전과제

다중 코어 시스템을 위한 프로그래밍을 위해 극복해야 할 도전 과제들

- **테스크 식별 (Identifying tasks)**
  - 독립된 병행 가능 테스크로 나눌 수 있는 영역을 찾는 작업이 필요하다.
- **균형 (Balance)**
  - 전체 작업에 균등한 기여도를 가지도록 테스크를 나누는 것이 중요하다.
- **데이터 분리 (Data splitting)**
  - 데이터 또한 개별 코어에서 사용할 수 있도록 나누어져야 한다.
- **데이터 종속성 (Data dependency)**
  - 데이터가 둘 이상의 테스크 사이에 종속성이 없는지 검토되어야 한다.
- **시험 및 디버깅 (Testing and debugging)**



## 병렬 실행의 유형

- **데이터 병렬 실행**
  - 동일한 데이터의 부분 집합을 다수의 코어에 분배한 뒤 각 코어에서 연산을 실행한다.
- **테스크 병렬 실행**
  - 데이터가 아니라 테스크를 다수의 코어에 분배한다.

> 대부분 이 두 전략을 혼용한다.



# 02. Multithreading Models (다중 스레딩 모델)

- **다중 스레딩 모델들**
  - Many-to-One (다 대 일)
  - One-to-One (일 대 일)
  - Many-to-Many (다 대 다)
- **사용자 스레드 (user thread)** : 커널 위에서 지원되며 커널의 지원 없이 관리된다.
- **커널 스레드 (kernel thread)** : 운영체제에서 직접 지원, 관리된다.

> 사용자 스레드와 커널 스레드는 어떤 연관 관계가 존재한다.



## 다-대-일 모델

- **많은 사용자 수준 스레드를 하나의 커널 스레드로 사상한다.**
  - **한번에 하나의** 스레드 만이 커널에 접근할 수 있다.
  - 다중 스레드가 다중 코어 시스템에서 **병렬로 실행될 수 없다.**

![스크린샷 2019-04-23 오후 10.10.22](../capture/스크린샷 2019-04-23 오후 10.10.22.png)



## 일-대-일 모델

- **각 사용자 스레드를 각각 하나의 커널 스레드로 사상한다.**
  - 다-대-일 모델보다 **많은 병렬성을 제공한다.**
  - 사용자 수준 스레드를 생성할 때 그에 따른 커널 스레드를 생성
    - 커널 스레드 생성의 **오버헤드가** 응용 프로그램의 **성능을 저하할** 수 있다.

![스크린샷 2019-04-23 오후 10.14.27](../capture/스크린샷 2019-04-23 오후 10.14.27.png)



## 다-대-다 모델 (1)

- **여러 개의 사용자 스레드를 그보다 작은 수 혹은 같은 수의 커널 스레드로 다중화한다.**
  - 커널의 스레드의 수는 응용 프로그램이나 특정 기계에 따라 결정된다.
  - **필요한 만큼** 많은 사용자 수준 스레드를 **생성** 가능.
  - 커널 스레드가 다중 처리기에서 **병렬로 실행될 수 있다.**

![스크린샷 2019-04-23 오후 10.17.44](../capture/스크린샷 2019-04-23 오후 10.17.44.png)



## 다-대-다 모델 (2)

- 다-대-다 모델의 변형 => **두 수준 모델 (two-level model)**
  - 많은 사용자 스레드를 적거나 같은 수의 커널 스레드로 **다중화를 유지하면서** 하나의 사용자 스레드가 **하나의 커널 스레드에 종속되도록 허용한다.**

![스크린샷 2019-04-23 오후 10.19.59](../capture/스크린샷 2019-04-23 오후 10.19.59.png)



# 03. Implicit Threading (암묵적 스레딩)

- 다중 스레드 응용의 설계를 도와주는 한 가지 방법은 스레딩의 생성과 관리 책임을 응용 개발자로 부터 **컴파일러와 실행시간 라이브러리에게 넘겨주는 것이다.**
- **암묵적 스레딩**
  - 스레드 풀
  - OpenMP
  - Grand Central Dispatch (GCD) 



## 스레드 풀

**스레드 풀 (Thread pools)**

* 프로세스가 시작될 때 **미리 일정한 수의 스레드들을 미리 만들어두고,** 서버가 요청을 받으면 **pool에 있는 하나의 스레드를 할당한다.**
* **스레드 풀의 장점**
  * 기존 스레드로 서비스 해주기 때문에 빠르다. **(생성하는 시간이 불필요하다.)**
  * 존재할 스레드 개수에 제한을 둔다. **(원할한 병렬 처리를 위해)**
  * 태스크를 실행할 때 다양한 전략을 사용할 수 있다.



## OpenMP

**OpenMP** 는 C, C++ 또는 Fortran 으로 작성된 API와 컴파일러 디렉티브의 집합이다.

* 공유 메모리 환경에서 병렬 프로그래밍을 할 수 있도록 도움을 준다.



## Grand Central Dispatch

**GCD** 는 Mac OS X와 iOS 운영체제를 위한 기술로서 C 언어, API 및 실행시간 라이브러리 각각을 확장하여 조합한 기술이다.



# 04. Threading Issues (스레드와 관련된 문제들)

다중 스레드 프로그램에서 고려해야 할 문제들을 논의



## fork 및 exec 시스템 호출

다중 스레드 프로그램에서 **fork와 exec의 의미는 달라질 수 있다.**

* **fork() 시스템 호출**
  * 한 프로그램의 스레드가 fork 를 호출
    * 새로운 프로세스는 **모든 스레드를 복제해야 하는가?**
    * **한 개의 스레드만 가지는** 프로세스이어야 하는가?
* **exec() 시스템 호출**
  * exec()의 매개변수로 지정된 프로그램이 모든 스레드를 포함한 **전체 프로세스를 대체시킨다.**

* **두 버전의 fork() 중 어느 쪽을 택할 것인지?**

  * fork() 를 부르자마자 다시 exec() 를 부른다면

    **: 호출한 스레드만 복사해준다.**

  * fork() 후 exec() 를 호출하지 않는다면

    **: 모든 스레드를 복제한다.**



## 신호 처리 (1)

**Signal** 은 프로세스에게 **어떤 사건이 일어났음을** 알려주기 위한 방법

* **Signal(신호)** 는 특정 사건에 생성되며 프로세스에게 전달되고 반드시 처리가 되어야한다.

* 신호는 둘 중 하나의 처리기에 의해 처리된다.

  * **디폴트 신호 처리기**

    : 커널에 의해 실행

  * **사용자 정의 처리기**

    : 디폴트 처리기를 대체



## 신호 처리 (2)

**Signal(신호)의 처리는 다중 스레드 프로그램에서 복잡하다!!**

* **어느 스레드에게** 신호를 전달해야 하는가?
* **신호 전달 방법은** 신호의 유형에 따라 다르다.
  * **동기식** 신호는 신호를 야기한 스레드에게 전달
  * **비동기식** 신호는 프로세스 내 모든 스레드에게 전달



## 스레드 취소

* 스레드가 끝나기 전에 **강제 종료시키는 작업**

  * 한 스레드가 결과를 찾았다면 나머지 스레드들은 종료되어도 된다.

  * **목표 스레드 (target thread)**

    : 취소되어야 할 스레드

* 목표 스레드의 취소 방식

  * **비동기식 취소 (Asynchronous cancellation)**

    * 한 스레드가 목적 스레드를 **즉시 강제 종료시킨다.**
    * 취소 스레드들에게 할당된 **자원 문제 발생**

  * **지연 취소 (Deferred cancellation)**

    * 목적 스레드가 주기적으로 자신이 강제 **종료해야 하는지를 검사한다.**

    * **취소점 (cancellation point)**

      : 스레드들이 자신이 취소되어도 안전하다고 판단되는 시점에서 **취소 여부를 검사한다.**



## 스레드 로컬 저장소

한 프로세스에 속한 스레드들은 프로세스의 **데이터를 모두 공유한다.**

* 상황에 따라서 각 스레드가 **자기만 접근할 수 있는 데이터를** 가질 필요가 있다

* **스레드 로컬 저장소 (Thread local storage)**
  * 지역변수가 하나의 함수가 호출되는 동안에만 접근할 수 있는 반면에 **TLS는 함수 호출 전후에도 접근할 수 있다.**