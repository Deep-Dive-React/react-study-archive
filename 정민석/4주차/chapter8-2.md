# 8.2장
## 리액트 팀이 권장하는 리액트 테스트 라이브러리
~~~
1.1 + 0.1 === 1.2 //false
~~~
개발을 하다보면 빠짐없이 에러가 생기는 것 같아요. 이는 프로그램이 복잡해질수록 개발자가 제어할 수 있는 범위를 벗어나기 시작하게 됩니다.  
그래서 테스트를 통해 원래 의도한대로 프로그램이 작동되는지 확인할 수 있습니다.  
> 백엔드와 프론트엔드 테스트 차이  

백엔드는 일반적으로 화이트박스 테스트를 진행하며 프론트엔드는 주로 블랙박스 테스트를 진행합니다.

프론트엔드에서의 테스트는 일반적인 사용자가 사용하는 환경과 유사하며 이 때문에 모든 경우의 수를 고려해야 합니다.

또한 크롬, 파이어폭스, 엣지, 웨일 등 다양한 브라우저에서 발생될 수 있는 시나리오를 고려해야 하기 때문에 테스트가 중요하다고 합니다.

그로 인해 다양한 테스팅 라이브러리들이 등장하였고 교재에서는 React Testing Library 위주로 다루게 됩니다.

### 8.2.1 React Testing Library란?
React Testing Library는 React 애플리케이션의 컴포넌트를 테스트하기 위한 도구입니다. 

이 라이브러리를 이해하려면 React Testing Library가 기반으로 하는 DOM Testing Library에 대해 알아둬야 합니다.

> DOM Testing Library

특징 : jsdom 기반으로 만들어짐
(jsdom : html이 없는 js만 존재하는 환경 ex_) Node.js같은 환경에서 HTML과 DOM을 사용할 수 있도록 해주는 라이브러리)

~~~
const jsdom = require('jsdom');
const { JSDOM } = jsdom;

// HTML 코드 작성
const html = `
  <html>
    <body>
      <div id="container">
        <button id="myButton">Click Me</button>
        <span id="displayText"></span>
      </div>
    </body>
  </html>
`;

// jsdom을 사용하여 DOM 생성
const dom = new JSDOM(html);
global.document = dom.window.document;

// 테스트 시작
test('버튼 클릭 시 텍스트 변경', () => {
  // 버튼 요소 가져오기
  const button = document.getElementById('myButton');
  expect(button).not.toBeNull();

  // 버튼 클릭 이벤트 발생시키기
  button.click();

  // 클릭 이후 텍스트 확인
  const displayText = document.getElementById('displayText');
  expect(displayText.textContent).toBe('Clicked');
});

~~~
jsdom 패키지를 사용하여 빈 HTML 페이지를 생성하고, 이 페이지에 버튼과 텍스트 요소를 추가합니다. 그런 다음 이 요소들을 테스트하여 버튼을 클릭하면 텍스트가 변경되는지 확인하는 예제입니다.

이처럼 jsdom을 이용하면 HTML이 있는것 처럼 DOM을 불러오고 조작할 수 있습니다.

이를 활용하여 테스트를 하는 것이 React Testing Library입니다. 리액트 컴포넌트를 실제로 렌더링하지 않고도 원하는대로 렌더링 되는지 확인할 수 있다고 합니다.

추가적으로 Provider, 훅, 등 다양한 요소들도 테스트 할 수 있습니다.

### 8.2.2 자바스크립트 테스트의 기초

간단한 테스트 예제를 살펴 봅시다.
~~~
function sum(a,b){
  return a+b
}
~~~

이런 함수가 있을때
~~~
let actual = sum(1,2)
let expected = 3
if (expected !== actual){
  throw new Error('에러')
}
~~~
이런식으로 함수의 return값과 실제 예상값을 비교하여 둘의 차이를 비교합니다.

테스트는 대부분 다음과 같은 과정을 거치게 됩니다.
1. 테스트할 함수나 모듈을 선정한다.
2. 함수나 모듈이 반환하기 기대하는 값을 적는다.
3. 함수나 모듈의 실제 반환값을 적는다.
4. 3번의 기대에 따라 2번의 기대가 일치하는지 확인한다.
5. 기대하는 결과를 반환한다면 테스트는 성공, 아니면 에러를 던진다.

보통 테스트 코드를 작성할 때는 기본 파일(테스트 할 프로그램)이 있고 다른 파일에서 import 불러와서 작성합니다.

이를 위해 node.js에서 assert 라이브러리를 제공한다고 합니다.

> 테스팅 프레임워크

유명한 javascript 테스팅 프레임워크에는 Jest, Mocha, Karma, Jasmine등이 있습니다.

Jest 코드 예제
~~~
// math.js

// 덧셈 함수
function sum(a, b) {
  return a + b;
}

// 뺄셈 함수
function subtract(a, b) {
  return a - b;
}

// 함수를 모듈로 내보냅니다.
module.exports = {
  sum,
  subtract
};
~~~

~~~
// test.js

// 테스트할 함수 또는 모듈을 불러옵니다.
const { sum, subtract } = require('./math');

// describe() 블록은 테스트 스위트를 정의합니다.
describe('Math functions', () => {
  // 각각의 it() 블록은 특정 테스트 케이스를 정의합니다.
  it('should add two numbers correctly', () => {
    // expect() 함수를 사용하여 예상되는 결과를 확인합니다.
    expect(sum(1, 2)).toBe(3);
    expect(sum(-1, 1)).toBe(0);
    expect(sum(0, 0)).toBe(0);
  });

  it('should subtract two numbers correctly', () => {
    expect(subtract(3, 1)).toBe(2);
    expect(subtract(-1, 1)).toBe(-2);
    expect(subtract(0, 0)).toBe(0);
  });
});

~~~

이런 식으로 간단하게 코드를 작성하는 것만으로도 테스트케이스 실패 갯수, 정보등이 터미널에 출력됩니다.

### 8.2.3 리액트 컴포넌트 테스트 코드 작성하기

자바스크립트에서 테스트를 배워봤으니 리액트에서 테스트를 배워봅시다.

기본적으로 다음과 같은 순서로 진행됩니다.
1. 컴포넌트를 렌더링한다.
2. 필요하다면 컴포넌트에서 특정 액션을 수행한다.
3. 컴포넌트 렌더링과 2번의 액션을 통해 기대하는 결과와 실제를 비교한다.

create-react-app으로 프로젝트를 생성하면 기본적으로 테스팅 라이브러리가 포함되어 있습니다.
![image](https://github.com/Deep-Dive-React/react-study-archive/assets/42230162/e3d709a6-50cc-4f0b-a24e-fb2d49c5efc4)

위의 코드는 다음과 같이 요약할 수 있습니다.
1. App을 렌더링한다.
2. 렌더링하는 컴포넌트 내부에서 'learn react'라는 문자열을 가진 DOM요소를 찾는다.
3. expect(linkElement).toBeInTheDocument()라는 어센셜을 활용해 2번에서 찾은 요소가 document내부에 있는지 확인한다.

그렇다면 다음의 컴포넌트 테스트에 대해 알아봅시다.

#### 정적 컴포넌트
정적 컴포넌트는 사용자 입력 또는 외부 상태 변화에 의존하지 않고, 단순히 정적인 내용을 렌더링하는 컴포넌트를 말합니다. 이러한 컴포넌트는 입력 없이 렌더링되고 예측 가능한 결과를 반환합니다.

~~~
// StaticComponent.js

import React from 'react';

const StaticComponent = () => {
  return (
    <div>
      <h1>Hello, world!</h1>
      <p>This is a static component.</p>
    </div>
  );
}

export default StaticComponent;

~~~

~~~
// StaticComponent.test.js

import React from 'react';
import { render, screen } from '@testing-library/react';
import StaticComponent from './StaticComponent';

test('정적 컴포넌트 렌더링 테스트', () => {
  render(<StaticComponent />);
  
  // 정적 컴포넌트의 텍스트가 화면에 렌더링되는지 확인
  expect(screen.getByText('Hello, world!')).toBeInTheDocument();
  expect(screen.getByText('This is a static component.')).toBeInTheDocument();
});

~~~
이 테스트 코드에서는 render() 함수를 사용하여 정적 컴포넌트를 렌더링하고, screen.getByText() 함수를 사용하여 특정 텍스트가 화면에 렌더링되는지를 확인합니다.

#### 동적 컴포넌트
동적 컴포넌트는 사용자 입력 또는 외부 상태에 따라 동적으로 렌더링되는 컴포넌트를 말합니다. 이러한 컴포넌트는 상태 변화에 따라 렌더링 결과가 변할 수 있습니다.

~~~
// DynamicComponent.js

import React, { useState } from 'react';

const DynamicComponent = () => {
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount(count + 1);
  };

  return (
    <div>
      <h1>Counter: {count}</h1>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

export default DynamicComponent;

~~~
~~~
// DynamicComponent.test.js

import React from 'react';
import { render, fireEvent, screen } from '@testing-library/react';
import DynamicComponent from './DynamicComponent';

test('동적 컴포넌트 렌더링 및 이벤트 테스트', () => {
  render(<DynamicComponent />);
  
  // 초기 카운터 값 확인
  expect(screen.getByText('Counter: 0')).toBeInTheDocument();

  // 버튼 클릭 후 카운터 값 변경 확인
  fireEvent.click(screen.getByText('Increment'));
  expect(screen.getByText('Counter: 1')).toBeInTheDocument();
});

~~~
이 테스트 코드에서는 동적 컴포넌트를 렌더링하고, 버튼 클릭 이벤트를 발생시켜 카운터 값이 제대로 변경되는지를 확인합니다.


#### 비동기 이벤트가 발생하는 컴포넌트

~~~
// AsyncComponent.js

import React, { useState } from 'react';

const AsyncComponent = () => {
  const [data, setData] = useState('');

  const fetchData = async () => {
    const response = await fetch('https://api.example.com/data');
    const result = await response.json();
    setData(result.data);
  };

  return (
    <div>
      <button onClick={fetchData}>Fetch Data</button>
      <p>{data}</p>
    </div>
  );
}

export default AsyncComponent;

~~~
~~~
// AsyncComponent.test.js

import React from 'react';
import { render, fireEvent, screen, waitFor } from '@testing-library/react';
import AsyncComponent from './AsyncComponent';

// Mock fetch 함수
global.fetch = jest.fn(() =>
  Promise.resolve({
    json: () => Promise.resolve({ data: 'Mocked data' }),
  })
);

test('비동기 이벤트 발생 및 데이터 로딩 테스트', async () => {
  render(<AsyncComponent />);
  
  // 데이터가 로딩되기 전에는 'Mocked data' 텍스트가 없는지 확인
  expect(screen.queryByText('Mocked data')).toBeNull();

  // 버튼 클릭 후 데이터 로딩되는지 확인
  fireEvent.click(screen.getByText('Fetch Data'));

  // 데이터가 로딩될 때까지 기다림
  await waitFor(() => expect(screen.getByText('Mocked data')).toBeInTheDocument());
});

~~~
이 테스트 코드에서는 비동기 이벤트가 발생하는 컴포넌트를 렌더링하고, 버튼 클릭 이벤트를 발생시켜 데이터가 비동기적으로 로드되는지를 확인합니다. waitFor() 함수를 사용하여 데이터가 로딩될 때까지 대기합니다.


### 8.2.4 사용자 정의 훅 테스트하기

다른 컴포넌트들을 테스팅 해봤던 것처럼 사용자 정의 훅도 테스트를 할 수 있습니다.

react-hooks-testing-library를 활용하면 되는데요

useEffectDebugger라는 훅을 테스트해보겠습니다.

간단하게 기능을 추려서 예제를 작성해봤습니다.
~~~
// useEffectDebugger.js

import { useEffect } from 'react';

const useEffectDebugger = (effect, dependencies, dependencyName = '') => {
  useEffect(() => {
    console.log(`Effect ${dependencyName} triggered`);
    return effect();
  }, dependencies);
};

export default useEffectDebugger;

~~~

위의 코드는 useEffectDebugger라는 사용자 정의 훅을 정의합니다. 이 훅은 useEffect 훅을 디버깅하는데 사용됩니다. 각각의 의존성이 트리거될 때마다 콘솔에 해당 의존성이 트리거되었음을 표시합니다.

~~~
// useEffectDebugger.test.js

import { renderHook } from 'react-hooks-testing-library';
import useEffectDebugger from './useEffectDebugger';

test('useEffectDebugger 훅 테스트', () => {
  // 테스트할 useEffect 훅의 콜백 함수
  const effect = jest.fn();

  // 의존성 배열
  const dependencies = [1, true, 'dependency'];

  // useEffectDebugger 훅을 렌더링
  renderHook(() => useEffectDebugger(effect, dependencies, 'Test Dependency'));

  // 의존성이 변경될 때마다 effect 함수가 호출되는지 확인
  expect(effect).toHaveBeenCalledTimes(1); // 최초 렌더링 시 호출
  dependencies.forEach((dependency, index) => {
    expect(effect).toHaveBeenNthCalledWith(index + 1); // 콜백이 인덱스마다 호출됨
  });
});

~~~
위의 코드에서는 renderHook 함수를 사용하여 useEffectDebugger 훅을 렌더링하고, 전달된 의존성 배열이 변경될 때마다 효과 함수가 호출되는지 확인합니다. 

효과 함수가 정확히 예상대로 호출되는지 확인하기 위해 jest.fn()을 사용하여 효과 함수를 목(mock)으로 만들었습니다.

이 테스트 코드를 실행하면 useEffectDebugger 훅이 의존성이 변경될 때마다 효과 함수를 호출하는지 여부를 확인할 수 있습니다.

react-hooks-testing-library를 활용하면 굳이 테스트를 위한 컴포넌트를 만들지 않아도 간편하게 훅을 테스트 할 수 있으니 자주 사용해보면 좋을 것 같습니다.


### 8.2.5 테스트를 작성하기에 앞서 고려해야 할 점

테스트를 하는 이론적인 방법중 테스트 커버리지, TDD등이 있는데요

프론트엔드에서 테스트를 해야하는 범위가 너무 많다고 교재에서 말합니다.

실제로 출시된 앱도 끊임없이 버그가 발생하고 이상한 유저에 대해 속수무책인 경우도 있습니다.

따라서 무작정 테스트 코드를 작성하기 보다는 프로그램의 취약한 부분에 대해 먼저 고민해보고 개발하기를 권장합니다.

### 8.2.6 그 밖에 해볼 만한 여러가지 테스트

- 유닛테스트 : 각각의 코드나 컴포넌트가 독립적으로 분리된 환경에서 의도된 대로 정확히 작동하는지 검증하는 테스트
- 통합 테스트 : 유닛 테스트를 통과한 여러 컴포넌트가 묶여서 하나의 기능으로 정상적으로 작동하는지 확인하는 테스트 
- 엔드 투 엔드 테스트 : 흔히 E2E 테스트라 하며 실제 사용자처럼 작동하는 로봇을 활용해 앱의 전체적인 기능을 확인하는 테스트
