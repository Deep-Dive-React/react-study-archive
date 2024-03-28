d# 05. 리액트와 상태관리 라이브러리
## 1. 상태관리는 왜 필요한가?
### 리액트 상태관리의 역사
    과거 : 양방향 데이터 바인딩 (뷰 <-> 모델)
    현재 : 단방향 데이터 흐름으로 변경

#### 단방향 데이터 흐름 최초 채택, Flux
![flux](https://miro.medium.com/v2/resize:fit:4800/format:webp/0*ZTe7dIoLlUFYFbML.png)
- action : 작업 처리할 액션과 액션 발생할때 포함할 데이터 (액션타입, 데이터를 정의해 디스패처로 보낸다)
- dispatcher : 액션을 스토어로 보낸다, 콜백 함수 형태
- store : 실제 상태 값, 상태를 변경할 수 있는 매서드
- view : 리액트의 컴포넌트, 스토어에서 만들어진 데이터를 가져와 화면에 렌더링하는 역할

위의 사진처럼 뷰에서 액션을 호출하는 구조

#### React Auery, SWR
- 외부에서 데이터를 불러오는 fetch를 관리하는데 특화된 라이브러리
- HTTP 요청에 특화된 상태관리 라이브러리

## 2. 리액트 훅으로 시작하는 상태관리
### 가장 기본적인 방법 : useState와 useReducer
사용자 지정 훅을 활용한 지역 상태 관리
```javascript
function useCounter(initCount: number = 0) {
    const [counter, setCounter] = useState(initCount)

    function inc() {
        setCounter((prev)=>prev+1)
    }

    return {counter, inc}
}

function Counter1() {
    const {counter, inc} = useCounter()

    return (
        <>
            <h3>Counter1 : {counter}</h3>
            <button onClick={inc}>+</button>
        </>
    )

function Counter2() {
    const {counter, inc} = useCounter()

    return (
        <>
            <h3>Counter2 : {counter}</h3>
            <button onClick={inc}>+</button>
        </>
    )
}
```
이렇게 useCounter 훅을 만들면, 다른 컴포넌트에서 useCounter을 불러와 사용할 수 있다. (매번 count + 1하는 로직을 짜지 않아도 됨)

하지만, 이 방법의 한계점이 존재하는데..<br>
Counter1에서 불러온 counter와, Counter2에서 불러온 counter가 각각 다른 상태를 가지게 된다. useState를 기반으로 한 상태를 `지역상태` 라고 하는데, 이 지역상태는 해당 컴포넌트 내에서만 유효하다는 한계가 있다.

이 현상 해결 방법!<br>
=> 상태를 컴포넌트 밖으로 한 단계 끌어올리기<br>
=> 하지만 매번 컴포넌트 트리(상하계층)를 고려하고 구분하는 작업은 컴포넌트 간의 관계가 복잡해지면 많은 귀찮음이 동반된다.

### 상태 관리 라이브러리 Recoil, Jotai, Zustand 살펴보기
useState와 useReducer을 활용한 상태관리는 지역상태라는 한계점이 존재하는데 이를 해결하기 위해서는 공통적으로 사용하는 방법이 있다.<br>
1. 컴포넌트의 최상단에 store 이라는 공간을 만들고,
2. 컴포넌트에서 필요한 상태가 있으면 store에서 가져와 사용하면 된다.
3. store은 컴포넌트들이 사용하면서, 상태가 변화되는 순간을 감지해 컴포넌트의 렌더링을 일으킨다.

![store](https://velog.velcdn.com/images%2Fzzangzzong%2Fpost%2Ff248cea4-1f2d-48dd-9cfa-fe9afd116771%2Fimage.png)

#### 1) Recoil (페이스북이 만든 상태 관리 라이브러리)
    리액트의 핵심 API
    1. RecoilRoot
        => 애플리케이션의 최상단에 위치
        => 역할 : Recoil에서 생성되는 상태값을 저장하기 위한 `스토어` 생성

        RecoilRoot 내부 코드 살펴보기
        -> RecoilRoot는 useContext를 이용해 스토어의 상태를 하위 컴포넌트들이 접근할 수 있게 해준다.
        -> 스토어의 상태값에 접근할 수 있는 함수들이 있으며, 이 함수를 활용해 상태값에 접근하거나 변경할 수 있다.
        -> 값의 변경이 발생하면 이를 참조하고 있는 하위 컴포넌트에 콜백을 실행해 모두 알린다.

    2. atom
        => 상태를 나타내는 Reocil의 최소 상태 단위
        => key 값을 필수로 가진다. 다른 atom과 구별하는 식별자 역할
        => default는 atom의 초깃값

    3. useRecoilValue
        => 하위 컴포넌트에서 atom의 `값`을 읽어오는 훅
        => 상태변경은 일으키지 않음 (렌더링도 발생 X)

    4. useRecoilState
        => useState와 유사하게 값을 가져오고, 값 변경도 가능
        => 리렌더링 발생

#### 2) Jotai (Reocil에서 영감을 받아 개발)
    Jotail의 핵심 API
    1. atom
        => 최소 단위의 상태
        => Reocil과 다른 점
            1) atom 하나로 파생된 상태까지 만들 수 있다.
            2) key를 명시할 필요 없다. 
    
    2. useAtomValue
        => atom의 `값` 읽어오기

    3. useAtom
        => useState과 비슷하게 사용 (Recoil의 useRecoilState과 비슷)
---
### Recoil vs Jotail 코드로 비교하기
1. atom 정의
```javascript
// recoil의 atom 생성
const counterState = atom({
    key: 'counterState',
    default : 0,
})

// recoil의 atom 기반으로 또 다른 상태 생성 방법
// selector 이용
 const inBiggerThan10 = selector({
    key: 'above10State',
    get: ({get}) => {
        return get(counterState) > 10
    },
 })
        
// Jotai의 atom 생성
const counterState = atom(0)

// Jotai의 파생 상태 생성 방법
const isBiggerThan10 = atom((get) => get(counterState) > 10)
```
    다른점
    1. atom의 key 정의 유무
    recoil > key를 정의해야 한다.
    Jotai > 객체의 참조를 WeakMap에 보관해 객체 자체가 변경되지 않는 한 별도의 키가 없이도 객체의 참조를 통해 값 관리 가능

    2. atom에서 파생된 값 생성
    recoil > selector 필요
    Jotai > atom만으로 생성 가능
---
#### 3) Zustand (작고 빠르며 확장에도 유연)
Zustand는 Redux에 영감을 받아 만들어졌다.<br>
Recoil & Jotai : atom이라는 최소 단위 상태를 관리<br>
Zustand : 하나의 `스토어를 중앙 집중형`으로 활용해 스토어 내부에서 상태 관리
    
    즉, Zustand는 스토어 내부에서 상태 정의, get, set을 모두 한 번에 정의한다.
    (Recoil에서는 atom으로 상태 정의, useRecoilValue로 상태 get, useRecoilState로 상태 set 하지만 Zustand는 스토어 내에서 이 모든걸 정의)

```javascript
import {create} from 'zustand'

// 스토어 정의 (정의, set)
const useCounterStore = create((set) => ({
    count : 1,
    inc : () => set((state) => ({count : state.count + 1})),
    dec : () => set((state) => ({count : state.count - 1})),
}))

function Counter() {
    // 스토어에 있는 값 중에서 필요한 것 가져와 씀
    const {count, inc, dec} = useCounterStore()
    return(
        <div class="counter">
            <span>{count}</span>
            <button onClick={inc}>up</button>
            <button onClick={dec}>down</button>
        </div>
    )
}
```