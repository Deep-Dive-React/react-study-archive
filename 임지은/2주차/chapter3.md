# 03. 리액트 훅 깊게 살펴보기
## 1. useState
- 컴포넌트 내부에서 상태를 정의하고, 이 상태를 관리할 수 있게 해주는 훅
- 리액트에서 렌더링은 함수 컴포넌트의 return과 클래스 컴포넌트의 render 함수를 실행한 다음, `실행 결과를 이전의 리액트 트리와 비교해 리렌더링이 필요한 부분만 업데이트해 이뤄진다`
```javascript
function Component() {
    const [, triggerRender] = useState()
    let state = 'hello'

    function handleButtonClick() {
        state = 'hi'
        // useState 반환값의 두 번째 원소를 실행해 리액트에서 렌더링이 일어나게끔 변경
        triggerRender()
    }

    return(
        <>
            <h1>{state}</h1>
            <button onClick={handleButtonClick}>hi</button>
        </h1>
    )
}
```
위 코드에서 state의 변경된 값 렌더링되지 않음 <br>
Why?
- 리액트의 렌더링은 함수 컴포넌트에서 반환한 결과물인 return의 값을 비교해 실행
- 위 코드는 triggerRender()을 통해 렌더링 시켰지만, 렌더링될 때마다 state가 'hello'로 `초기화` 된다.

useState는 클로저에 의존해 구현돼 있을 것이다.

### 게으른 초기화
useState의 인수로 특정한 값을 넘기는 함수 넣은 것<br>
무거운 연산이 요구될 때 사용

    localStorage, sessionStorage에 대한 접근, map, filter, find와 같은 배열에 대한 접근, 초깃값 계산을 위해 함수 호출이 필요할 때

## 2. useEffect
### 클린업 함수 
- useEffect > return문 내에 정의
- 클린업 함수는 비록 새로운 값을 기반으로 렌더링 뒤에 실행되지만 이 변경된 값을 읽는 것이 아니라 함수가 정의됐을 당시에 선언됐던 
**이전 값**을 보고 실행된다.
```javascript
import {useState, useEffect} from 'react'

export default function App() {
    const [counter, setCounter] = useState(0)

    function handleClick() {
        setCounter((prev) => prev + 1)
    }

    useEffect(()=>{
        function addMouseEvent() {
            console.log(counter)
        }

        window.addEventListener('click', addMouseEvent)

        // 클린업 함수
        return () => {
            console.log('클린업 함수 실행!', counter)
            // 핸들러 무한히 추가되는 것 방지
            window.removeEventListenr('click', addMouseEvent)
        }
    }, [counter])

    return (
        <>
            <h1>{counter}</h1>
            <button onClick={handleClick}>+</button>
        </>
    )
}
```
```bash
클린업 함수 실행! 0 <- 이전 값 (클린업 함수 부분)
1 <- 변경된 값

클린업 함수 실행! 1
2

...
```
- 익명 함수 (함수명 없음)
```javascript
useEffect(()=>{
    logging(user.id)
}, [user.id]
)
```
- 기명 함수 (함수명 명시)
```javascript
useEffect(
    function logActiveUser() {
        logging(user.id)
    }, [user.id]
)
```

## 3. useMemo
- 비용이 큰 연산에 대한 결과를 저장(메모이제이션)해 두고, 이 저장된 값을 반환하는 훅
- 최적화
```javascript
useMemo(()=>{값을 반환하는 생성함수=콜백함수}, {해당 함수가 의존하는 값의 배열})
```
- 렌더링 발생 시, <br>의존성 배열의 값이 변경되지 않으면 => 함수 재실행 X, 이전에 기억해둔 값 반환<br>
의존성 배열의 값이 변경됐다면 => 첫번째 인수의 함수를 실행한 후에 그 값을 반환하고 그 값을 다시 기억

- 만약 빈 배열을 넣는다면 useEffect와 마찬가지로 마운트 될 때에만 값을 계산하고 그 이후론 계속 memoization된 값을 꺼내와 사용한다.
#### 예제
```javascript
import { useMemo, useEffect, useState } from "react";

function App() {
  const [number, setNumber] = useState(1);
  const [isKorea, setIsKorea] = useState(true);

  // 1번 location
  // const location = {
  //   country: isKorea ? "한국" : "일본"
  // };

  // 2번 location
  const location = useMemo(() => {
    return {
      country: isKorea ? '한국' : '일본'
    }
  }, [isKorea])

  useEffect(() => {
    console.log("useEffect 호출!");
  }, [location]);

  return (
    <header className="App-header">
      <h2>하루에 몇 끼 먹어요?</h2>
      <input
        type="number"
        value={number}
        onChange={(e) => setNumber(e.target.value)}
      />
      <hr />

      <h2>내가 있는 나라는?</h2>
      <p>나라: {location.country}</p>
      <button onClick={() => setIsKorea(!isKorea)}>Update</button>
    </header>
  );
}

export default App;
```
    1번 location (useMemo 사용 안함)
    input값을 변경하면 계속해서 useEffect가 호출된다.
    useEffect는 location이 변경될 때 호출되도록 했는데 why?
        input > setNumber을 통해 컴포넌트가 재렌더링되고, 이때 location 객체의 주소값이 새로 부여되기 때문에 useEffect는 location이 변경되었다고 인지한다. 
    
    2번 location (useMemo 사용)
    isKorea가 변경될 때에만!!

## 4. useCallback
- 인수로 넘겨받은 콜백 자체를 기억한다.
- useCallback은 특정 함수를 새로 만들지 않고 다시 재사용한다.
- useMemo와 작동방식이 동일한데, useMemo는 "값"을 기억해주는 것이고, useCallback은 "함수"를 기억해주는 것이다!

useMemo와 useCallback의 궁극적인 목표는 불필요한 렌더링을 줄이는 것이다! => 성능개선이 목적

## 5. useRef
```javascript
// 생성
const 변수명 = useRef(초기값)

// 반환요소에 접근
<input ref={변수명} />
```

- 변화는 감지하지만 **렌더링을 발생시키지 않는다**
=> 성능향상 (state는 변화시, 재렌더링됨)
- 변수명에 접근하려면 `변수명.current`
- DOM 요소에 손쉽게 접근 (ref 속성 사용)
- 전생애주기를 통해 유지 (언마운트 되기 전까지는 값을 계속 기억함) 
=> ref는 렌더링 이후에도 값이 유지되지만, 변수는 렌더링되면 초기화된다.

## 6. useContext
- 일일이 하위 컴포넌트로 props 전달해야 하는 번거로움 해결 => props drilling 해결

- `createContext`, `useContext`
```javascript
import React,{createContext, useContext} from 'react'

const TextContext = createContext("Hello world")

function End(){
    const text = useContext(TextContext);
    return (
        <div>
            {text}
        </div>
    )
}

function Middle(){
    return (
        <div>
            <End/>
        </div>
    )
}

export default function App() {
    return (
        <div>
            <TextContext.Provider value={"Hello world"}>
                <Middle/>
            </TextContext.Provider>
        </div>
    )
}
```

- 상위 컴포넌트, (상위 컴포넌트로 감싸져 있는) 하위 컴포넌트가 있을 때, 상위 컴포넌트가 재렌더링되면 자동으로 하위 컴포넌트들 모두 재렌더링된다.<br>
=> 이러한 현상을 막으려면 `React.memo`를 써야 한다.<br>
=> memo는 props 변화가 없으면 리렌더링되지 않고 같은 결과물을 반환한다.

## 7. useReducer
- useState와 비슷 (클로저를 활용해 값을 가둬서 state를 관리한다.)
- state 값을 변경하는 시나리오를 **제한적**으로 두고 이에 대한 변경을 빠르게 확인할 수 있게끔 한다. 

```javascript
const [state, dispatch] = useReducer(reducer, initialState, init);
```
**반환**
- state : 컴포넌트에서 사용할 state(상태)
- dispatch : reducer 함수를 실행시키며, 컴포넌트 내에서 state의 업데이트를 일으키기 위해서 사용하는 함수
```javascript
dispatcher({type: 'up'})
```


**parameter**
- reducer : 컴포넌트 외부에서 state를 업데이트하는 로직을 담당하는 함수. 현재의 state와 action 객체를 인자로 받아서, 기존의 state를 대체할 새로운 state를 반환하는 함수
```javascript
type Action = { type: 'up' | 'down' | 'reset'; payload?: State}

function reducer(state: State, action: Action) : State {
    switch (action.type) {
        case 'up':
            return {count: state.count + 1} 
        case 'down':
            return {count: state.count - 1 > 0 ? state.count-1:0}
        case 'reset':
            return (action.payload || {count: 0})
        default:
            throw new Error(`Unexpected action type ${action.type}`)
    }
}
```
위와 같이 state를 변경해주는 action을 'up', 'down', 'reset'으로 제한한다.

- initialState : 초기 state
- init : 초기함수