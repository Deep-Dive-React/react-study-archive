# 5장

# 리액트와 상태관리 라이브러리

## 5.1 상태관리는 왜 필요할까?

상태란 무엇일까요?

상태는 어떠한 의미를 지닌 값이며 애플리케이션의 시나리오에 따라 지속적으로 변경될 수 있는 값입니다.

상태로 분류되는 것들은 다음과 같다고 합니다.

- UI : 다크/라이트모드, 라디오, input, 알림창의 노출 여부
- URL : 쿼리에 사용되는 값들 ex) [https://www.google.com/search?q=상태관리](https://www.google.com/search?q=%EC%83%81%ED%83%9C%EA%B4%80%EB%A6%AC&) 에서 ‘q=상태관리’
- 폼 : 로딩중, 제출완료, 접근불가, 유효
- 서버에서 가져온값 : API 요청

⇒ 쉽게 말하자면 자동으로 변하는 것들? 이라고 생각하면 될 것 같습니다.

이러한 상태들을 관리하는 것은 서비스가 커짐에 따라 효과적으로 해야합니다. 

만약 전체 애플리케이션 내에서 전체적으로 관리해줘야하는 상태가 있다면 어떻게 해야할까요?

상태를 전역 변수에 둬야할까요? 

별도의 클로저를 만들어야 할까요? 그렇다면 유효한 범위는 어떻게 제한해야 할까요? 

상태변화가 일어남에 따라 tearing 현상이 나오게 되는데 이를 어떻게 방지할 수 있을까요?

> tearing 현상이란 ?

화면의 이미지가 수직 방향으로 분할되어 보이는 현상으로, 화면의 위쪽 부분은 이전 프레임의 이미지를, 아래쪽 부분은 새로운 프레임의 이미지를 보여줍니다. 이로 인해 화면이 끊어져 보이는 것처럼 느껴질 수 있습니다.

![image](https://github.com/Deep-Dive-React/react-study-archive/assets/42230162/b995b933-6be2-406e-9862-3ae415f28732)

## 5.1.1 리액트 상태 관리의 역사

역사를 알면 왜 쓰는지에 대해 알 수 있습니다

다음은 상태관리를 하기 위한 역사를 알아보겠습니다.

## Flux 패턴의 등장 (1/5)
기존의 웹 애플리케이션은 MVC 패턴을 주로 사용했다고 합니다.
MVC 패턴은 모델(데이터를 처리하는 로직)과 뷰(사용자에게 보여주는 화면)을 분리하고자 나온 패턴입니다.
MVC 패턴에서 Controller는 Model의 데이터를 조회하거나 업데이트하는 역할을 합니다.  
Model이 업데이트 되면, View는 화면에 반영합니다. View가 Model을 업데이트 할 수도 있습니다. Model이 업데이트 되어 View가 따라서 업데이트 되고, 업데이트 된 View가 다시 다른 Model을 업데이트 한다면, 또 다른 View가 업데이트 될 수 있습니다.

하지만 밑에 그림을 보시면 알 수 있듯 모델과 뷰가 늘어날수록 복잡해져 이를 추적하거나 이해하기가 어려운 상황이 되었습니다.

<img width="428" alt="image" src="https://github.com/Deep-Dive-React/react-study-archive/assets/42230162/aaf4f88c-2bc1-47e9-b845-332a8942ea9f">

페이스북에서 이야기하는 MVC의 양방향 데이터 흐름이 만들어 낸 예측하기 어려운 버그 중 하나는 알림 버그라고 합니다.  
페이스북에 로그인 했을 때, 화면 위 메시지 아이콘에 알림이 떠 있지만, 그 메시지 아이콘을 클릭하면 아무런 메시지가 없는 버그를 보신 적이 있으실 겁니다.

![image](https://github.com/Deep-Dive-React/react-study-archive/assets/42230162/e6f753dd-2d1a-44b4-b1d3-0fdcd955896c)

이러한 문제를 해결하기 위해 페이스북에서는 단방향 솔루션인 Flux를 개발하게 됐다고 합니다.
  
<img width="509" alt="image" src="https://github.com/Deep-Dive-React/react-study-archive/assets/42230162/be15e142-d82a-4fbc-8e67-c6e3f1a9e46e">

데이터가 항상 단방향 데이터으로 전달되어 데이터 변화를 휠씬 예측하기 쉽게 했다고 합니다.   

Flux를 크게 Dispatcher, Store, View 세 부분으로 구성됩니다.

### Dispatcher
Flux의 모든 데이터 흐름을 관리하는 허브 역할을 하는 부분입니다. Action이 발생되면 Dispatcher로 전달되는데, Dispatcher는 전달된 Action을 보고, 등록된 콜백 함수를 실행하여 Store에 데이터를 전달합니다. Dispatcher는 전체 어플리케이션에서 한 개의 인스턴스만 사용됩니다.

### Store
액션의 타입에 따라 어떻게 변경할지 정의되어있는 곳입니다.
어플리케이션의 모든 상태 변경은 Store에 의해 결정이 됩니다. Dispatcher로 부터 메시지를 수신 받기 위해서는 Dispatcher에 콜백 함수를 등록해야 합니다. Store가 변경되면 View에 변경되었다는 사실을 알려주게 됩니다. Store은 싱글톤으로 관리됩니다.

### View
리액트의 컴포넌트에 해당하는 부분으로 스토어에서 만들어진 데이터를 가져와 화면에 렌더링하는 역할을 맡습니다.
또한 자식 View로 데이터를 흘려 보내는 뷰 컨트롤러의 역할도 함께 합니다.
그리고 그림처럼 사용자가 입력하여 생긴 액션에 따라 상태를 업데이트 하기 위해 dispatcher로 넘기기도 합니다.

### Action
Dispatcher에서 콜백 함수가 실행 되면 Store가 업데이트 되게 되는데, 이 콜백 함수를 실행 할 떼 데이터가 담겨 있는 객체가 인수로 전달 되어야 합니다. 이 전달 되는 객체를 Action이라고 하는데, Action은 대채로 액션 생성자(Action creator)에서 만들어집니다.

그렇다면 실제 구현을 한번 해보겠습니다.
~~~
// actions.js
export const INCREMENT = 'INCREMENT';
export const DECREMENT = 'DECREMENT';

export const increment = () => ({
  type: INCREMENT
});

export const decrement = () => ({
  type: DECREMENT
});
~~~
액션에서는 위와 같이 타입을 지정해준다고 합니다.

~~~
// dispatcher.js
const listeners = [];
let state = { count: 0 };

export const dispatch = action => {
  state = reducer(state, action);
  listeners.forEach(listener => listener());
};

export const getState = () => state;

export const subscribe = listener => {
  listeners.push(listener);
};

const reducer = (state, action) => {
  switch (action.type) {
    case INCREMENT:
      return {
        ...state,
        count: state.count + 1
      };
    case DECREMENT:
      return {
        ...state,
        count: state.count - 1
      };
    default:
      return state;
  }
};

~~~
dispatcher는 스토어에 action을 넘겨주는 역할을 합니다.  
reducer가 스토어의 역할을 한다고 합니다. 보시면 내부에 각각의 액션에 따른 return 값에 대한 정의가 되어있습니다.
~~~
// Counter.js
import React from 'react';
import { dispatch, getState, subscribe, increment, decrement } from './dispatcher';

class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = getState();
  }

  componentDidMount() {
    subscribe(this.handleChange);
  }

  handleChange = () => {
    this.setState(getState());
  };

  handleIncrement = () => {
    dispatch(increment());
  };

  handleDecrement = () => {
    dispatch(decrement());
  };

  render() {
    const { count } = this.state;
    return (
      <div>
        <h2>Counter: {count}</h2>
        <button onClick={this.handleIncrement}>Increment</button>
        <button onClick={this.handleDecrement}>Decrement</button>
      </div>
    );
  }
}

export default Counter;
~~~
마지막 뷰에 해당하는 코드입니다. 이를 통해 단방향 Flux 패턴을 구현할 수 있습니다.  

결국 데이터의 흐름을 추적하고 코드를 이해하기 한결 쉬워졌다고 합니다.  

리액트와 잘 맞아 flux 패턴을 따르는 다음과 같이 다양한 라이브러리가 등장하게 됐다고 합니다.  
alt, refluxjs, nuclearjs, fluxible, fluxxor

## Redux의 등장 (2/5)

Redux는 flux구조를 구현하기 위해 등장했다고 합니다.  

또한 elm 아키텍쳐를 도입해서 구현을 했다고 하는데 elm에 대해 알아보죠
elm은 React와 같은 웹 사이트나 웹 어플리케이션을 만드는 데 사용하는 툴과 경쟁하고 있다고도 하네요  
![image](https://github.com/Deep-Dive-React/react-study-archive/assets/42230162/f882f25e-cf46-44e9-91c5-971772f629bc){: width="50%"}


Elm은 단방향 데이터 흐름을 기반으로하는 아키텍처를 제공하며, 강력한 타입 시스템과 장애 회복 기능을 포함하고 있어 안정적이고 확장 가능한 웹 애플리케이션을 쉽게 개발할 수 있습니다.

Elm 애플리케이션은 모델(Model), 뷰(View), 업데이트 함수(Update) 구조를 가지고 있습니다.  

모델은 항상 업데이트 함수(update)에 의해서만 변경됩니다.  
모델의 상태를 기반으로 사용자에게 보여지는 화면을 나타냅니다.

뷰는 순수 함수(pure function)로 정의되어야 하며, 모델을 입력으로 받아 HTML을 출력하는 함수입니다.  
사용자 입력이나 외부 이벤트에 의해 모델의 상태를 업데이트하는 함수입니다.

업데이트는 모델을 수정하는 방식을 말합니다. 위에서 flux 패턴의 store와 상태를 관리하고 업데이트한다는 점이 비슷합니다.  

이러한 편의성을 바탕으로 리액트의 표준으로 자리잡았다고 합니다.

## ContextAPi와 useContext (3/5)
## 훅의 탄생과 react query, swr (4/5)
## recoil, zustand, jotai, valito에 이르기까지 (5/5)
