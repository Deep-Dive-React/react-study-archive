# 2장
# 2.1 JSX란?
JSX는 JavaScript XML의 약자로, JavaScript 코드 안에서 XML 형식으로 작성할 수 있게 해주는 문법 확장입니다. JSX를 사용하면 React 애플리케이션에서 UI를 선언적으로 작성할 수 있어서 가독성이 높고 유지보수가 쉬워집니다.

- JSXElement
JSXElement는 JSX로 작성된 요소를 의미합니다. 예를 들어, <div>나 <h1>과 같은 HTML 요소들을 JSXElement라고 합니다.
```
const element = <div>Hello, World!</div>;
```
- JSXAttributes
JSXAttributes는 JSXElement에 부여되는 속성을 나타냅니다. HTML 속성과 유사하게 JSXElement에 속성을 부여할 수 있습니다.
```
const element = <img src="logo.png" alt="Logo" />;
```
- JSXChildren
JSXChildren은 JSXElement 내부에 포함된 자식 요소들을 나타냅니다. JSXElement는 여는 태그와 닫는 태그 사이에 자식 요소를 포함할 수 있습니다.
```
const element = (
  <div>
    <h1>Title</h1>
    <p>Paragraph</p>
  </div>
);
```

- JSXStrings
JSXStrings는 JSX 내부에서 문자열을 나타냅니다. JSX 내부에서 작은 따옴표나 큰 따옴표로 감싸진 부분은 문자열로 해석됩니다.
```
const greeting = 'Hello, ';
const name = 'World';
const element = <div>{greeting + name}</div>;
```

# 2.2 가상 DOM과 리액트 파이버
리액트의 가장 큰 특징을 꼽자면 역시 가상DOM 입니다.
가상 DOM을 알기 전에 역시 DOM에 대해 알아봐야겠죠?

1. 브라우저는 사용자가 요청한 주소에서 html파일을 다운로드 합니다.
2. 브라우저의 렌더링 엔진은 html을 파싱해 DOM노드로 구성된 트리를 만듭니다.
2-1. CSS파일도 함께 다운로드 하여 CSSOM 트리를 만들게 됩니다.
3. 브라우저는 사용자의 눈에 보이는 노드만 방문합니다.
4. 3번에서 방문하지 않은 노드를 대상으로 css정보를 적용합니다.

DOM은 HTML과 스크립팅언어(Javascript)를 서로 이어주는 역할을 하고 있다고 합니다.

## 가상DOM
가상 DOM은 실제 DOM의 가벼운 복사본이라고 생각하면 됩니다. 
리액트에서는 가상 DOM을 사용하여 실제 DOM과의 비교를 효율적으로 수행하여 변경된 부분만 업데이트할 수 있습니다. 
특정 페이지에서 데이터가 변했다고 가정했을 때 리액트를 이용해 돔을 업데이트시키는 절차는 다음과 같습니다.

1. 변화가 일어난 부분을 파악하고 변화된 버전을 가상돔으로 바꾸려고 함
2. 데이터가 업데이트 되면 전체 UI를 가상돔에 리렌더링한다.
3. 가상돔끼리 비교하여 변화 전의 가상돔과 변화된 가상돔을 비교한다.
4. 바뀐 부분만 실제 돔에 적용을 함으로서 레이아웃 계산은 한 번만 이행된다.

이를 통해 성능 향상과 웹 애플리케이션의 빠른 렌더링을 달성할 수 있습니다.
하지만 항상 속도가 빠른것을 보장하는 것은 아니라고 하네요 :(

리액트 가상돔(Virtual DOM)은 실제 DOM(Document Object Model)을 추상화한 자바스크립트 객체입니다. 실제 DOM을 직접 조작하는 대신 가상돔을 사용하여 변경 사항을 추적하고 필요한 부분만 실제 DOM에 적용함으로써 성능을 향상시킵니다.

가상돔이 더 빠른 이유

1. 변경 감지
Dirty Checking: 리액트 이전에는 전체 트리를 재귀적으로 탐색하여 변경된 노드를 찾는 방식을 사용했습니다. 이 방식은 불필요한 비용이 발생할 수 있습니다.
Observable: 리액트는 state 변화를 관찰하고 렌더링 업데이트를 알리는 Observable 패턴을 사용합니다. 이 방식은 변경된 부분만 렌더링하여 효율성을 높입니다.

2. 렌더링 최적화
Virtual DOM: 가상돔은 메모리 상의 객체이기 때문에 실제 DOM을 렌더링하는 것보다 훨씬 빠릅니다.
Reconciliation: 리액트는 새로운 가상돔과 이전 가상돔을 비교하여 최소한의 변경만 실제 DOM에 적용합니다.

가상돔 객체 생성
React.createElement 함수는 컴포넌트를 가상돔 객체로 변환합니다.
각 가상돔 객체는 type, props, children 속성을 가지고 있습니다.
렌더링 과정
render 함수는 가상돔을 실제 DOM으로 변환합니다.
React Fiber는 렌더링 과정을 스케줄링하고 우선순위를 부여하여 최적화합니다.
React Fiber는 단일 연결 리스트를 사용하여 가상돔 트리를 관리합니다.

[참고]:https://velog.io/@yesbb/virtual-dom%EC%9D%98-%EC%84%B1%EB%8A%A5%EC%9D%B4-%EB%8D%94-%EC%A2%8B%EC%9D%80%EC%9D%B4%EC%9C%A0

# 2.3 클래스 컴포넌트와 함수 컴포넌트
### 클래스 컴포넌트
- ES6 클래스를 사용하여 정의
- render 메서드를 통해 UI를 정의
- state와 lifecycle 메서드를 사용 가능
- 더 복잡하고 동적인 컴포넌트에 적합
```
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
    };
  }

  render() {
    return (
      <div>
        <h1>카운트: {this.state.count}</h1>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          증가
        </button>
      </div>
    );
  }
}
```
### 함수 컴포넌트
- ES6 함수 또는 React Hook을 사용하여 정의
- props를 받아 UI를 정의
- 클래스 컴포넌트보다 간단하고 가벼움
- 재사용 가능한 작은 컴포넌트에 적합
```
const MyComponent = ({ props }) => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h1>카운트: {count}</h1>
      <button onClick={() => setCount(count + 1)}>증가</button>
    </div>
  );
};
```
![image](https://github.com/Deep-Dive-React/react-study-archive/assets/42230162/faef58bb-8834-46e2-99d7-09c7179fff5c)

# 2.4 렌더링은 어떻게 일어나는가?

1. 컴포넌트 인스턴스 생성
컴포넌트 props를 사용하여 컴포넌트 인스턴스를 생성합니다.
클래스 컴포넌트의 경우 constructor 메서드가 호출됩니다.

2. 렌더링
컴포넌트의 render 메서드를 호출하여 가상 DOM을 생성합니다.
가상 DOM은 실제 DOM으로 변환되어 화면에 표시됩니다.

3. 업데이트
state 또는 props가 변경되면 컴포넌트가 다시 렌더링됩니다.
리액트는 최적화 알고리즘을 사용하여 불필요한 렌더링을 최소화합니다.

# 2.5 컴포넌트와 함수의 무거운 연산을 기억해두는 메모이제이션

메모이제이션은 컴포넌트 또는 함수의 결과를 기억하여 동일한 입력값에 대해 동일한 결과를 반환하도록 하는 기술입니다.

장점
- 성능 향상 : 무거운 연산을 반복적으로 수행하는 것을 방지합니다.
- 메모리 사용량 감소 : 동일한 결과를 여러 번 저장하지 않아도 됩니다.

사용 방법
- React.memo HOC를 사용하여 컴포넌트를 메모이화합니다.
- useMemo Hook을 사용하여 함수 결과를 메모이화합니다.
```
const MyComponent = React.memo(() => {
  // 무거운 연산
  const result = expensiveCalculation();

  return (
    <div>
      <h1>결과: {result}</h1>
    </div>
  );
});
```
메모이제이션은 모든 상황에 적합하지 않습니다. 불필요한 렌더링을 방지하는 데 도움이 될 수 있지만, 과도하게 사용하면 오히려 성능 저하를 초래할 수 있습니다.
