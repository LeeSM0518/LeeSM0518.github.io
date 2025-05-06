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

## Basic: Using a ScrollView
---

ScrollView는 여러 컴포넌트와 뷰를 포함할 수 있는 범용 스크롤 컨테이너이다. 스크롤 가능한 항목들은 서로 다른 형태일 수 있으며, horizontal 속성을 설정하면 세로뿐만 아니라 가로로도 스크롤할 수 있다.

{% raw %}
```tsx
import React from 'react';
import {Image, ScrollView, Text} from 'react-native';

const logo = {
  uri: 'https://reactnative.dev/img/tiny_logo.png',
  width: 64,
  height: 64,
};

const App = () => (
  <ScrollView>
    <Text style={{fontSize: 96}}>Scroll me plz</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{fontSize: 96}}>If you like</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{fontSize: 96}}>Scrolling down</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{fontSize: 96}}>What's the best</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{fontSize: 96}}>Framework around?</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{fontSize: 80}}>React Native</Text>
  </ScrollView>
);

export default App;
```
{% endraw %}

<br/>

ScrollView는 pagingEnabled props를 사용하여 스와이프 제스처로 뷰를 페이지 단위로 넘기도록 설정할 수 있다.

ScrollView는 크기가 제한된 항목을 표시할 때 가장 적합하다. 화면에 보이지 않더라도 ScrollView에 포함된 모든 요소와 뷰는 한 번에 렌더링된다. 따라서 화면에 다 담기지 않는 긴 목록을 표시해야 할 경우에는 `FlatList` 를 사용하는 것이 좋다.

<br/>

## Basic: Using List Views
---

React Native는 데이터 목록을 보여주는 다양한 컴포넌트를 제공한다. 일반적으로 `FlatList` 나 `SectionList` 중 하나를 사용한다.

`FlatList` 컴포넌트는 변화하지만 구조가 유사한 데이터를 스크롤 가능한 리스트 형태로 보여준다. `FlatList` 는 항목 수가 시간이 지나면 바뀔 수 있는 긴 데이터 리스트에 적합하다. `ScrollView` 와 달리, `FlatList` 는 화면에 현재 표시되는 항목만 렌더링하여 전체 항목을 한꺼번에 렌더링하지 않는다.

`FlatList` 컴포넌트를 사용하려면 `data` 와 `renderItem` 두 props가 필수이다. `data` 는 리스트의 데이터 원본이며, `renderItem` 은 데이터 원본에서 하나의 아이템을 받아 화면에 렌더링할 형식의 컴포넌트를 반환하는 함수이다.

{% raw %}
```tsx
import React from 'react';
import {FlatList, StyleSheet, Text, View} from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingTop: 22,
  },
  item: {
    padding: 10,
    fontSize: 18,
    height: 44,
  },
});

const FlatListBasics = () => {
  return (
    <View style={styles.container}>
      <FlatList
        data={[
          {key: 'Devin'},
          {key: 'Dan'},
          {key: 'Dominic'},
          {key: 'Jackson'},
          {key: 'James'},
          {key: 'Joel'},
          {key: 'John'},
          {key: 'Jillian'},
          {key: 'Jimmy'},
          {key: 'Julie'},
        ]}
        renderItem={({item}) => <Text style={styles.item}>{item.key}</Text>}
      />
    </View>
  );
};

export default FlatListBasics;
```
{% endraw %}

<br/>

데이터를 논리적인 섹션으로 나눠서, 각 섹션에 헤더를 표시하는 방식으로 렌더링하고 싶다면 `SectionList` 를 사용하는 것이 적절하다.

{% raw %}
```tsx
import React from 'react';
import {SectionList, StyleSheet, Text, View} from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingTop: 22,
  },
  sectionHeader: {
    paddingTop: 2,
    paddingLeft: 10,
    paddingRight: 10,
    paddingBottom: 2,
    fontSize: 14,
    fontWeight: 'bold',
    backgroundColor: 'rgba(247,247,247,1.0)',
  },
  item: {
    padding: 10,
    fontSize: 18,
    height: 44,
  },
});

const SectionListBasics = () => {
  return (
    <View style={styles.container}>
      <SectionList
        sections={[
          {title: 'D', data: ['Devin', 'Dan', 'Dominic']},
          {
            title: 'J',
            data: [
              'Jackson',
              'James',
              'Jillian',
              'Jimmy',
              'Joel',
              'John',
              'Julie',
            ],
          },
        ]}
        renderItem={({item}) => <Text style={styles.item}>{item}</Text>}
        renderSectionHeader={({section}) => (
          <Text style={styles.sectionHeader}>{section.title}</Text>
        )}
        keyExtractor={item => `basicListEntry-${item}`}
      />
    </View>
  );
};

export default SectionListBasics;
```
{% endraw %}

<br/>

## Basic: Platform-Specific Code
---

경우에 따라 플랫폼별로 코드를 다르게 구현하는 것이 적합할 때도 있다. 예를 들어, Android와 iOS에서 각각 다른 UI 컴포넌트를 구현하고 싶을 수 있다.

React Native는 플랫폼별 코드를 분리할 수 있도록 두 가지 방법을 제공한다.
1. `Platform` 모듈 사용하기
2. 플랫폼 전용 파일 확장자 사용하기

일부 컴포넌트의 속성은 특정 플랫폼에서만 작동할 수 있으며, 이런 속성들은 공식 문서에서 `@platform` 주석이 달려 있다.

<br/>

### Platform module
---

React Native는 어떤 앱에서 실행 중인지 확인할 수 있는 모듈을 제공한다. 이를 활용해서 플랫폼 특화된 코드를 구현할 수 있다.

{% raw %}
```tsx
import {Platform, Text, View} from "react-native";  
  
export default function PlatformSpecificCode() {  
  let platform = Platform.OS === "ios" ? "iOS" : "Android";  
  return (  
    <View style={{  
      flex: 1,  
      justifyContent: 'center',  
      alignItems: 'center',  
    }}>  
      <Text>{platform}</Text>  
    </View>  
    )  
}
```
{% endraw %}

- `Platform.IO` 는 `ios` 이거나 `android` 가 될 수 있다.

<br/>

`Platform.select` 메서드를 사용해 구현할 수도 있다. 이 메서드는 `'ios' | 'android' | 'native' | 'default'` 중에 하나의 키가 될 수 있는 객체를 전달하면 실행 중인 플랫폼에 맞는 값을 받는다. 모바일 기기에서 실행 중이라면 `ios` 또는 `android` 키로 적용되고, 그 둘이 없을 경우 `native` 키가 사용되며, 그것도 없으면 `default` 키가 적용된다.

{% raw %}
```tsx
import {Platform, StyleSheet, Text, View} from "react-native";  
  
export default function PlatformSpecificCode() {  
  let platform = Platform.OS === "ios" ? "iOS" : "Android";  
  return (  
    <View style={styles.container}>  
      <Text>{platform}</Text>  
    </View>  
    )  
}  
  
const styles = StyleSheet.create({  
  container: {  
    flex: 1,  
    justifyContent: 'center',  
    alignItems: 'center',  
    ...Platform.select({  
      ios: {  
        backgroundColor: 'blue',  
      },  
      android: {  
        backgroundColor: 'green',  
      },  
      default: {  
        backgroundColor: 'red',  
      }  
    })  
  }  
})
```
{% endraw %}

<br/>

플랫폼 특화된 컴포넌트를 사용하려면 다음과 같이 사용하면 된다.

{% raw %}
```tsx
const Component = Platform.select({  
ios: () => require('ComponentIOS'),  
android: () => require('ComponentAndroid'),  
})();  
  
<Component />;
```
{% endraw %}

<br/>

#### Detecting the Android version
---

{% raw %}
```tsx
import {Platform} from 'react-native';

if (Platform.Version === 25) {
  console.log('Running on Nougat!');
}
```
{% endraw %}

<br/>

#### Detecting the iOS version
---

{% raw %}
```tsx
import {Platform} from 'react-native';

const majorVersionIOS = parseInt(Platform.Version, 10);
if (majorVersionIOS <= 9) {
  console.log('Work around a change in behavior');
}
```
{% endraw %}

<br/>

### Platform-specific extensions
---

플랫폼별 코드가 복잡하다면 코드를 여러 파일로 나누는 것이 좋다. React Native는 파일 이름에 `.ios` 나 `.android` 확장자가 포함되어 있으면 자동으로 이를 감지하여, 다른 컴포넌트에서 해당 플랫폼에 맞는 파일을 불러온다.

<br/>

## UI: Style
---

모든 코어 컴포넌트는 `style` prop을 받는다. style 이름과 값은 웹의 CSS 방식과 유사하다. 단, 이름은 `background-color` 대신 `backgroundColor` 처럼 카멜 케이스로 작성해야 한다.

`style` prop은 일반적인 JavaScript 객체 형태로 지정할 수 있다. 또한 스타일 배열을 전달할 수 있으며, 배열의 마지막에 있는 스타일이 우선 적용되므로 이를 활용해 스타일을 상속하거나 덮어쓸 수 있다.

컴포넌트가 점점 복잡해질수록 여러 스타일을 한 곳에서 정의하기 위해 `StyleSheet.create` 를 사용하는 것이 더 깔끔한 방법이다. 다음은 그 예시이다.

{% raw %}
```tsx
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const LotsOfStyles = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.red}>just red</Text>
      <Text style={styles.bigBlue}>just bigBlue</Text>
      <Text style={[styles.bigBlue, styles.red]}>bigBlue, then red</Text>
      <Text style={[styles.red, styles.bigBlue]}>red, then bigBlue</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    marginTop: 50,
  },
  bigBlue: {
    color: 'blue',
    fontWeight: 'bold',
    fontSize: 30,
  },
  red: {
    color: 'red',
  },
});

export default LotsOfStyles;
```
{% endraw %}

<br/>

자주 사용하는 패턴 중 하나는, 컴포넌트가 style prop을 받아 내부의 서브 컴포넌트에 스타일을 전달하는 방식이다. 이 방식을 활용하면 CSS에서처럼 스타일이 "계단식으로(cascade)" 적용되도록 만들 수 있다.

<br/>

## UI: Height and Width
---

컴포넌트의 height과 width는 화면에서의 크기를 결정한다.

<br/>

### Fixed Dimensions
---

컴포넌트의 크기를 설정하는 일반적인 방법은 `style` 에 고정된 `width` 와 `height` 값을 지정하는 것이다. React Native에서의 모든 크기 단위는 단위가 없으며, 이는 density independent pixels를 의미한다.

{% raw %}
```tsx
import React from 'react';
import {View} from 'react-native';

const FixedDimensionsBasics = () => {
  return (
    <View>
      <View
        style={{
          width: 50,
          height: 50,
          backgroundColor: 'powderblue',
        }}
      />
      <View
        style={{
          width: 100,
          height: 100,
          backgroundColor: 'skyblue',
        }}
      />
      <View
        style={{
          width: 150,
          height: 150,
          backgroundColor: 'steelblue',
        }}
      />
    </View>
  );
};

export default FixedDimensionsBasics;
```
{% endraw %}

- 컴포넌트의 크기가 화면 크기에 따라 계산되지 않고 항상 일정한 값으로 고정되어야 할 때 사용된다.

<br/>

### Flex Dimensions
---

`flex` 를 스타일에 사용하면, 컴포넌트가 사용 가능한 공간을 기준으로 자동으로 확장하거나 축소되도록 만들 수 있다. 일반적으로 `flex: 1` 을 사용하며, 이는 해당 컴포넌트가 부모 안에서 가능한 모든 공간을 차지하되, 형제 컴포넌트들과 균등하게 나누도록 한다. `flex` 값이 클수록 형제 컴포넌트에 비해 더 많은 공간을 차지하게 된다.

{% raw %}
```tsx
import React from 'react';
import {View} from 'react-native';

const FlexDimensionsBasics = () => {
  return (
    <View style={{flex: 1}}>
      <View style={{flex: 1, backgroundColor: 'powderblue'}} />
      <View style={{flex: 2, backgroundColor: 'skyblue'}} />
      <View style={{flex: 3, backgroundColor: 'steelblue'}} />
    </View>
  );
};

export default FlexDimensionsBasics;
```
{% endraw %}

<br/>

### Percentage Dimensions
---

화면의 특정 비율을 채우고 싶지만 flex 레이아웃을 사용하고 싶지 않은 경우, 스타일에 퍼센트 값을 사용할 수 있다. `flex` 와 마찬가지로 퍼센트 단위의 크기를 사용하려면 부모 컴포넌트에 명확한 크기가 정의되어 있어야 한다.

{% raw %}
```tsx
import React from 'react';
import {View} from 'react-native';

const PercentageDimensionsBasics = () => {
  // Try removing the `height: '100%'` on the parent View.
  // The parent will not have dimensions, so the children can't expand.
  return (
    <View style={{height: '100%'}}>
      <View
        style={{
          height: '15%',
          backgroundColor: 'powderblue',
        }}
      />
      <View
        style={{
          width: '66%',
          height: '35%',
          backgroundColor: 'skyblue',
        }}
      />
      <View
        style={{
          width: '33%',
          height: '50%',
          backgroundColor: 'steelblue',
        }}
      />
    </View>
  );
};

export default PercentageDimensionsBasics;
```
{% endraw %}

<br/>

## UI: Layout with Flexbox
---

컴포넌트는 자식 요소들의 레이아웃을 Flexbox 알고리즘을 사용하여 지정할 수 있다. Flexbox는 다양한 화면 크기에서도 일관된 레이아웃을 제공하도록 설계되었다.

> Flexbox는 React Native에서 웹의 CSS와 거의 동일하게 동작하지만, 몇 가지 기본값에서 차이가 있다. `flexDirection` 의 기본값은 `row` 가 아닌 `column` 이다. `alignContent` 의 기본값은 `stretch` 가 아닌 `flex-start` 이다. `flexShrink` 의 기본값은 `1` 이 아닌 `0` 이다. `flex` 속성은 단일 숫자만 지원한다.
{: .prompt-warning }

<br/>

### Flex

`flex` 는 자식 요소들이 주 축(main axis)을 따라 남은 공간을 어떻게 "채울지"를 정의한다. 각 요소의 flex 값에 따라 공간이 나누어진다.

{% raw %}
```tsx
import React from 'react';
import {StyleSheet, View} from 'react-native';

const Flex = () => {
  return (
    <View
      style={[
        styles.container,
        {
          flexDirection: 'column',
        },
      ]}>
      <View style={{flex: 1, backgroundColor: 'red'}} />
      <View style={{flex: 2, backgroundColor: 'darkorange'}} />
      <View style={{flex: 3, backgroundColor: 'green'}} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
});

export default Flex;
```
{% endraw %}

<br/>

### Flex Direction
---

`flexDirection` 은 자식 요소들이 어떤 방향으로 배치될지를 제어한다. 이를 주 축(main axis)이라고 부르며, 주 축에 수직인 축은 교차 축(cross axis)이라고 한다. 줄바꿈이 발생할 경우, 교차 축 방향으로 줄이 새로 배치된다.

| **값**          | **의미**   | **동작 설명**                                                      |
| -------------- | -------- | -------------------------------------------------------------- |
| column _(기본값)_ | 위 → 아래   | 자식 요소들을 위에서 아래로 정렬합니다. 줄바꿈이 되면 다음 줄은 컨테이너의 **위쪽 오른쪽**에서 시작합니다. |
| row            | 왼쪽 → 오른쪽 | 자식 요소들을 왼쪽에서 오른쪽으로 정렬합니다. 줄바꿈 시 다음 줄은 **왼쪽 아래**에서 시작됩니다.       |
| column-reverse | 아래 → 위   | 자식 요소들을 아래에서 위로 정렬합니다. 줄바꿈 시 다음 줄은 **아래쪽 오른쪽**에서 시작됩니다.        |
| row-reverse    | 오른쪽 → 왼쪽 | 자식 요소들을 오른쪽에서 왼쪽으로 정렬합니다. 줄바꿈 시 다음 줄은 **오른쪽 아래**에서 시작됩니다.      |

<br/>

{% raw %}
```tsx
import React, {useState} from 'react';
import {StyleSheet, Text, TouchableOpacity, View} from 'react-native';
import type {PropsWithChildren} from 'react';

const FlexDirectionBasics = () => {
  const [flexDirection, setflexDirection] = useState('column');

  return (
    <PreviewLayout
      label="flexDirection"
      values={['column', 'row', 'column-reverse', 'row-reverse']}
      selectedValue={flexDirection}
      setSelectedValue={setflexDirection}>
      <View style={[styles.box, {backgroundColor: 'powderblue'}]} />
      <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'steelblue'}]} />
    </PreviewLayout>
  );
};

type PreviewLayoutProps = PropsWithChildren<{
  label: string;
  values: string[];
  selectedValue: string;
  setSelectedValue: (value: string) => void;
}>;

const PreviewLayout = ({
  label,
  values,
  selectedValue,
  setSelectedValue,
  children,
}: PreviewLayoutProps) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default FlexDirectionBasics;
```
{% endraw %}

<br/>

### Layout Direction
---

레이아웃 `direction` 은 계층 구조에서 자식 요소와 텍스트가 어떤 방향으로 배치될지를 지정한다. 또한 `start` 와 `end` 가 어떤 방향을 의미하는지도 이 설정에 따라 달라진다.

- `LTR` (default value) : 텍스트와 자식 요소들이 왼쪽에서 오른쪽으로 배치된다.
- `RTL` : 텍스트와 자식 요소들이 오른쪽에서 왼쪽으로 배치된다.

<br/>

{% raw %}
```tsx
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';
import type {PropsWithChildren} from 'react';

const DirectionLayout = () => {
  const [direction, setDirection] = useState('ltr');

  return (
    <PreviewLayout
      label="direction"
      selectedValue={direction}
      values={['ltr', 'rtl']}
      setSelectedValue={setDirection}>
      <View style={[styles.box, {backgroundColor: 'powderblue'}]} />
      <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'steelblue'}]} />
    </PreviewLayout>
  );
};

type PreviewLayoutProps = PropsWithChildren<{
  label: string;
  values: string[];
  selectedValue: string;
  setSelectedValue: (value: string) => void;
}>;

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}: PreviewLayoutProps) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default DirectionLayout;
```
{% endraw %}

<br/>

### Justify Content
---

`justifyContent` 는 컨테이너의 주 축(main axis)을 따라 자식 요소들을 어떻게 정렬할지를 지정한다. 예를 들어, `flexDirection`이 `row` 인 경우에는 수평 정렬, `column` 인 경우에는 수직 정렬에 사용된다. 이 속성을 사용하면 자식 요소를 중앙에 정렬하거나, 간격을 동일하게 나눌 수 있다.

| **값**                  | **동작**        | **설명**                                                   |
| ---------------------- | ------------- | -------------------------------------------------------- |
| **flex-start** _(기본값)_ | 시작 정렬         | 자식들을 주 축의 **시작 지점**에 붙여 정렬                               |
| **flex-end**           | 끝 정렬          | 자식들을 주 축의 **끝 지점**에 정렬                                   |
| **center**             | 중앙 정렬         | 자식들을 주 축의 **가운데**에 정렬                                    |
| **space-between**      | 양쪽 끝 + 간격 균등  | **첫 요소는 시작**, **마지막 요소는 끝**에 정렬, **남은 공간은 요소 사이에 균등 배분** |
| **space-around**       | 바깥도 포함해 간격 균등 | 자식들 주변에 동일한 간격 배치. 양 끝 요소 바깥쪽에도 **공간이 포함됨**              |
| **space-evenly**       | 완전 균등 간격      | **요소 간, 시작과 첫 요소 사이, 끝과 마지막 요소 사이 모두 같은 간격**             |

<br/>

{% raw %}
```tsx
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';
import type {PropsWithChildren} from 'react';

const JustifyContentBasics = () => {
  const [justifyContent, setJustifyContent] = useState('flex-start');

  return (
    <PreviewLayout
      label="justifyContent"
      selectedValue={justifyContent}
      values={[
        'flex-start',
        'flex-end',
        'center',
        'space-between',
        'space-around',
        'space-evenly',
      ]}
      setSelectedValue={setJustifyContent}>
      <View style={[styles.box, {backgroundColor: 'powderblue'}]} />
      <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'steelblue'}]} />
    </PreviewLayout>
  );
};

type PreviewLayoutProps = PropsWithChildren<{
  label: string;
  values: string[];
  selectedValue: string;
  setSelectedValue: (value: string) => void;
}>;

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}: PreviewLayoutProps) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default JustifyContentBasics;
```
{% endraw %}

<br/>

### Align Items
---

`alignItems` 는 자식 요소들을 교차 축(cross axis)을 따라 어떻게 정할지를 정의한다. `justifyContent` 가 주 축(main axis)에 대해 적용되는 반면, `alignItems` 는 교차 축에 대해 적용된다.

|**값**|**동작**|**설명**|
|---|---|---|
|**stretch** _(기본값)_|늘리기|자식 요소들을 교차 축 방향으로 **컨테이너의 크기에 맞게 늘림** (단, 자식 요소에 고정된 크기가 없을 경우에만)|
|**flex-start**|시작 정렬|자식 요소들을 교차 축의 **시작 지점**에 정렬|
|**flex-end**|끝 정렬|자식 요소들을 교차 축의 **끝 지점**에 정렬|
|**center**|중앙 정렬|자식 요소들을 교차 축의 **가운데**에 정렬|
|**baseline**|기준선 정렬|자식 요소들을 **공통 기준선(baseline)**에 정렬. 자식 요소 중 하나를 기준선으로 지정 가능|

<br/>

{% raw %}
```tsx
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';
import type {PropsWithChildren} from 'react';

const AlignItemsLayout = () => {
  const [alignItems, setAlignItems] = useState('stretch');

  return (
    <PreviewLayout
      label="alignItems"
      selectedValue={alignItems}
      values={['stretch', 'flex-start', 'flex-end', 'center', 'baseline']}
      setSelectedValue={setAlignItems}>
      <View style={[styles.box, {backgroundColor: 'powderblue'}]} />
      <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
      <View
        style={[
          styles.box,
          {
            backgroundColor: 'steelblue',
            width: 'auto',
            minWidth: 50,
          },
        ]}
      />
    </PreviewLayout>
  );
};

type PreviewLayoutProps = PropsWithChildren<{
  label: string;
  values: string[];
  selectedValue: string;
  setSelectedValue: (value: string) => void;
}>;

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}: PreviewLayoutProps) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
    minHeight: 200,
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default AlignItemsLayout;
```
{% endraw %}


<br/>

## Reference
---

<https://reactnative.dev/docs/getting-started>
