# 05장 | 리액트와 상태 관리 라이브러리
## 5.1 상태 관리는 왜 필요한가?
상태:어떠한 의미를 지닌 값으로 애플리케이션의 시나리오에 따라 지속적으로 변경될 수 있는 값
* UI:각종 input, 알림창의 노출 여부 등 많은 종류의 상태 존재
* URL:브라우저에서 관리되고 있는 상태값. 
* 폼:loading, submit, disable, validation 등 모두가 상태로 관리됨
* 서버에서 가져온 값:대표적으로 API 요청이 있음

### 5.1.1 리액트 상태 관리의 역사
#### Flux 패턴의 등장
* 웹 애플리케이션이 비대해지고 상태도 많아짐에 따라 어디서 어떤 일이 일어나서 상태가 변했는지 등을 추적하고 이해하기 어려운 상황이 발생
* 이에 대한 해결책으로 단방향으로 데이터 흐름을 변경하는 것을 제안 
* 이것이 Flux 패턴의 시작
    용어의 정의
    * 액션:어떠한 작업을 처리할 액션과 그 액션 발생 시 함께 포함시킬 데이터
    * 디스패처:액션을 스토어에 보내는 역할
    * 스토어:실제 상태에 따른 값과 상태를 변경할 수 있는 메서드를 갖고 있음
    * 뷰:리액트의 컴포넌트에 해당하는 부분으로, 스토어에서 만들어진 데이터를 가져와 화면을 렌더링하는 역할
#### 시장 지배자 리덕스의 등장
Flux 구조를 구현하기 위해 만들어진 라이브러리이며, 여기에 Elm 아키텍처를 도입했다.  
##### Elm
* 웹페이지를 선언적으로 작성하기 위한 언어
* 데이터 흐름을 세 가지로 분류하고, 이를 단방향으로 강제해 웹 애플리케이션의 상태를 안정적으로 관리하고자 노력했다.
    Elm 아키텍처의 핵심
    ▪ model:애플리케이션의 상태
    ▪ view:모델을 표현하는 HTML
    ▪ update:모델을 수정하는 방식

#### Context API와 useContext
리액트 팀은 전역 상태를 하위 컴포넌트에 주입할 수 있는 새로운 Context API 출시  
props 상태로 넘겨주지 않더라도 Context API를 사용하면 원하는 곳에서 Context Provider가 주입하는 상태를 사용할 수 있게 됨  
Context API는 상태 관리가 아닌 주입을 도와주는 기능이며, 렌더링을 막아주는 기능 또한 존재하지 않으니 사용할 때 주의가 필요
#### 훅의 탄생, 그리고 React Query와 WSR
리액트는 16.8버전에서 함수 컴포넌트에 사용할 수 있는 다양한 훅 API 추가
훅 API는 기존에 무상태 컴포넌트를 선언하기 위해서만 제한적으로 사용됐던 함수 컴포넌트가 클래스 컴포넌트 이상의 인기를 구가할 수 있도록 많은 기능 제공  
가장 큰 변경점 중 하나로 꼽을 수 있는 것은 state를 매우 손쉽게 재사용 가능하도록 만들 수 있다는 것
```java
function useCounter() {
    const [count, setCount] = useState(0)

    function increase() {
        setCount((prev) => prev + 1)
    }

    return {count, increase}
}
```
React Query와 SWR 두 라이브러리는 모두 외부에서 데이터를 불러오는 fetch를 관리하는 데 특화된 라이브러리지만, API 호출에 대한 상태를 관리하기 떄문에 HTTP 요청에 특화된 상태 라이브러리이다.
```java
import React from 'react'
import useSWR from 'swr'

const fetcher = (url) => fetch(url).then((res) => res.json())
export default function App() {
    const {data, error} = useSWR (
        'https://api.github.com/repos/vercel/swr',
        fetcher,
    )
    if (error) return 'An error has occured.'
    if (!data) return 'Loading...'

    return(
        <div>
            <p>{JSON.stringify(data)}</p>
        </div>
    )
}
```
#### Recoil,Zustand,Jotai,Valtio
작은 크기의 상태를 효율적으로 관리  

### 5.1.2 정리
익숙한 것을 선택하는 것도 나쁘지 않지만 다양한 옵션을 살펴보고 비교하면서 어떤 식으로 구현하고 있는지 살펴보는 것도 많은 도움이 될 것임

## 5.2 리액트 훅으로 시작하는 상태 관리
현재 리덕스 외의 다른 상태 관리 라이브러리를 선택하는 경우도 많아지고 있다.

### 5.2.1 가장 기본적인 방법:useState와 useReducer
##### useState
useState의 등장으로 리액에서 동일한 인터페이스의 상태를 생성하고 관리하기 쉬워짐
```java
function useCounter(initCount: number = 0) {
    const [counter, setCounter]  = useState(initCount)
    function inc() {
        setCounter((prev) => prev + 1)
    }

    return {counter, inc}
}
```
* 0을 초깃갑승로 상태를 관리
* inc라는 함수를 선언해 숫자를 1씩 증가시킬 수 있게 구현
* 상태값인 counter와 inc 함수를 객체로 반환
    리액트 훅을 기반으로 만든 사용자 정의 혹은 함수 컴포넌트라면 어디서든 손쉽게 재사용 가능
##### useReducer
useReducer 또한 지역 상태를 관리할 수 있는 훅이다.
##### useState와 useReducer 기반의 사용자 지정 훅의 한계
훅을 사용할 때마다 컴포넌트별로 초기화되므로 컴포넌트에 따라 서로 다른 상태를 가질 수밖에 없다.
##### 두 컴포넌트가 동일한 counter 상태를 바라보게 하기 위한 방법
컴포넌트 밖으로 한 단계 끌어 올리는 방법이 있다.
```java
function Counter1({counter, inc}:{counter: number; inc: () => void}) {
    return (
        <>
            <h3>Counter1: {counter} </h3>
            <button Onclick={inc}>+</button>
        </>
    )
}
function Counter2({counter, inc}: {counter:number; inc: () => void}) {
    return (
        <> 
            <h3>Counter2: {counter}</h3>
            <button onClick = {inc}>+</button>
        </>
    )
}
function Parent() {
    const {counter, inc} = useCounter()
    return (
        <>
            <Counter1 counter={counter} inc={inc} />
            <Counter2 counter={counter} inc={inc} />
    )
}
```
    useState와 useReducter  
    - 지역 상태
    - 여러 컴포넌트에 걸쳐 공유하기 위해서는 컴포넌트 트리를 재설계하는 등의 수고로움 필요
### 5.2.2 지역 상태의 한계를 벗어나보자:useState의 상태를 바깥으로 분리하기
새로운 상태를 사용자의 UI에 보여주기 위해서는 리렌더링이 필요  
리렌더링은 함수 컴포넌트의 재실행, useState의 두 번째 인수 호출 등의 방식으로 발생  
#### 리렌더링을 하기 위한 방법
* useState, useReducer의 반환값 중 두 번째 인수가 어떻게든 호출된다. 설령 그것이 컴포넌트 렌더링과 관계없는 직접적인 상태를 관리하지 않아도 상관없다. 어떠한 방식으로든 두 번째 인수가 호출되면 리액트는 다시 컴포넌트를 렌더링한다.
* 부모 함수(부모 컴포넌트)가 리렌더링되거나 해당 함수(함수 컴포넌트)가 다시 실행돼야 한다. 그러나 위 경우 부모 컴포넌트가 없으며, props도 없기 때문에 일일이 Counter()를 재실행시켜야 하지만 매우 비효율적이다.
#### 함수 외부에서 상태를 참조하고 렌더링까지 자연스럽게 일어나게 하기
1. 컴포넌트 외부 어딘가에 상태를 두고 여러 컴포넌트가 같이 쓸 수 있어야 한다.
2. 외부에 있는 상태를 사용하는 컴포넌트는 상태의 변화를 알아챌 수 있어야 하고 그 상태가 변화될 때마다 리렌더링이 일어나서 컴포넌트를 최신 상태값 기준으로 렌더링해야 한다. 상태 감지는 상태를 변경시키는 컴포넌트뿐만 아니라 이 상태를 참조하는 모든 컴포넌트에서 동일하게 작동해야 한다.
3. 상태가 원시값이 아닌 객체인 경우에 그 객체에 내가 감지하지 않는 값이 변한다 하더라도 리렌더링이 발생해서는 안 된다.
     타입스크립트의 타입을 선언해 두면 직접 코드를 작성하기 앞서 만들어야 할 함수의 기본적인 타입을 정의해 두고 이야기할 수 있어 유용하다.
```
type Initializer<T> = T extends any ? T | ((prev: T) => T) : never
type Store<State> = {
    get: () => State
    set: (action: Initializer<State>) => State
    subscribe: (callback: () => void) => () => void
}
```
    Store<State> 함수 작성해보기
```java
export const createStore = <State extends unkown>(
    initialState: Initializer<State>,
): Store<State> => {
    let state = typeof initialState !== 'function' ? initialState : initialState()

    const callbacks = new Set<() => void>()
    const get = () => state
    const set = (nextState: State | ((prev: State) => State)) => {
        state = 
            typeof nextState == 'function'
            ? (nextState as (prev: State) => State)(state)
            : nextState
        callbacks.forEach((callback) => callback())

        return state
    }

    const subscribe = (callback: () => void) => {
        callbacks.add(callback)
        return () => {
            callbacks.delete(callback)
        }
    }
    return {get, set, subscribe}
}
```
##### useStor 훅으로 store의 변화 감지
```java
export const useStore = <State extends unkown>(store: Store<State>) => {
    const [state, setState] = useState<State>(() => store.get())
    useEffect(() => {
        const unsubscribe = store.subscribe(() => {
            setState(store.get())
        })
        return unsubscribe
    }, [store])

    return [state, store.set] as count
}
```

### 5.2.3 useState와 Context를 동시에 사용해 보기
Context를 활용해 해당 스토어를 하위 컴포넌트에 주입한다면 컴포넌트에서는 자신이 주입된 스토어에 대해서만 접근할 수 있다.
```java
export const CouterStoreContext = createContext<Store<CounterStore>>(
    createStore<CounterStore>({count: 0, text: 'hello' }),
)

export const CounterStoreProvider = ({
    initialState,
    children,
}: PropsWithChildren <{
    initialState: CounterStore
}>) => {
    const storeRef = useRef<Store<CounterStore>>()
    //스토어를 생성한 적이 없다면 최초에 한 번 생성한다.
    if(!storeRef.current) {
        storeRef.current = createStore(initialState)
    }

    return (
        <CounterStoreContext.Provider value={storeRef.current}>
        {children}
        </CounterStoreContext.Provider>
    )
}
```
#### useContext를 사용해 스토어에 접근할 수 있는 새로운 훅 만들기
```java
export const useCounterContextSelector = <State extends unknown>(
    selector: (state: CounterStore) => State,
) => {
    const store = useContext(CounterStoreContext)
    const subscription = useSubscription(
        useMemo(
            () => ({
                getCurrentValue: () => selector(store.get()),
                subscribe: store.subscribe,
            }),
            [store, selector],
        ),
    )

    return [subscription, store,set] as const
}
```
#### 상태 관리 라이브러리 작동 방식
* useState, useReducer가 가지고 있는 한계, 컴포넌트 내부에서만 사용할 수 있는 지역 상태라는 점을 극복하기 위해 외부 어딘가에 상태를 둔다. 이는 컴포넌트의 최상단 내지는 상태가 필요한 부모가 될 수도 있고, 혹은 격리된 자바스크립트 스코프 어딘가일 수도 있다.
* 이 외부의 상태 변경을 각자의 방식으로 감지해 컴포넌트의 렌더링을 일으킨다.

### 5.2.4 상태 관리 라이브러리 Recoil, Jai, Zustand 살펴보기
#### 페이스북이 만든 상태 관리 라이브러리 Recoil
리액트의 핵심 API : RecoilRoot, atom, useRecoilValue, useRecoilState
###### RecoilRoot
Reocil을 사용하기 위해서는 RecoilRoot를 애플리케이션의 최상단에 선언해 둬야 한다.  
```java
export default function App() {
    return <RecoilRoot>{/* some components */}</RecoilRoot>
}
```
RecoilRoot 코드에 있는 useStoreRef는 Recoil에서 생성되는 atom과 같은 상태값을 저장하는 스토어다.
* Recoil의 상태값은 RecoilRoot로 생성된 Context의 스토어에 저장된다.
* 스토어의 상태값에 접근할 수 있는 함수들이 있으며, 이 함수를 활용해 상태값에 접근하거나 상태값을 변경할 수 있다.
* 값의 변경이 발생하면 이를 참조하고 있는 하위 컴포넌트에 모두 알린다.

###### atom
atom은 상태를 나타내는 Recoil의 최소 상태 단위다.
```java
type Statement = {
    name: String
    amount: number
}

const InitialStatements: Array<Statement> = [
    {name : '과자', amount: -500},
    {name : '용돈', amount: 10000},
    {name : '네이버페이충전', amount: -5000},
]

//Atom 선언
const statementsAtom = atom<Array<Statement>>({
    key: 'statements',
    default: InitialStatements,
})
```

###### useRecoilValue
atom의 값을 읽어오는 훅
```java
function Statements() {
    const statements = useRecoilValue(statementsAtom)
    return (
        <>{/* something.. */}</>
    )
}
```
###### useRecoilState
useState와 유사하게 값을 가져오고, 이 값을 변경할 수 있는 훅

###### 정리
* <RecoilRoot/>를 선언해 하나의 스토어를 만듬
* atom 이라는 상태 단위를 <RecilRoot />에서 만든 스토어에 등록
* atom은 Recoil에서 관리하는 작은 상태 단위이며, 각 값은 고유한 값인 key를 바탕으로 구별됨
* 컴포넌트는 Recoil에서 제공하는 훅을 통해 atom의 상태 변화를 구독하고, 값이 변경되면 forceUpdate같은 기법을 통해 리렌더링을 실행해 최신 atom 값을 가져오면 됨

#### Recoil에서 영감을 받은, 그러나 조금 더 유연한 Jotai
* 리덕스처럼 하나의 큰 상태를 애플리케이션에 내려주는 방식이 아니라, 작은 단위의 상태를 위로 전파할 수 있는 구조를 취하고 있음  
* 리액트 Context의 문제점인 불필요한 리렌더링이 일어난다는 문제를 해결하고자 설계됨
* 메모이제이션이나 최적화를 거치지 않아도 리렌더링이 발생되지 않도록 설계됨

###### atom
최소 단위의 상태를 의미  
atom 하나만으로 상태를 만들 수도, 또 이에 파생된 상태를 만들 수도 있다.  

###### useAtomValue
1. Recoil과 다르게 컴포넌트 루트 레벨에서 Context가 존재하지 않아도 된다.
2. store에 atom 객체 그 자체를 키로 활용해 값 저장
3. atom의 값이 어디서 변경되더라도 useAtomValue로 값을 사용하는 쪽에서는 언제든 최신 값의 atom을 사용해 렌더링할 수 있게 된다.

###### useAtom
useState와 동일한 형태의 배열 반환
1. atom의 현재 값을 나타내는 useAtomValue 훅의 결과 반환
2. useSetATom 훅 반환 - atom을 수정할 수 있는 기능 제공

###### 사용법
상태를 선언하기 위해 atom이라는 API를 사용하여 useState와는 다르게 컴포넌트 외부에서도 선언할 수 있다.  
atom은 값 뿐만 아니라 함수를 인수로 받을 수 있어서 다른 atom의 값으로부터 파생된 atom을 만들 수도 있다.  

###### 특징
1. Recoil의 atom 개념을 도입하면서도 API가 간결하다.
별도의 문자열 키가 없이도 각 값들을 관리할 수 있다.