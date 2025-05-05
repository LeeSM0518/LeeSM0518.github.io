---
title: React Native Docs
date: 2025-05-01 00:00:00 +0900
categories: react-native
tags:
  - react-native
  - app
description: React Native 공식 문서 정리
---

![image1](/assets/img/react-native1.png)

## Basic: Core Components and Native Components
---

React Native는 React와 각 플랫폼의 네이티브 기능을 활용해 Android와 iOS 애플리케이션을 개발할 수 있는 오픈 소스 프레임워크이다. 먼저 React Native가 어떻게 동작하는지 알아보겠습니다.

<br/>

### Views and mobile development
---

Android와 iOS 개발에서 `view` 는 UI의 기본 빌딩 블록이다. 텍스트, 이미지, 사용자 입력, 라인, 버튼 등을 나타내는데 사용된다. 

![image2](/assets/img/react-native2.png)

<br/>

### Native Components
---

런타임시, React Native는 해당 컴포넌트에 대응되는 Android 및 iOS 뷰를 생성한다. React Native 컴포넌트는 Android와 iOS의 동일한 뷰를 기반으로 하기 때문에, React Native 앱은 다른 앱과 동일한 외형, 성능을 가진다. 이러한 플랫폼 기반 컴포넌트를 'Native Components' 라고 부른다.

React Native는 앱 개발을 바로 시작할 수 있도록 필수적이고 즉시 사용할 수 있는 네이티브 컴포넌트들을 기본으로 제공한다. 이를 React Native의 Core Components 라고 한다.

<br/>

### Core Components
---

React Native에는 다양한 기능을 위한 Core Component들이 많이 있다. 대부분 다음과 같은 핵심 컴포넌트를 주로 사용하게 된다.


| React Native UI Component | Android View   | iOS View         | Web Analog             | Description                                                   |
| ------------------------- | -------------- | ---------------- | ---------------------- | ------------------------------------------------------------- |
| `<View>`                  | `<ViewGroup>`  | `<UIView>`       | non-scrolling  `<div>` | flexbox, style, touch handling, accessibility controls를 지원한다. |
| `<Text>`                  | `<TextView>`   | `<UITextView>`   | `<p>`                  | 문자열을 표시하고 스타일을 적용하며 중첩할 수 있고, 터치 이벤트를 처리할 수 있다.               |
| `<Image>`                 | `<ImageView>`  | `<UIImageView>`  | `<img>`                | 다양한 종류의 이미지를 표시한다.                                            |
| `<ScrollView>`            | `<ScrollView>` | `<UIScrollView>` | `<div>`                | 여러 컴포넌트와 뷰를 포함할 수 있는 범용 스크롤 컨테이너                              |
| `<TextInput>`             | `<EditText>`   | `<UITextField>`  | `<input type="text">`  | 사용자 입력                                                        |

<br/>

다음과 같이 여러 Core Component를 사용하는 방법에 대해서 살펴볼 예정이다.

{% raw %}
```tsx
import React from 'react';
import {View, Text, Image, ScrollView, TextInput} from 'react-native';

const App = () => {
  return (
    <ScrollView>
      <Text>Some text</Text>
      <View>
        <Text>Some more text</Text>
        <Image
          source={{
            uri: 'https://reactnative.dev/docs/assets/p_cat2.png',
          }}
          style={{width: 200, height: 200}}
        />
      </View>
      <TextInput
        style={{
          height: 40,
          borderColor: 'gray',
          borderWidth: 1,
        }}
        defaultValue="You can type in me"
      />
    </ScrollView>
  );
};

export default App;
```
{% endraw %}

<br/>

## Basic: React Fundamentals
---

React Native는 React 위에서 실행된다. React는 JavaScript로 UI를 만들기 위한 오픈 소스 라이브러리이다. React Native를 잘 활용하려면 React 자체를 이해하는 것이 도움이 된다. 다음으로 React의 핵심 개념에 대해 살펴보자.

<br/>

React의 핵심 개념은 다음과 같다.
- components
- JSX
- props
- state

<br/>

### Your first component
---

{% raw %}
```tsx
import {Text} from "react-native";  
  
export default function ReactFundamentals() {  
  return (  
    <Text>Hello, I am your cat!</Text>  
  )  
}
```
{: file='app/(tabs)/(home)/react-fundamentals/index.tsx'}
{% endraw %}

- React Native의 `Text` 코어 컴포넌트를 `import` 하여 `ReactFundamentals` 컴포넌트를 만들었다.
- 이와 같은 함수형 컴포넌트가 반환하는 값은 React 엘리먼트로 렌더링된다.
- `export default` 문법을 사용하여 앱 전역에서 사용할 수 있도록 내보낸다.

<br/>

### JSX
---

React와 React Native는 JSX를 사용한다. JSX는 `<Text>Hello, I am your cat!</Text>` 와 같이 자바스크립트 안에 엘리먼트를 작설할 수 있는 문법이다. 그러므로 다음과 같이 JSX에 변수를 넣을 수 있다.

{% raw %}
```tsx
import {Text} from "react-native";  
  
export default function ReactFundamentals() {  
  const name = "Maru";  
  return (  
    <Text>Hello, I am {name}!</Text>  
  )  
}
```
{: file='app/(tabs)/(home)/react-fundamentals/index.tsx'}
{% endraw %}

<br/>

다음과 같이 중괄호 안에서 표현식도 사용할 수 있다.

{% raw %}
```tsx
import {Text, View} from "react-native";  
  
export default function ReactFundamentals() {  
  const name = "Maru";  
  return (  
    <View>  
      <Text>Hello, I am {name}!</Text>  
      <Text>Hello, I am {getFullName("Sangmin", "lee")}!</Text>  
    </View>  
    )  
}  
  
const getFullName = (firstName: string, lastName: string) => {  
  return `${firstName} ${lastName}`;  
}
```
{: file='app/(tabs)/(home)/react-fundamentals/index.tsx'}
{% endraw %}

<br/>

### Custom Components
---

React에서는 컴포넌트를 서로 중첩하여 새로운 컴포넌트를 만들 수 있다. 이렇게 중첩 가능하고 재사용 가능한 컴포넌트는 React 패러다임의 핵심이다.

다음과 같이 `View` 컴포넌트 아래에 `Text` 와 `TextInput` 컴포넌트를 중첩할 수 있다.

{% raw %}
```tsx
import React from 'react';
import {Text, TextInput, View} from 'react-native';

const Cat = () => {
  return (
    <View>
      <Text>Hello, I am...</Text>
      <TextInput
        style={{
          height: 40,
          borderColor: 'gray',
          borderWidth: 1,
        }}
        defaultValue="Name me!"
      />
    </View>
  );
};

export default Cat;
```
{% endraw %}

<br/>

다음과 같이 컴포넌트를 여러 번 사용할 수도 있다.

{% raw %}
```tsx
import React from 'react';
import {Text, View} from 'react-native';

const Cat = () => {
  return (
    <View>
      <Text>I am also a cat!</Text>
    </View>
  );
};

const Cafe = () => {
  return (
    <View>
      <Text>Welcome!</Text>
      <Cat />
      <Cat />
      <Cat />
    </View>
  );
};

export default Cafe;
```
{% endraw %}

- `Cafe` 컴포넌트는 각 `Cat` 자식 컴포넌트들의 부모 컴포넌트가 된다.

<br/>

### Props
---

Props는 "properties"의 줄임말이며, React 컴포넌트를 커스텀할 수 있게 해준다.

{% raw %}
```tsx
import React from 'react';
import {Text, View} from 'react-native';

type CatProps = {
  name: string;
};

const Cat = (props: CatProps) => {
  return (
    <View>
      <Text>Hello, I am {props.name}!</Text>
    </View>
  );
};

const Cafe = () => {
  return (
    <View>
      <Cat name="Maru" />
      <Cat name="Jellylorum" />
      <Cat name="Spot" />
    </View>
  );
};

export default Cafe;
```
{% endraw %}

<br/>

React Native의 대부분의 코어 컴포넌트는 props로 커스텀 할 수 있다. 예를 들어 `Image` 는 `source` prop을 전달하여 이미지를 정의할 수 있다.

{% raw %}
```tsx
import React from 'react';
import {Text, View, Image} from 'react-native';

const CatApp = () => {
  return (
    <View>
      <Image
        source={{
          uri: 'https://reactnative.dev/docs/assets/p_cat1.png',
        }}
        style={{width: 200, height: 200}}
      />
      <Text>Hello, I am your cat!</Text>
    </View>
  );
};

export default CatApp;
```
{% endraw %}

<br/>

> JSX에서는 JavaScript 값을 `{}` 로 참조한다. 배열이나 숫자처럼 문자열이 아닌 값을 props로 전달할 때 유용하다(`<Cat food={["fish","kibble"]} age={2} />`).  하지만 JavaScript 객체는 자체적으로 중괄호 `{}` 로 정의되므로, JSX에서 객체를 전달할 때는 그것을 중괄호로 한 번 더 감싸야 한다.
{: .prompt-tip }

<br/>

### State
---

props가 컴포넌트의 렌더링 방식을 설정하는 "인자(argument)"라면, state는 컴포넌트의 "개인적인 데이터 저장소" 라고 볼 수 있다. state는 시간이 지나며 변화하거나 사용자 상호작용에 따라 바뀌는 데이터를 다룰 때 유용하다.

컴포넌트에 상태를 추가하려면 React의 `useState` 훅을 호출하면 된다. 훅은 React의 기능에 "접근(hook into)" 할 수 있게 해주는 특수한 함수이다. 다시 말해 `useState` 는 함수형 컴포넌트에 상태를 추가할 수 있도록 해주는 훅이다.

{% raw %}
```tsx
import React, {useState} from 'react';
import {Button, Text, View} from 'react-native';

type CatProps = {
  name: string;
};

const Cat = (props: CatProps) => {
  const [isHungry, setIsHungry] = useState(true);

  return (
    <View>
      <Text>
        I am {props.name}, and I am {isHungry ? 'hungry' : 'full'}!
      </Text>
      <Button
        onPress={() => {
          setIsHungry(false);
        }}
        disabled={!isHungry}
        title={isHungry ? 'Give me some food, please!' : 'Thank you!'}
      />
    </View>
  );
};

const Cafe = () => {
  return (
    <>
      <Cat name="Munkustrap" />
      <Cat name="Spot" />
    </>
  );
};

export default Cafe;
```
{% endraw %}

- React로부터 `useState` 를 import 한다.
- `useState` 로 `isHungry` 상태를 생성한다.
- `useState` 를 호출하면 두 가지가 일어난다.
    - 초기값과 함께 상태 변수를 생성한다.
    - 그 상태 변수의 값을 변경할 수 있는 함수도 함께 생성한다.

<br/>

## Basic: Handling Text Input
---

`TextInput` 은 사용자가 텍스트를 입력할 수 있게 하는 코어 컴포넌트이다. 텍스트가 변경될 때마다 호출될 함수를 받는 `onChangeText` prop 과 텍스트 입력이 완료되었을 때 호출될 함수를 받는 `onSubmitEditing` prop을 갖는다.

{% raw %}
```tsx
import React, {useState} from 'react';
import {Text, TextInput, View} from 'react-native';

const PizzaTranslator = () => {
  const [text, setText] = useState('');
  return (
    <View style={{padding: 10}}>
      <TextInput
        style={{height: 40, padding: 5}}
        placeholder="Type here to translate!"
        onChangeText={newText => setText(newText)}
        defaultValue={text}
      />
      <Text style={{padding: 10, fontSize: 42}}>
        {text
          .split(' ')
          .map(word => word && '🍕')
          .join(' ')}
      </Text>
    </View>
  );
};

export default PizzaTranslator;
```
{% endraw %}



<br/>

## Reference
---

<https://reactnative.dev/docs/getting-started>
