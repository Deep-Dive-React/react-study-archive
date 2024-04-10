# 8.2장
## 리액트 팀이 권장하는 리액트 테스트 라이브러리
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


### 8.2.3 리액트 컴포넌트 테스트 코드 작성하기


### 8.2.4 사용자 정의 훅 테스트하기


### 8.2.5 테스트를 작성하기에 앞서 고려해야 할 점


### 8.2.6 그 밖에 해볼 만한 여러가지 테스트
