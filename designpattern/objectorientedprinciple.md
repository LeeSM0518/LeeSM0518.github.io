# 2. 객체지향 원리

# 2.1. 추상화

**추상화란** 어떤 영역에서 필요로 하는 속성이나 행동을 추출하는 작업을 의미한다.

* 추상화가 가능한 개체들은 개체가 소유한 특성의 이름으로 하나의 **집합(class)을** 이룬다.



* **예시) 자동차 종류마다 엔진 오일을 교환하는 방식**

  * 자동차 종류에 따른 swith문

    ```java
    switch(자동차 종류)
      case 아우디:			// 아우디 엔진 오일을 교환하는 과정을 기술
    	case 벤츠:			 // 벤츠 엔진 오일을 교환하는 과정을 기술
    	case BMW:				// BMW 엔진 오일을 교환하는 과정을 기술
    end switch
    ```

  * 자동차 대신 이들의 추상화 개념인 자동차를 이용한 코드

    ```java
    void changeEngineOil(Car c) {
      c.changeEngineOil();
    }
    ```

    > 인자의 아우디, 벤츠, BMW의 추상화 개념인 Car를 사용했기 때문에, 이 코드는 어떤 새로운 자동차가 추가되더라도 변경할 필요가 없다. 



# 2.2. 캡슐화

소프트웨어 공학에서 요구사항 변경에 대처하는 고전적인 설계 원리로는 **응집도(cohesion) 와 결합도(coupling)가** 있다.

* **응집도** : 클래스나 모듈안의 요소들이 얼마나 밀접하게 관련되어 있는지를 나타낸다.
* **결합도** : 어떤 기능을 실행하는 데 다른 클래스나 모듈들에 얼마나 의존적인지를 나타낸다.

> **높은 응집도와 낮은 결합도를** 유지할 수 있도록 설계해야 요구사항을 변경할 때 유연하게 대처할 수 있다.



**캡슐화는** 정보 은닉(information hiding)을 통해 높은 응집도와 낮은 결합도를 갖도록 한다.

* **정보 은닉** : 알 필요가 없는 정보는 외부에서 접근하지 못하도록 제한하는 것이다.



* **예시) 배열 스택**

  ```java
  package object_oriented_principle;
  
  public class ArrayStack {
  
    public int top;
    public int[] itemArray;
    public int stackSize;
  
    public ArrayStack(int stackSize) {
      itemArray = new int[stackSize];
      top = -1;
      this.stackSize = stackSize;
    }
  
    public boolean isEmpty() {
      return (top == -1);
    }
  
    public boolean isFull() {
      return (top == this.stackSize - 1);
    }
  
    public void push(int item) {
      if (isFull()) {
        System.out.println("Inserting fail! Array Stack is full!");
      } else {
        itemArray[++top] = item;
        System.out.println("Inserted Item : " + item);
      }
    }
  
    public int pop() {
      if (isEmpty()) {
        System.out.println("Deleting fail! Array Stack is empty!");
        return -1;
      } else {
        return itemArray[top--];
      }
    }
  
    public int peek() {
      if (isEmpty()) {
        System.out.println("Peeking fail! Array Stack is empty");
        return -1;
      } else {
        return itemArray[top];
      }
    }
  
  }
  ```

  > 이 코드의 주의할 점은 자료구조에 모두 **public 키워드를** 붙여 외부에 공개되어 있다는 점이다. 즉, push 메서드나 pop 메서드를 사용하지 않고 직접 배열에 값을 저장하거나 수정할 수 있다. 그래서 이 코드는 **정보 은닉을 하지 못 하고 있는 코드이다.**

* **정보 은닉 코드**

  자료구조의 형태와 관련이 있는 것들은 **private 키워드를** 붙여 은닉한다.

  ```java
  private int top;
  private int[] itemArray;
  private int top;
  ```

* **main 메소드**

  정보 은닉한 ArrayStack 클래스를 사용하는 방법

  ```java
  public class StackClient {
    public static void main(String[] args) {
      ArrayListStack st = new ArrayListStack(10);
      st.push(20);
      System.out.println(st.peek());
    }
  }
  ```



# 2.3. 일반화 관계

## 2.3.1. 일반화는 또 다른 캡슐화

일반화 관계는 객체지향 프로그래밍 관점에서는 상속 관계라 한다. 따라서 속성이나 기능의 재사용만 강조해서 사용하는 경우가 많다. **이는 일반화 관계를 극히 한정되게 바라보는 시각이다.** 

**일반화(generalization)는** "여러 개체들이 가진 공통된 특성을 부각시켜 하나의 개념이나 법칙으로 성립시키는 과정"이라 한다.

* **예시) 과일**

  : 사과, 배, 바나, 오렌지 등은 과일의 한 종류이므로 과일을 **특수화(specialization)한** 개념이다.

  <img src="../../capture/스크린샷 2019-07-29 오후 8.21.22.png" width=500>



* **예시) 장바구니에 있는 과일 가격의 총합 구하는 코드**

  * **switch 활용**

    ```java
    가격 총합 = 0;
    while(장바구니에 과일이 있다) {
      switch(과일 종류) {
        case 사과:
          가격 총합 = 가격 총합 + 사과 가격;
        case 배:
          가격 총합 = 가격 총합 + 배 가격;
        case 바나나:
          가격 총합 = 가격 총합 + 바나나 가격;
        case 오렌지:
          가격 총합 = 가격 총합 + 오렌지 가격
      }
    }
    ```

    > 위의 코드는 새로운 과일의 종류가 나타날 때마다 항상 코드를 수정해야 하므로 변경사항에 **유연성 있게 대처하지 못한다.**

  * **다형성 활용**

    ```java
    int computeTotalPrice(LinkedList<Fruit> f) {
      int total = 0;
      Iterator<Fruit> itr = f.iterator();
      
      while(itr.hasNext()) {
        Fruit curFruit = itr.next();
        // calculatePrice 메소드는 실제 과일 객체의 종류에 따라 다르게 실행된다.
        total = total + curFruit.calculatePrice();
      }
      return total;
    }
    ```

    > 일반화 관계는 외부 세계에 자식 클래스를 캡슐화하는 개념으로 볼 수 있으며, 이때 캡슐화 개념은 한 클래스 안에 있는 속성 및 연산들의 캡슐화에 한정되지 않고 **일반화 관계를 통해 클래스 자체를 캡슐화하는 것으로 확장된다.**



* **예시) 일반화는 또 다른 캡슐화**

  사람이 자동차를 운전하는 상황

  <img src="../../capture/스크린샷 2019-07-29 오후 8.35.43.png">



## 2.3.2. 일반화 관계와 위임

많은 사람이 일반화 관계를 속성이나 기능의 상속, 즉 재사용을 위해 존재한다고 오해하고 있다.

* **ArrayList를 이용한 Stack의 구현**

<img src="../../capture/스크린샷 2019-07-31 오전 12.27.30.png" width=200>

> 이 예시는 기능의 재사용이라는 측면으로만 보면 성공적이지만, ArrayList 클래스에 정의된 **스택과 전혀 관련 없는 수많은 연산이나 속성도 같이 상속받게 된다.** 이렇게 되면 스택의 무결성 조건인 'LIFO' 에 위배된다.



* **'is a kind of 관계'** : 기본적으로 일반화 관계는 'is kind of 관계' 가 성립되어야 한다.
  * **예시)** Stack "is a kind of" ArrayList. 배열 목록 대신 스택을 사용할 수 없으므로, 이 예시는 거짓이다.



어떤 클래스의 일부 기능만 재사용하고 싶은 경우에는 **위임(delegation)을** 사용한다.

* **위임(delegation)** : 자신이 직접 기능을 실행하지 않고 다른 클래스의 객체가 기능을 실행하도록 위임하는 것이다.



* **위임을 사용해 일반화를 대신하는 과정**

  1. 자식 클래스에 부모 클래스의 인스턴스를 참조하는 속성을 만든다. 이 속성 필드를 this로 초기화한다.
  2. 서브 클래스에 정의된 각 메서드에 1번에서 만든 위임 속성 필드를 참조하도록 변경한다.
  3. 서브 클래스에서 일반화 관계 선언을 제거하고 위임 속성 필드에 슈퍼 클래스의 객체를 생성해 대입한다.
  4. 서브 클래스에서 사용된 슈퍼 클래스의 메서드에도 위임 메서드를 추가한다.
  5. 컴파일하고 잘 동작하는지 확인.

  

  * **예제 코드) ArrayList를 위임해 Stack 클래스 구현**

  1. ArrayList 클래스의 인스턴스를 참조하는 속성인 asList 객체를 만든 후, 이 속성 필드를 this로 초기화

  ```java
  public class MyStack<String> extends ArrayList<String> {
    
    private ArrayList<String> arList = this;
    
    public void push(String element) {
      add(element);
    }
    
    public String pop() {
      return remove(size() - 1);
    }
    
  }
  ```

  2. push와 pop 메서드에 asList 객체를 참조하도록 변경한다.

  ```java
  package object_oriented_principle;
  
  import java.util.ArrayList;
  
  public class MyStack<String> extends ArrayList<String> {
  
    private ArrayList<String> arList = this;
  
    // 2. push 와 pop 메서드에 asList 객체를 참조하도록 변경한다.
    public void push(String element) {
      arList.add(element);
    }
  
    public String pop() {
      return arList.remove(size() - 1);
    }
  
  }
  ```

  3. ArrayList와 MyStack 클래스 사이의 일반화 관계를 제거하고 arList를 ArrayList 객체로 생성해 초기화한다.

  ```java
  package object_oriented_principle;
  
  import java.util.ArrayList;
  
  public class MyStack<String> extends ArrayList<String> {
  
    private ArrayList<String> arList = new ArrayList<>();
  
    public void push(String element) {
      arList.add(element);
    }
  
    public String pop() {
  		return arList.remove(arList.size() - 1);
    }
  
  }
  ```

  4. arList 객체의 isEmpty와 size 메서드에 위임 메서드를 서브 클래스에 추가한다.

  ```java
  package object_oriented_principle;
  
  import java.util.ArrayList;
  
  public class MyStack<String>{
  
    private ArrayList<String> arList = new ArrayList<>();
  
    public void push(String element) {
      arList.add(element);
    }
  
    public String pop() {
      return arList.remove(arList.size() - 1);
    }
  
    public boolean isEmpty() {
      return arList.isEmpty();
    }
    
    public int size() {
      return arList.size();
    }
  
  }
  ```



* **Vector 클래스를 사용(위임)해 Stack 클래스 구현**

  ```java
  package object_oriented_principle;
  
  import java.util.Vector;
  
  public class VectorStack {
    
    private Vector<String> stacks = new Vector<>();
    
    public void push(String element) {
      stacks.add(element);
    }
    
    public String pop() {
      return stacks.remove(stacks.size() - 1);
    }
    
    public boolean isEmpty() {
      return stacks.isEmpty();
    }
    
    public int size() {
      return stacks.size();
    }
    
  }
  ```



## 2.3.3. 집합론 관점으로 본 일반화 관계

* **일반화 관계**

  <img src="../../capture/스크린샷 2019-07-31 오전 2.13.55.png" width=400>

  > 부모 클래스 A는 전체 집합 A에 해당하고. 그 부분 집합 A1, A2, A3는 각각 A의 자식 클래스에 해당한다.

  위 예시의 제약 조건

  * A = A1 U A2 U A3
  * A1 ∩ A2 ∩ A3 = ∅



* **일반화 관계에서의 제약 조건**

  <img src="../../capture/스크린샷 2019-07-31 오전 2.18.00.png" width=400>

  * **제약 {disjoint}** : 자식 클래스 객체가 동시에 두 클래스에 속할 수 없다는 의미
  * **제약 {complete}** : 자식 클래스에 해당하는 부모 클래스의 객체와 부모 클래스의 객체에 해당하는 자식 클래스의 객체가 하나만 존재한다는 의미



* **예시) VIP 회원과 일반 회원**

  * 회원을 VIP 회원과 일반 회원으로 분류한다.
  * 회원은 회원 등급과 관계 없이 물건을 구매할 수 있다.

  <img src="../../capture/스크린샷 2019-07-31 오전 2.24.44.png" width=450>

* **집합론을 통한 연관 관계의 일반화**

  <img src="../../capture/스크린샷 2019-07-31 오전 2.28.47.png" width=550>

  > 회원들과 물건 클래스와의 연관 관계는 모든 자식 클래스에서 공통적으로 갖는 연관 관계이므로 부모 클래스인 회원 클래스로 연관 관계를 이동하는 것이 클래스 다이어그램을 간결하게 만든다.

  

집합론적인 관점에서 일반화는 상호 배타적인 부분 집합으로 나누는 과정으로 간주할 수 있으며, 이를 이용해 **상호 배타적인 특성이 요구되는 상황에 일반화 관계를 적용할 수 있다.**



* **일반화 관계를 이용한 상호 배타적 관계 모델링**

  <img src="../../capture/스크린샷 2019-07-31 오전 2.35.06.png" width=500>

  > 학생은 '놀기' 와 '공부하기' 중 어느 한 상태에만 있을 수 있다하면, 학생이 공부하는 상태이면 책만 볼 수 있고, 놀기 상태이면 장난감만 다룰 수 있다. 이런 경우에는 전형적으로 **상호 배타적인 두 상태를 모델링해야** 하며 이때 **일반화 관계가** 유용하게 사용된다.



* **특수화(specialization)**

  특수화는 일반화의 역관계, 즉 **부모 클래스에서 자식 클래스를 추출하는 과정이다.** 특수화가 필요한 경우는 어떤 속성이나 연관 관계가 특정 자식 클래스에서만 관련이 있고 다른 자식 클래스에서는 관련이 없는 경우다. 위쪽의 회원 예시를 보면 VIP 회원과 일반 회원의 차이가 없기 때문에, 회원이 왜 등급으로 구분되는지 알 수 없다. 반대로 말해 VIP 회원만 할인 쿠폰을 받을 수 있다고 한다면, 이는 **회원을 VIP 회원과 일반 회원으로 특수화하는 이유가 된다.**

  <img src="../../capture/스크린샷 2019-07-31 오전 3.13.38.png" width=550>



* **변별자(discriminator)** : UML에서는 일반화 관계의 분류 기준을 변별자라 하며 일반화 관계를 표시하는 선 옆에 변별자 정보를 표시한다. 한 인스턴가 동시에 여러 클래스에 속할 수 있는 것을 **다중 분류(multiple classification)라** 하며 '<<다중>>' 이라는 스테레오 타입을 사용해 표현한다.

  <img src="../../capture/스크린샷 2019-07-31 오전 3.22.26.png">



* **일반 회원이면서 동시에 지역 주민에게 경품 제공 요구사항 추가시**
  * **VIP-Local** (VIP 회원이면서 동시에 지역 주민)
  * **VIP-Non Local** (VIP 회원이지만 비지역민)
  * **Ord-Local** (일반 회원이면서 동시에 지역 주민)
  * **Ord-Non Local** (일반 회원이지만 비지역민)

* **집합과 일반화 관계**

  <img src="../../capture/스크린샷 2019-07-31 오전 3.31.41.png">



# 2.4. 다형성

객체지향에서 **다형성(polymorphism)은** '서로 다른 클래스의 객체가 같은 메시지를 받았을 때 각자의 방식으로 동작하는 능력'이다. 다형성은 **일반화 관계와** 함께 자식 클래스를 개별적을 다룰 필요 없이 **한 번에 처리할 수 있게 하는 수단을 제공한다.**



* **예시) 애완동물**

  **Pet 추상 클래스**

  ```java
  public abstract class Pet {
    public abstract void talk();
  }
  ```

  **Cat, Dog, Parrot 클래스**

  ```java
  public class Cat extends Pet {
    @Override
    public void talk() {
      System.out.println("야옹");
    }
  }
  
  public class Dog extends Pet {
    @Override
    public void talk() {
      System.out.println("멍멍");
    }
  }
  
  public class Parrot extends Pet {
    @Override
    public void talk() {
      System.out.println("안녀엉");
    }
  }
  ```

  **Main 클래스**

  ```java
  class Main {
  
    public static void main(String[] args) {
      Pet[] pets = new Pet[]{
          new Dog(),
          new Cat(),
          new Parrot()
      };
  
      for (Pet pet : pets) {
        pet.talk();
      }
    }
    
  }
  ```

  > Pet 클래스에 talk 메소드를 정의하고 Cat, Dog, Parrot 클래스에서 여러 가지 울음 방식에 맞게 재정의한다. 위의 메인 클래스를 보면 pets에 Dog, Cat, Parrot 객체들이 배열 요소들로 저장되고 for문을 실행시키면 Pet 타입의 pet이 각 요소들을 참조하여 각각에 정의된 talk 메서드가 실행된다. 이를 **다형성** 이라 한다.



# 2.5. 피터 코드의 상속 규칙

**피터 코드(Peter Coad)는** 상속의 오용을 막기 위해 상속의 사용을 엄격하게 제한하는 규칙들을 만들었다.

* **5가치 규칙 중에 어느 하나라도 만족하지 않는다면 상속을 사용해서는 안된다.**
  1. 자식 클래스와 부모 클래스 사이는 '역할 수행(is role played by)' 관계가 아니어야 한다.
  2. 한 클래스의 인스턴스는 다른 서브 클래스의 객체로 변환할 필요가 절대 없어야 한다.
  3. 자식 클래스가 부모 클래스의 책임을 무시하거나 재정의하지 않고 확장만 수행해야 한다.
  4. 자식 클래스가 단지 일부 기능을 재사용할 목적으로 유틸리티 역학을 수행하는 클래스를 상속하지 않아야 한다.
  5. 자식 클래스가 '역할(role)', '트랜잭션(transaction)', '디바이스(device)' 등을 특수화(specialization)해야 한다.



* **예시) 사람과 운전자, 회사원의 관계**

  <img src="../../capture/스크린샷 2019-07-31 오후 1.04.14.png" width=400>

  1. '운전자'나 '회사원'은 어떤 순간에 '사람'이 수행하는 **역할의 하나다.** 그러므로 사람과 운전자, 회사원은 상속 관계로 표현되어서는 안 되므로 규칙에 위배된다.
  2. '운전자'는 어떤 시점에서 '회사원'이 될 필요가 있으며 '회사원' 역시 '운전자'가 될 필요가 있다. 이런 경우 **객체의 변환 작입이 필요하므로** 규칙에 위배된다.
  3. 아직 클래스 정의되어 있지 않기 때문에, 점검할 수 없다.
  4. **기능만 재사용할 목적으로** 상속 관계를 표현하지는 않았으므로 규칙을 준수한다.
  5. 아직 클래스가 정의되어 있지 않기 때문에, 점검할 수 없다.