# 03장 | 리액트 훅 깊게 살펴보기
## 3.1 리액트의 모든 훅 파헤치기
### 3.1.1 useState
useState 함수는 함수 컴포넌트 내부에서 상태를 정의하고 상태를 관리할 수 있게 해주는 훅이다.

##### useState 구현 살펴보기
useState 훅의 기본 사용법
```java
import {useState} from 'react'
const [state, setState] = useState(initialState)
```
useState 훅의 반환 값은 배열이며, 배열의 첫 번째 원소로 state 값 자체를 사용할 수 있고, 두 번째 원소인 setState 함수를 사용해 해당 state의 값을 변경할 수 있다.
함수의 실행이 끝났음에도 함수가 선언된 환경을 기억할 수 있는 방법은 클로저다.
클로저를 사용함으로써 외부에 해당 값을 노출시키지 않고 오직 리액트에서만 쓸 수 있었고, 함수 컴포넌트가 매번 실행되더라도 useState에서 이전의 값을 정확하게 꺼내 쓸 수 있게 됐다.

##### 개으른 초기화
useState에 변수 대신 함수를 넘기는 것
리액트에서는 렌더링이 실행될 때마다 함수 컴포넌트의 함수와 useState의 값이 재실행된다는 점을 명심
useState 내부에 함수를 넣으면 최초 렌더링 이후에는 실행되지 않고 최초의 state 값을 넣을 때만 실행됨
    게으른 최적화는 무거운 연산이 요구될 때 사용하면 좋다.

### 3.1.2 useEffect
애플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 내는 메커니즘
state와 props의 변화 속에서 일어나는 렌더링과정에서 실행되는 부수 효과 함수
부수 효과가 어떤 상태값과 함께 실행되는지 살펴보는 것이 중요하다.
```java
function Component() {
    // ..
    useEffect(() => {
        //do something
    }, [props, state])
    // ..
}
```
첫 번째 인수 : 부수 효과가 포함된 함수
두 번째 인수 : 의존성 배열 전달

##### 클린업 함수의 목적
클린업 함수는 이전 state를 참조해 실행된다.

##### 의존성 배열
의존성 배열은 보통 빈 배열을 두거나, 아예 아무런 값도 넘기지 않거나, 혹은 사용자가 직접 원하는 값을 넣어줄 수 있다. 
보통 컴포넌트가 렌더링됐는지 확인하기 위한 방법으로 사용된다.
useEffect는 컴포넌트의 렌더링이 완료된 이후에 실행된다.

##### useEffect의 구현
핵심은 의존성 배열의 이전 값과 현재 값의 얕은 비교다.
리액트는 값을 비교할 때 Object.is를 기반으로 하는 얕은 비교를 수행한다.
의존성 배열과 현재 의존성 배열의 값에 하나라도 변경 사항이 있다면 callback으로 선언한 부수효과를 실행하는 것이 useEffect의 본질이다.

##### useEffect 주의점
###### eslint-disable-line react-hooks/exhaustive-deps 주석은 최대한 자제하라
useEffect는 반드시 의존성 배열로 전달한 값의 변경에 의해 실행돼야 하는 훅이다. 그러나 의존성 배열을 넘기지 않은 채 콜백 함수 내부에서 특정 값을 사용한다는 것은, 부수 효과가 실제로 관찰해서 실행돼야 하는 값과는 별개로 작동한다는 의미한다.
useEffect에 빈 배열을 넘기기 전에는 정말로 useEffect의 부수 효과가 컴포넌트의 상태와 별개로 작동해야만 하는지, 혹은 여기서 호출하는 게 최선인지 한 번 더 검토해 봐야 한다.

##### useEffect의 첫 번째 인수에 함수명을 부여하라
적절한 이름을 붙이면 해당 useEffect의 목적을 파악하기 쉬워진다.

##### 거대한 useEffect를 만들지 마라
##### 불필요한 외부 함수를 만들지 마라
useEffect 밖에서 함수를 선언하면 불필요한 코드가 많아지고 가독성이 떨어진다.

### 3.1.3 useMemo
비용이 큰 연산에 대한 결과를 저장(메모제이션)해 두고 저장된 값을 반환하는 훅
첫 번째 인수로는 어떠한 값을 반환하는 생성 함수를, 두 번째 인수로는 해당 함수가 의존하는 값의 배열을 전달한다.
메모이제이션은 값뿐만 아니라 컴포넌트도 가능하다(그러나 이 때는 React.memo를 쓰는 것이 더 현명하다).
메모이제이션을 활용하면 무거운 연산을 다시 수행하는 것을 막을 수 있다.
```java
import {useMemo} from 'react'
const memoizedValue = useMemo(() => expensiveComputation(a, b), [a, b])
```

### 3.1.4 useCallback
특정 함수를 새로 만들지 않고 다시 재사용한다.
useMemo : 값의 메모미제이션을 위해 사용
useCallback : 함수의 메모이제이션을 위해 사용
useCallback의 첫 번째 인수로 함수를, 두 번째 인수로 의존성 배열을 집어 넣으면 의존성 배열이 변경되지 않는 한 함수를 재생성하지 않는다.

### 3.1.5 useRef
컴포넌트 내부에서 렌더링이 일어나도 변경 가능한 상태값을 저장한다.
useRef의 최초 기본값은 return 문에 정의해 둔 DOM이 아니고 useRef()로 넘겨받은 인수다.
개발자가 원하는 시점의 값을 렌더링에 영향을 미치지 않고 보관해 두고 싶다면 useRef를 사용하는 것이 좋다.
    [useRef와 useState의 차이점]
    ▪ useRef는 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경할 수 있다.
    ▪ useRef는 그 값이 변하더라도 렌더링을 발생시키지 않는다.
```java
function RefComponent() {
        const count = useRef(0)
        function handleClick() {
            count.current += 1
        }
        return <button onClick={handleClick}>{count.current}</button>
}
```

### 3.1.6 useContext
##### context란?
props 내려주기를 극복하기 위해 등장한 개념
명시적인 props 전달 없이도 선언한 하위 컴포넌트 모두에서 자유롭게 원하는 값을 사용할 수 있다.
useContext를 사용하면 상위 컴포넌트에서 선언된 <Context.provider />에서 제공한 값을 사용할 수 있다.
여러 개의 Provider가 있다면 가장 가까운 Provider의 값을 가져오게 된다.
##### useContext를 사용할 때 주의할 점
useContext를 함수 컴포넌트 내부에서 사용할 때는 항상 컴포넌트 재활용이 어려워진다.
또한 useContext 자체로는 렌더링 최적화에 아무런 도음이 되지 않는다.

### 3.1.7 useReducer
useState와 비슷한 형태를 띠지만 좀 더 복잡한 상태값을 미리 정의해 놓은 시나리오에 따라 관리할 수 있다.
    [반환값 ]
    state: 현재 useReducer가 가지고 있는 값
    dispatcher: state를 업데이트하는 함수. state를 변경할 수 있는 action을 넘겨준다.
    [2개에서 3개의 인수를 필요로 함]
    reducer: useReducer의 기본 action을 정의하는 함수
    initialState: 두 번째 인수, useReducer의 초깃값
    init: 초깃값을 지연해서 생성시키고 싶을 때 사용하는 함수
##### useReducer의 목적
state 값을 변경하는 시나리오를 제한적으로 두고 이에 대한 변경을 빠르게 확인할 수 있게끔 하는 것

### 3.1.8 useImperativeHandle
##### forwardRef 살펴보기
ref는 useRef에서 반환한 객체로, 리액트 컴포넌트의 props인 ref에 넣어 HTMLElement에 접근하는 용도로 사용된다.
ref도 key와 마찬가지로 컴포넌트의 props로 사용할 수 있는 예약어로써, 별도로 선언돼지 않아도 사용할 수 있다.
    [forwardRef의 탄생 배경]
    ref를 전달하는 데 있어서 일관성을 제공하기 위해서
    네이밍의 자유가 주어진 props보다 더 확실하게 ref를 전달할 것임을 예측할 수 있다.
    사용하는 쪽에서도 안정적으로 받아서 사용할 수 있다.
##### useImperativeHandle이란?
부모에게서 넘겨받은 ref를 원하는 대로 수정할 수 있는 훅이다.

### 3.1.9 useLayoutEffect
형태나 사용 예제가 useEffect와 동일하나, 모든 DOM의 변경(렌더링) 후에 콜백 함수 실행이 동기적으로 발생한다.
동기적으로 발생한다 = 리액트가 useLayoutEffect의 실행이 종료될 때까지 기다린 다음에 화면을 그린다
useLayoutEffect는 브라우저에 변경 사항이 반영되기 전에 실행된다.
useEffect는 브라우저에 변경 사항이 반영되고 실행된다.
따라서 항상 useLayoutEffect가 useEffect보다 먼저 실행된다.
[실행 순서]
1. 리액트가 DOM을 업데이트
2. useLayoutEffect 실행
3. 브라우저에 변경 사항을 반영
4. useEffect 실행
##### 언제 사용할까 
DOM은 계산됐지만 이것이 화면에 반영되기 전에 하고 싶은 작업이 있을 때와 같이 필요할 때만 사용하는 것이 좋다.

### 3.1.10 useDebugValue
디버깅하고 싶은 정보를 이 훅에다 사용하면 리액트 개발자 도구에서 볼 수 있다.
```java
function useDate() {
    const date = new Date()
    useDebugValue(date, (date) => `현재 시간: ${date.toISOString()}`)
    return date
}
```
### 3.1.11 훅의 규칙
1. 최상위에서만 훅을 호출해야 한다. 반복문이나 조건문, 중첩된 함수 내에서 훅을 실행할 수 없다. 
조건문이 필요하다면 반드시 훅 내부에서 수행해야 한다.
2. 훅을 호출할 수 있는 것은 리액트 함수 컴포넌트, 혹은 사용자 정의 훅의 두 가지 경우뿐이다. 일반 자바스크립트 함수에서는 훅을 사용할 수 없다.

### 3.1.12 정리

## 3.2 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?
서로 다른 컴포넌트 내부에서 같은 로직을 공유하는 방법으로 사용자 정의 훅과 고차 컴포넌트가 있다.
    서로 다른 컴포넌트 내부에서 같은 로직을 공유한다 ?   
    비슷한 동작을 하는 코드를 반복해서 작성하지 않고, 여러 컴포넌트가 그 코드를 공유하여 사용할 수 있게 되는 것을 의미한다.

### 3.2.1 사용자 정의 훅
내부에 useState와 useEffect 등을 가지고 자신만의 원하는 훅을 만드는 기법   
고차 컴포넌트와 달리 사용자 정의 훅은 리액트에서만 사용할 수 있다.   
반드시 use로 시작하는 함수를 만들어야 한다는 규칙이 있다.   

### 3.2.2 고차 컴포넌트
컴포넌트 자체의 로직을 재사용하기 위한 방법   
고차함수의 일종으로 리액트가 아니더라도 자바스크립트 환경에서 널리 쓰일 수 있다.   
리액트에서 가장 유명한 고차 컴포넌트는 React.memo다.   

##### React.memo란?
React.memo는 렌더링하기에 앞서 props를 비교해 이전과 props가 같다면 렌더링 자체를 생략하고 이전에 기억해 둔 컴포넌트를 반환하여 불필요한 렌더링작업을 생략할 수 있다.   
useMemo를 해도 동일하게 메모이제이션 할 수 있으나, 코드를 작성하는 입장에서 혼선을 빚을 수 있으므로 memo를 사용하는 편이 좋다.   
##### 고차함수 만들어보기
    고차함수 : 함수를 인수로 받거나 결과를 변환하는 함수
```java
//고차함수인 Array.prototype.map 사용 예제
const list = [1, 2, 3]
const doubledList = list.map((item) => item*2)
```
(item) => item * 2 --> 함수를 인수로 받음

```java
function add(a) {
    return function(b) {
        return a+b
    }
}
const result = add(1) 
const result2 = result(2) // 3 반환
```
1. result에는 add(1)을 호출했을 때 반환되는 함수가 할당됨   
여기서 할당된 함수는 현재 a에 1이 할당되어 b를 받아 a+b를 반환하는 함수
2. result2는 add(1)을 호출하여 반환된 함수를 다시 호출하여 얻은 결과값이다.   
이처럼 고차 함수를 활용하면 함수를 인수로 받거나 새로운 함수를 반환해 새로운 결과를 만들어 낼 수 있다.

##### 고차 함수를 활용한 리액트 고차 컴포넌트 만들어보기
```java
interface LoginProps {
    loginRequired?: boolean
}

//withLoginComponent는 함수 컴포넌트를 인수로 받고 컴포넌트를 반환하는 고차 컴포넌트
function withLoginComponent<T>(Component: ComponentType<T>) {
    return function (props: T & LoginProps) {
        const {loginRequired, ...restProps} = props
        if(loginRequired) {
            return <>로그인이 필요합니다.</>
        }

        return <Component {...(restProps as T)} />
    }
}

//Component라는 함수 컴포넌트 자체를 withLoginComponent라 불리는 고차 컴포넌트로 감싸줬다.
const Component = withLoginComponent((props: {value : string }) => {
    return <h3>{props.value}</h3>
})

export default function App() {
    //로그인 관련정보를 가져온다.
    const isLogin = true
    return <Component value = "text" loginRequired={isLogin} />
}
```
이 컴포넌트는 loginRequired가 있거나 true이면 "로그인이 필요합니다"가 출력되고,
loginRequired가 없거나 false이면 <h3>text</h3>가 출력된다.

##### 고차컴포넌트 vs 사용자 정의 훅
고차 컴포넌트는 컴포넌트 전체를 감쌀 수 있다는 점에서 사용자 정의 훅보다 영향력이 크다.   
사용자 정의 훅 : 단순히 값을 반환하거나 부수효과를 실행   
고차 컴포넌트 : 컴포넌트의 결과물에 영향을 미칠 수 있는 다른 공통된 작업을 처리할 수 있다.

##### 고차컴포넌트 사용 시 주의할 점
1. with로 시작하는 이름을 사용해야 한다. 
2. 부수 효과를 최소화해야한다(props를 임의로 수정/추가/삭제하는 일은 없어야 한다).
3. 여러 개의 고차 컴포넌트로 컴포넌트를 감쌀 경우 복잡성이 커진다.

### 3.2.3 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?
두 가지 모두 중복된 로직을 분리해 컴포넌트의 크기를 줄이고 가독성을 향상시키는 데 도움이 된다.

#### 사용자 정의 훅이 필요한 경우
useEffect, useState와 같이 리액트에서 제공하는 훅으로만 공통 로직을 분리할 수 있는 경우   
사용자 정의 훅 그자체로는 렌더링에 영향을 미치지 않기 때문에 개발자가 원하는 방향으로 훅을 사용할 수 있다.
```java
function HookComponent() {
    const {loggedIn} = useLogin()
    useEffect(() => {
        if(!loggedIn) {
            // ..
        }
    }, [loggedIn])
}
```
로그인 정보를 가지고 있는 훅인 useLogin은 단순히 loggedIn에 대한 값만 제공하고 컴포넌트를 사용하는 쪽에서 원하는 대로 사용 가능하다.   

#### 고차 컴포넌트를 사용해야 하는 경우
렌더링 결과물에도 영향을 미치는 공통 로직이라면 고차 컴포넌트를 사용하는 것이 좋다.   

##### 예시
1. 로그인되지 않은 어떤 사용자가 컴포넌트에 접근하려 할 때 애플리케이션 관점에서 컴포넌트를 감추고 로그인을 요구하는 공통 컴포넌트를 노출하려는 경우
2. 특정 에러가 발생했을 때 현재 컴포넌트 대신 에러가 발생했음을 알릴 수 있는 컴포넌트를 노출하는 경우
```java
function HookComponent() {
    const {loggedIn} = useLogin()

    if(!loggedIn) {
        return <LoginComponent />
    }

    return <>안녕하세요.</>
}

const HOCComponent = withLoginComponent(() => {
    return <>안녕하세요.</>
})
```

### 3.2.4 정리
사용자 정의 훅과 고차 컴포넌트 중 어떤 것을 사용할 지 고민될 때가 있을 것이다.   
공통화하고 싶은 작업이 무엇인지, 이를 처리해야 하는 상황을 살펴보고 적절한 방법을 찾는다면 애플리케이션을 더 효율적으로 개발할 수 있을 것이다.   