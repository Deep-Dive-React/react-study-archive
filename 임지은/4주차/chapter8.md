# 08. 좋은 리액트 코드 작성을 위한 환경 구축하기
## ESLint를 활용한 정적 코드 분석 
### ESLint
- 정적 코드 분석
- 코드의 실행과는 별개로 코드 그 자체만으로 잠재적으로 버그를 야기할 수 있는 코드를 찾아내어 문제의 소지가 있는 코드를 사전에 수정하는 것을 의미

#### ESLint는 어떻게 코드를 분석할까?
    1. 자바스크립트 코드를 문자열로 읽는다.
    2. 자바스크립트 코드를 분석할 수 있는 파서(parser)로 코드를 구조화한다. 
    3. 2번에서 구조화한 트리를 AST(Abstract Syntax Tree)라 하며, 이 구조화된 트리를 기준으로 각종 규칙과 대조한다.
    4. 규칙과 대조했을 때 이를 위반한 코드를 알리거나(report) 수정한다(fix).

2번 과정에 주목하자.

ESLint가 자바스크립트를 분석하는 파서 : `espree`
#### espree가 파일 구조화시키는 방식
```javascript
function hello(str) {}
```
위의 코드를 espree로 분석 (https://astexplorer.net/ 사이트에서 만들어줌)
```javascript
{
  "type": "Program",
  "start": 0,
  "end": 22,
  "body": [
    {
      "type": "FunctionDeclaration", // function 부분
      "start": 0,
      "end": 22,
      "id": {
        "type": "Identifier", // hello 부분
        "start": 9,
        "end": 14,
        "name": "hello"
      },
      "expression": false,
      "generator": false,
      "async": false,
      "params": [
        {
          "type": "Identifier", // str 부분
          "start": 15,
          "end": 18,
          "name": "str"
        }
      ],
      "body": {
        "type": "BlockStatement", // {} 부분
        "start": 20,
        "end": 22,
        "body": []
      }
    }
  ],
  "sourceType": "module"
}
```
코드의 정확한 위치와 같은 세세한 정보도 분석해주는데 그 이유는

`ESLint` `Prettier`와 같은 도구가 코드의 줄바꿈, 들여쓰기 등을 파악할 수 있게 되기 때문

### ESLint 규칙 만드는 방법

**debugger의 사용을 금지하자**

1. debugger을 espree로 분석
```javascript
{
  "type": "Program",
  "start": 0,
  "end": 8,
  "body": [
    {
      "type": "DebuggerStatement", // 이 이름을 규칙 생성에 이용!!
      "start": 0,
      "end": 8
    }
  ],
  "sourceType": "module"
}
```

2. no-debugger 규칙 생성 
```javascript
module.exports = {
    meta: {
        type: "problem",

        docs: {
            description: "Disallow the use of `debugger`",
            recommended: true,
            url: "https://eslint.org/docs/latest/rules/no-debugger"
        },

        fixable: null,
        schema: [],

        messages: {
            unexpected: "Unexpected 'debugger' statement."
        } // 문제 발생시 출력되는 메세지
    },

    // create가 실제로 코드에서 문제점을 확인하는 곳
    create(context) {

        return {
            DebuggerStatement(node) { // DebuggerStatement를 만나면
            // 해당 노드를 리포트해 debugger를 사용했음을 알린다.
                context.report({ 
                    node,
                    messageId: "unexpected"
                });
            }
        };

    }
};
```
### eslint-plugin
규칙을 모아놓은 패키지

`eslint-plugin-import` 

자바스크립트에서 다른 모듈을 불러오는 import와 관련된 다양한 규칙을 제공

`eslint-plugin-react`

리액트 관련 규칙
-> JSX 배열에 키를 선언하지 않았다는 경고!

### eslint-config
eslint-plugin을 한데 묶어서 완벽하게 한 세트로 제공하는 **패키지**

eslint-config를 만드는 것은 매우 번거롭기 때문에, 개발자가 만들기보다는 IT 기업들에서 공개한 잘 만들어진 eslint-config를 설치해서 사용한다.

-> eslint-config-airbnb

-> @titicaca/triple-config-kit

-> eslint-config-next

#### eslint-config와 eslint-plugin의 네이밍 규칙
1. eslint-config, eslint-plugin 접두사를 준수
2. 반드시 한 단어로 구성

### 나만의 ESLint 규칙 만들기
#### 1) 이미 존재하는 규칙을 커스터마이징해서 적용하기 : import React를 제거하기 위한 ESLint 규칙 만들기

리액트 17 버전부터는 새로운 JSX 런타임 덕분에 import React 구문이 필요 업어졌다.

따라서, 기존에 import React를 선언한 코드를 없애는 것이 좋다!

`no-restricted-imports` 규칙 사용 (<- 기존에 있던 것)

```javascript
module.exports = {
    rules: {
        'no-restricted-imports' : [
            'error',
            {
            // paths에 금지시킬 모듈 추가
            paths : [
                {
                    // 모듈명
                    name = 'react',
                    // 모듈의 이름
                    importNames : ['default'],
                    // 경고 메시지
                    message:
                    "import React from 'react'는 react 17부터 더 이상 필요하지 않습니다. 필요한 것만 react로부터 import 해서 사용해주세요"
                }
            ]
            }
        ]
    }
}
```

#### 2) 완전히 새로운 규칙 만들기: new Date를 금지시키는 규칙
자바스크립트 환경에서는 현재 시간을 알기 위해 new Date()를 사용하곤 한다.

그러나 이 현재 시간은 **기기에 종속된** 현재 시간으로, 기기의 현재 시간을 바꾸려면 new Date()가 반환하는 현재 시간 또한 변경된다.

"사용 중인 기기에 상관 없이 한국의 시간을 반환해야 한다면?"

=> new Data() 대신 `ServerDate()` 를 만들어 이 함수만 사용해야 한다!! <- 이걸 규칙을 만들자

#### 1. new Data() AST 구조 파악하기
```javascript
{
  "type": "Program",
  "loc": {
    "source": null,
    "start": {
      "line": 1,
      "column": 0
    },
    "end": {
      "line": 1,
      "column": 10
    }
  },
  "range": [
    0,
    10
  ],
  "body": [
    {
      "type": "ExpressionStatement", // 해당 코드이 표현식 전체
      "loc": {
        "source": null,
        "start": {
          "line": 1,
          "column": 0
        },
        "end": {
          "line": 1,
          "column": 10
        }
      },
      "range": [
        0,
        10
      ],
      "expression": {
        "type": "NewExpression", // new(생성자) ExpressionStatement.espression.type으로 접근해야한다.
        "loc": {
          "source": null,
          "start": {
            "line": 1,
            "column": 0
          },
          "end": {
            "line": 1,
            "column": 10
          }
        },
        "range": [
          0,
          10
        ],
        "callee": {
          "type": "Identifier", // Date, ExpressionStatement.espression.callee로 접근해야한다.
          "loc": {
            "source": null,
            "start": {
              "line": 1,
              "column": 4
            },
            "end": {
              "line": 1,
              "column": 8
            }
          },
          "range": [
            4,
            8
          ],
          "name": "Date",
          "typeAnnotation": null,
          "optional": false
        },
        "typeArguments": null,
        "arguments": [] // 인수, ExpressionStatement.espression.arguments
      },
      "directive": null
    }
  ],
  "comments": [],
  "errors": []
}
```
정리하자면

`new Date()`
- new : ExpressionStatement.espression.type
- Date : ExpressionStatement.espression.callee
- () : ExpressionStatement.espression.arguments

#### AST를 기반으로 ESLint 규칙 만들기
```javascript
/**
 * @type {import('eslint').Rule.RuleModule}
 **/

module.exports = {
    meta: {
        type : 'suggestion',
        docs:{
            description : 'disallow use of the new Date()',
            recommended : false,
        },
        fixable : 'code',
        schema: [],
        messages : {
            message : 
            'new Date()는 클라이언트에서 실행 시 해당 기기의 시간에 의존적이라 정확하지 않습니다. 현재 시간이 필요하다면 ServerDate()를 사용해주세요.'
        },
    },

    create: function(context) {
        return {
            // new 생성자를 사용할 때 ESLint가 실행
            NewExpression : function(code) {
                if (node.callee.name == 'Date' && node.arguments.length === 0) {
                    context.report({
                        node : node,
                        messageId : 'message',
                        // 위에서 fixable 'code'라고 했으므로, 코드 수정을 진행해준다.
                        fix : function(fixer) {
                            return fixer.replaceText(node, 'ServerDate()')
                        }
                    })
                }
            }
        }
    }
}
```


#### 만든 규칙을 eslint-plugin '묶음' 형태로 배포하기