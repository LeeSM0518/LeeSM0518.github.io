# 변수 정의 및 사용법

* **변수 (variable)** : 값을 저장하는 메모리 공간. 
  * 변수는 메모리에 만들어진다.

# 연산자 종류

* **산술 연산자**

<img src="/Users/sangminlee/Library/Application Support/typora-user-images/image-20191028224535358.png" alt="image-20191028224535358" style="zoom:40%;" />

* **대입 연산자**

  <img src="/Users/sangminlee/Library/Application Support/typora-user-images/image-20191028231536807.png" alt="image-20191028231536807" style="zoom:30%;" />

* **관계 연산자**

  <img src="/Users/sangminlee/Library/Application Support/typora-user-images/image-20191028231605391.png" alt="image-20191028231605391" style="zoom:30%;" />

* **논리 연산자**

  <img src="/Users/sangminlee/Library/Application Support/typora-user-images/image-20191028231628368.png" alt="image-20191028231628368" style="zoom:30%;" />

* **비트 연산자**

  <img src="/Users/sangminlee/Library/Application Support/typora-user-images/image-20191028231652423.png" alt="image-20191028231652423" style="zoom:30%;" />

  

# 배열, 리스트 정의 및 사용법

* **배열과 리스트의 정의**
  * **배열** : 동일한 데이터형만 묶을 수 있다.
  * **리스트** : 정수, 문자열, 실수 등 서로 다른 데이터형도 하나로 묶을 수 있으며, 여러 개의 데이터가 저장되어 있는 장소이다.

* **리스트**

  ```python
  scores = [] 			# 리스트 선언
  scores.append(10) # 값 추가
  scores[0] = 80 		# 인덱스 접근
  
  for element in scores:
    print(element)
    
  len(scores) 
  min(scores)
  max(scores)
  
  3 in scores
  3 not in scores
  
  [1, 2] + [3, 4, 5] = [1, 2, 3, 4, 5]
  [1, 2] * 2 = [1, 2, 1, 2]
  
  scores = [1, 2, 3, 4, 5, 6, 7, 8]
  scores[3:6] => [4, 5, 6]
  
  index = scores.index(3) => 2
  
  scores.remove(3) => [1, 2, 4, 5, ...]
  
  a = [1, 2, 3]
  b = [1, 2, 3]
  a == b => True
  
  a.sort()
  
  scores = [10, 20, 30, 40, 50]
  values = list(scores) # 깊은 복사
  ```

  <img src="/Users/sangminlee/Library/Application Support/typora-user-images/image-20191028225901302.png" alt="image-20191028225901302" style="zoom:40%;" />

* 리스트는 참조 호출이다.

  ```python
  def func2(list):
    list[0] = 99
    
  values = [0, 1, 1, 2, 3, 5, 8]
  print(values)
  func2(values)
  print(values)
  ```

  ```
  0, 1, 1, 2, 3, 5, 8
  99, 1, 1, 2, 3, 5, 8
  ```

* 리스트 함축

  ```python
  list1 = [3, 4, 5]
  list2 = [x*2 for x in list1] 
  print(list2) 
  ```

  ```
  [6, 8, 10]
  ```

  조건 붙은 리스트 함축

  <img src="/Users/sangminlee/Library/Application Support/typora-user-images/image-20191028230203604.png" alt="image-20191028230203604" style="zoom:40%;" />

  ```
  [0, 2, 4, 6, 8]
  ```

* 2차원 배열

  ```python
  s = [ 
  	[ 1, 2, 3, 4, 5 ] ,
  	[ 6, 7, 8, 9, 10 ], 
  	[11, 12, 13, 14, 15 ] 
  ]
  
  # 행과 열의 개수를 구한다. 
  rows = len(a)
  cols = len(a[0])
  
  for r in range(rows):
  	for c in range(cols):
  		print(s[r][c], end=",")
  	print()
  ```

  <img src="/Users/sangminlee/Library/Application Support/typora-user-images/image-20191028231741388.png" alt="image-20191028231741388" style="zoom:33%;" />

# 코딩 문제

## 터틀 프로그램

* **별 그리기**

  ```python
  import turtle
  
  star = turtle.Turtle()
  
  for i in range(5):
      star.forward(50)
      star.right(144)
  ```

* **다각형 그리기**

  ```python
  import turtle
  
  polygon = turtle.Turtle()
  
  numSides = 6
  sideLength = 70
  angle = 360.0 / numSides
  
  for i in range(numSides):
      polygon.forward(sideLength)
      polygon.right(angle)
  ```

* **기본 사각형**

  ```python
  import turtle
  
  turtle.shape('turtle')
  
  turtle.forward(200)
  turtle.right(90)
  turtle.forward(200)
  turtle.right(90)
  turtle.forward(200)
  turtle.right(90)
  turtle.forward(200)
  
  for i in range(0, 4):
    turtle.forward(200)
    turtle.right(90)
  
  turtle.done()
  ```

* **거북이 랜덤 움직이기**

  ```python
  import turtle
  import random
  
  swidth, sheight, pSize, exitCount = 300, 300, 3, 0
  r, g, b, angle, dist, curX, curY = [0] * 7
  
  turtle.title('거북이가 맘대로 다니기')
  turtle.shape('turtle')
  turtle.pensize(pSize)
  turtle.setup(width = swidth + 30, height = sheight + 30)
  turtle.screensize(swidth, sheight)
  
  while True:
      r = random.random()
      g = random.random()
      b = random.random()
      turtle.pencolor((r, g, b))
  
      angle = random.randrange(0, 360)
      dist = random.randrange(1, 100)
      turtle.left(angle)
      turtle.forward(dist)
      curX = turtle.xcor()
      curY = turtle.ycor()
  
      if (-swidth / 2 <= curX and curX >= swidth / 2) and (-sheight / 2 <= curY and curY <= sheight / 2):
          pass
      else:
          turtle.penup()
          turtle.goto(0, 0)
          turtle.pendown()
  
          exitCount += 1
          if exitCount >= 5:
              break
  
  turtle.done()
  ```

* **사각형 여러개 그리기**

  ```python
  import turtle
  
  for i in range(3):
      turtle.left(20)
  
      turtle.forward(50)
      turtle.left(90)
      turtle.forward(50)
      turtle.left(90)
      turtle.forward(50)
      turtle.left(90)
      turtle.forward(50)
      turtle.left(90)
  ```

  

## 계산기 프로그램

```python
def cal(x, op, y):
    if op == '+':
        print(x, ' + ', y, ' = ', x + y)
    elif op == '-':
        print(x, ' - ', y, ' = ', x - y)
    elif op == '*':
        print(x, ' * ', y, ' = ', x * y)
    elif op == '/':
        print(x, ' / ', y, ' = ', x / y)

cal(a, c, b)
```

```
value: 1
value: 3
cal: +
1  +  3  =  4
```



## 구구단 출력하기

* **구구단 순서대로 출력**

  ```python
  for i in range(1, 10):
      for j in range(2, 10):
          print('%d x %d = %2d' %(j, i, i*j), end = ' ')
      print()
  ```

  <img src="/Users/sangminlee/Library/Application Support/typora-user-images/image-20191028232946730.png" alt="image-20191028232946730" style="zoom:100%;" />

  

* **구구단 거꾸로 출력**

  ```python
  for i in range(9, 0, -1):
      for j in range(9, 1, -1):
          print('%d x %d = %2d' %(j, i, i*j), end = ' ')
      print()
  ```

  <img src="/Users/sangminlee/Library/Application Support/typora-user-images/image-20191028233006005.png" alt="image-20191028233006005" style="zoom:30%;" />
