# 08장 | 좋은 리액트 코드 작성을 위한 환경 구축하기
ESLint를 활용한 정적 분석과 리액트 테스트 라이브러리를 충분히 활용하면 여러 가지 문제를 점검하고 확인할 수 있다.
## 8.1 ESLint를 활용한 정적 코드 분석
정적 코드 분석:코드의 실행과 별개로 코드 스멜(잠재적으로 버그를 야기할 수 있는 코드)을 찾아내어 문제의 소지가 있는 코드를 사전에 수정하는 것  
가장 많이 사용되는 정적 코드 분석 도구가 ESLint다.
### 8.1.1 ESLint 살펴보기
ESLint는 자바스크립트 코드를 정적 분석해 잠재적인 문제를 발견하고 수정까지 도와주는 도구다.
##### ESLint는 어떻게 코드를 분석할까?
1. 자바스크립트 코드를 문자열로 읽는다.
2. 자바스크립트 코드를 분석할 수 있는 파서로 코드를 구조화한다.
3. 2번에서 구조화한 트리를 AST(Abstract Syntax Tree)라 하며 이 구조화된 트리를 기준으로 각종 규칙과 대조한다.
4. 규칙과 대조했을 때 이를 위반한 코드를 알리거나 수정한다.
여기서 자바스크립트를 분석하는 파서에는 여러 가지가 있는데, ESLint는 기본값으로 espree를 사용한다.
```java
function hello(str) {}
```
이 코드를 espree로 분석하면 다음과 같이 JSON 형태로 구조화된 결과를 얻을 수 있다.
```java
{
    "type":"Program",
    "start":0,
    "end":22,
    "range": [0, 22],
    "body": [
        {
            "type":"FunctionDeclaration",
            "start":0,
            "end":22,
            "range":[0,22],
            "id": {
                "type":"Identifier",
                "start":9,
                "end":14,
                "range":[9, 14],
                "name": "hello"
            },
            "expression":false,
            "generation":false,
            "async":false,
            "params" [
                {
                    "type":"Identifier",
                    "start":15,
                    "end":18,
                    "range": [15, 18],
                    "name":"str"
                }
            ],
            "body":{
                "type":"BlockStatement",
                "start":20,
                "end":22,
                "range":[20, 22],
                "body": []
            }
        }
    ],
    "sourceType"
}
```
다음과 같이 espree 같은 코드 분석 도구는 아주 세세한 정보를 분석해 알려준다.  
* ESLint 규칙:ESLint가 espree로 코드를 분석한 결과를 바탕으로, 어떤 코드가 잘못된 코드이며 어떻게 수정해야 할지 정하는 것
* plugins:특정한 규칙의 모음
###### no-debugger 규칙 예시
debugger는 코드 개발 과정에서만 사용해야 하는 구문으로, 프로덕션 애플리케이션에는 절대 존재해서는 안 되는 구문이다.  
ESLint를 이용해 사용을 금지하는 규칙을 만들면 다음과 같다.
```java
module.exports = {
    meta: {
        type:'problem',
        docs: {
            description:'Disallow the use of `debugger`',
            recommended: true,
            url:'https://eslint.org/docs/rules/no-debugger',
        },
        fixable:null,
        schema:[],
        messages: {
            unexpected:"Unexpected `debugger` statement.",
        },
    },
    create(context) {
        return {
            DebuggerStatement(node) {
                context.report({
                        node,
                        messageId: 'unexptected',
                })
            }
        }
    }
}
```
* meta:해당 규칙과 관련된 메타 정보를 나타냄
* message:규칙을 어겼을 때 반환하는 경고 문구
* docs:문서화에 필요한 정보
* fixable:수정 가능한지 여부를 나타냄
* create:espree로 만들어진 AST 트리를 실제로 순회해, 여기서 선언한 특정 조건을 만족하는 코드를 찾고, 이러한 작업을 코드 전체에서 반복. 즉, 실제로 코드에서 문제점을 확인하는 곳이다.
다음으로 ESLint를 설치하고 사용하려면 어떤 것들을 준비해야 하는지 살펴보자.
### 8.1.1 eslint-plugin과 eslint-config
##### eslint-plugin
앞서 언급했던 규칙을 모아놓은 패키지  
* eslint-plugin-import:다른 모듈을 불러오는 import와 관련된 다양한 규칙 제공
* eslint-plugin-react:JSX 배열에서 key가 누락된 경우 경고를 보여줌
##### eslint-config
eslint-plugin을 한데 묶어서 완벽하게 한 세트로 제공하는 패키지  
개발 환경에 맞는 규칙과 정의를 일괄적으로 적용하고 싶을 때 사용
###### eslint-plugin과 eslint-config의 네이밍 규칙
1. eslint-plugin, eslint-config라는 접두사를 준수해야 함
2. 반드시 한 단어로 구성해야 함. ex. eslint-plugin-naber
지금 바로 사용할 수 있는 라이브러리에는 대표적으로 무엇이 있는지 살펴보자.
###### eslint-config-airbnb
에어비앤비 개발자뿐만 아니라, 에어비앤비 개발자가 유지보수하고 있는 가장 유명한 eslint-config
###### @titicaca/triple-config-kit
스타트업 개발사인 트리플에서 개발하고 있으며, 현재도 꾸준히 업데이트되고 있다.  
* 자체적으로 정의한 규칙을 기반으로 운영
* 외부로 제공하는 규칙에 대한 테스트 코드 존재
* 일반적인 npm 라이브러리 구축 및 관리를 위한 시스템이 잘 구축돼 있음
###### eslint-config-next
리액트 기반 Next.js 프레임워크를 사용하고 있는 프로젝트에서 사용할 수 있는 eslint-config  
단순히 자바스크립트 코드를 정적으로 분석할 뿐만 아니라 페이지나 컴포넌트에서 반환하는 JSX 구문 및 HTML 코드 또한 정적 분석 대상으로 분류해 재공  
이는 Next.js 기반 웹 서비스의 성능 향상에 도움이 될 수 있어서 Next.js로 자성된 코드라면 반드시 설치하는 것이 좋다.
### 8.1.3 나만의 ESLint 규칙 만들기
##### 이미 존재하는 규칙을 커스터마이징해서 적용하기: import React를 제거하기 위한 ESLint 규칙만들기
리액트 17버전부터는 새로운 JSX 런타임 덕분에 import React 구문이 필요없어졌다.  
따라서 import React를 삭제하게 되면 번들러의 크기를 줄일 수 있다.
```
module.exports = {
    rules:{
        'no-restricted-imports':[
            'error,
            {
                paths: [
                    {
                        name:'react',
                        importNames:['default'],
                        message:'import React from 'react'는 react 17부터 더 이상 필요하지 않습니다. 필요한 것만 react로부터 import 해서 사용해 주세요.",
                    },
                ],
            },
        ],
    },
}
```
특정 모듈을 import하는 것을 금지하기 위해 만들어진 규칙인 no-restricted-imports 규칙을 사용해 import React를 금지시킴
##### 완전히 새로운 규칙 만들기: new Date를 금지시키는 규칙
new Date()가 나타내는 시간은 기기에 종속된 현재 시간으로, 기기의 현재 시간을 바꿔버리면 new Date()가 반환하는 현재 시간도 변경된다.  
개발하고자 하는 서비스가 사용 중인 기기에 상관없이 한국의 시간을 반환해야 한다고 가정했을 때 new Date()는 사용하지 못하고, 서버의 시간을 반환하는 함수인 ServerDate()를 만들어 이 함수만 사용해야 함.
    AST로 확인한 금지해야 할 new Date()의 노드
    1. type:NewExpression  
    2. callee.name:Date  
    3. ExpressionStatement.expression.arguments가 빈 배열인 경우  
ELiint의 create함수를 통해 규칙을 만들어 보면 다음과 같다.
```
/*
* @type {import('eslint').Rule.RuleModule}
*/
module.exports = {
    meta:{
        type:'suggestion',
        docs:{
            description: 'disallow use of the new Date()',
            recommended: false,
        },
        fixable: 'code',
        schema: [],
        messages: {
            message:'new Date()는 클라이언트에서 실행 시 해당 기기의 시간에 의존적이라 정확하지 않습니다. 현재 시간이 필요하다면 ServerDate()를 사용해 주세요.',
        },
    },
    create: function(context) {
        return {
            NewExpresstion: function(node) {
                if(node.callee.name === 'Date' && node.arguments.length === 0) {
                    context.report({
                        node:node,
                        messageId: 'message',
                        fix:function(fixer) {
                            return fixer.replaceText(node, 'ServerDate()')
                        },
                    })
                }
            },
        }
    },
}
```

### 8.1.4 주의할 점
##### Prettier와의 충돌
Prettier: 코드의 포매팅을 도와주는 도구로 줄바꿈, 들여쓰기, 작은따옴표와 큰따옴표 등 포매팅과 관련된 작업을 담당  
ESLint에서도 Prettier에서 처리하는 작업을 처리할 수 있기 때문에 두 가지 모두를 실행한다면 에러가 발생할 수 있다.  
    해결방법
    1. 서로 규칙이 충돌되지 않게끔 규칙 선언
    2. 자바스크립트나 타입스크립트는 ESLint에, 그 외의 파일은 모두 Prettier에게 맞기기
##### 규칙에 대한 예외 처리, 그리고 react-hooks/no-exhaustive-deps
일부 코드에서 특정 규칙을 임의로 제외시키고 싶으면 eslint-disalbe- 주석을 사용  
```java
console.log('hello world') // eslint-disable-line no-console
```
주의할 점 : eslint-disalbe을 많이 사용하고 있다면 무시하는 것이 옳은지, 해당 규칙을 제거하는 것이 옳은지 꼭 점검해봐야 함
##### ESLint 버전 충돌
eslint-config와 eslint-plugin이 지원하는 ESLint 버전을 확인하고, 설치하고자 하는 프로젝트에서 ESLint 버전을 어떻게 지원하고 있는지 살펴봐야 한다.  
이런 사전준비를 하지 않는다면 ESLint를 사용할 때마다 버전이 맞지 않는다는 오류 메세지를 마주하게 될 것이다.

### 8.1.5 정리
아직도 프로젝트에 ESLint가 적용돼 있지 않다면 적용해 보는 것을 권장한다.

## 8.2 리액트 팀이 권장하는 리액트 테스트 라이브러리
테스트 : 개발자가 만든 프로그램이 코딩을 한 의도대로 작동하는지 확인하는 일련의 작업
프런트엔드는 사용자에게 완전히 노출된 영역이므로 어떻게 작동할지 최대한 예측해서 확인해야 한다.

### 8.2.1 React Testing Library란?
React Testing Library:리액트를 기반으로 한 테스트를 수행하기 위해 만들어진 DOM Testing Library 기반의 테스팅 라이브러리  
DOM Testing Library은 jsdom을 기반으로 반들어짐  
jsdom:순수하게 자바스크립트로 작성된 라이브러리로, HTML이 없는 자바스크립트만 존재하는 환경에서 HTML과 DOM을 사용할 수 있도록 해주는 라이브러리  
리  
리액트 테스팅 라이브러리를 활용하면 실제로 리액트 컴포넌트를 렌더링하지 않고도 리액트 컴포넌트가 원하는 대로 렌더링되고 있는지 확인할 수 있다.

### 8.2.2 자바스크립트 테스트의 기초
기본적인 테스트 코드 작성 방식
1. 테스트할 함수나 모듈을 선정
2. 함수나 모듈이 반환하길 기대하는 값을 적는다.
3. 함수나 모듈의 실제 반환 값을 적는다.
4. 3번의 기대에 따라 2번의 결과가 일치하는지 확인한다.
5. 기대하는 결과를 반환한다면 테스트는 성공이며, 만약 기대와 다른 결과를 반환하면 에러를 던진다.
Node.js의 assert를 통해 다음과 같이 테스트할 수 있다.
```java
const assert = require('assert')
function sum(a, b) {
    return a + b
}
assert.equal(sum(1,2), 3)
```
이처럼 테스트 결과를 확인할 수 있도록 도와주는 라이브러리를 어설션 라이브러리라고 한다.  
###### 테스팅 프레임워크
* 좋은 테스트 코드는 다양한 테스트 코드가 작성되고 통과하는 것뿐만 아니라 어떤 테스트가 무엇을 테스트하는지 일목요연하게 보여주는 것도 중요한데, 이런 테스트의 기승전결을 완성해 주는 것이 바로 테스팅 프레임워크다.  
* 자바스크립트에서 유명한 테스팅 프레임워크 : Jest, Mocha, Karma, Jasmine 
* Node.js의 assert만 사용했을 때는 단순히 실패에 대한 단편적인 정보를 알 수 있지만, 테스트 프레임워크를 사용하면 테스트 코드를 작성하는 것뿐만 아니라 테스트에 대한 결과와 관련 정보를 확인 가능하다.
### 8.2.3 리액트 컴포넌트 테스트 코드 작성하기
컴포넌트 테스트 순서
1. 컴포넌트를 렌더링한다.
2. 필요하다면 컴포넌트에서 특정 액션을 수행한다.
3. 컴포넌트 렌더링과 2번의 액션을 통해 기대하는 결과와 실제 결과를 비교한다.

##### 프로젝트 생성
    npx create-react-app react-test --template typescript
##### 리액트 컴포넌트에서 테스트하는 일반적인 시나리오
특정한 무언가는 지닌 HTML 요서가 있는지 여부
1. getBy...
2. findBy...
3. queryBy...

##### 정적 컴포넌트
정적 컴포넌트:별도의 상태가 존재하지 않아 항상 같은 결과를 반환하는 컴포넌트  
테스트를 원하는 컴포넌트를 렌더링한 다음, 테스트를 원하는 요소를 찾아 원하는 테스트를 수행

##### 동적 컴포넌트
###### 사용자가 useState를 통해 입력을 변경하는 컴포넌트
Jest 에서 사용자 작동을 흉내내는 메서드는 type 외에도 클릭, 더블클릭, 클리어 등 다양하며, 이러한 메서드를 활용하면 웬만한 사용자 작동을 재현할 수 있다.  
###### 비동기 이벤트가 발생하는 컴포넌트
MSW:Node.js나 브라우저에서 모두 사용할 수 있는 모킹 라이브러리로, 브라우저에서는 서비스 워커를 활용해 실제 네트워크 요청을 가로채는 방식으로 모킹을 구현한다.  
이러한 방식은 fetch의 모든 기능을 그대로 사용하면서도 응답에 대해서만 모킹할 수 있으므로 fetch를 모킹하는 것이 훨씬 수월해진다.  

### 8.2.4 사용자 정의 훅 테스트하기
react-hooks-testing-library를 활용하면 훅을 더욱 편리하게 테스트할 수 있다.  

### 8.2.5 테스트를 작성하기에 앞서 고려해야 할 점
테스트 커버지리:해당 소프트웨어가 얼마나 테스트됐는지를 나타내는 지표  
테스트 커버리지는 얼마나 많은 코드가 테스트되고 있는지를 나타내는 지표일 뿐, 테스트가 잘되고 있는지를 나타내는 것이 아니다.  
테스트 코드를 작성하기 전에 생각해 봐야 할 최우선 과제는 애플리케이션에서 가장 취약하거나 중요한 부분을 파악해야 하는 것이다.  
### 8.2.6 그 밖에 해볼 만한 여러 가지 테스트
* 유닛 테스트:각각의 코드나 컴포넌트가 독립적으로 분리된 환경에서 의도된 대로 정확히 작동하는지 검증하는 테스트
* 통합 테스트:유닛 테스트를 통과한 여러 컴포넌트가 묶어서 하나의 기능으로 정상적으로 작동하는지 확인하는 테스트
* 엔드 투 엔드:실제 사용자처럼 작동하는 로봇을 활용해 애플리케이션의 전체적인 기능을 확인하는 테스트

### 8.2.7 정리
테스트할 수 있는 방법은 여러 가지가 있지만 테스트가 이뤄야 할 목표는 애플리케이션이 비즈니스 요구사항을 충족하는지 확인하는 것 한 가지뿐이다.