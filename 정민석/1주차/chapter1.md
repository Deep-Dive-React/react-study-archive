# 1.1 자바스크립트의 동등 비교
리액트 렌더링이 일어나는 이유 중 하나가 props의 동등 비교에 따른 결과 객체의 얕은 비교를 기반으로 이뤄진다고 합니다.

예를 들어 부모 컴포넌트에서 자식 컴포넌트에게 props로 객체를 전달할 때, React는 객체의 얕은 비교를 수행하여 렌더링을 결정하는데
만약 객체의 레퍼런스가 변경되지 않으면 해당 객체의 내용이 변경되어도 React가 변경을 감지하지 못하게 됩니다.
```
// 부모 컴포넌트
import React, { useState } from 'react';
import ChildComponent from './ChildComponent';

function ParentComponent() {
  const [data, setData] = useState({ name: 'John', age: 30 });

  const handleChangeData = () => {
    // 객체의 내용을 변경해도 렌더링이 발생하지 않음
    setData({ name: 'John', age: 31 });
  };

  return (
    <div>
      <ChildComponent data={data} />
      <button onClick={handleChangeData}>Change Data</button>
    </div>
  );
}

export default ParentComponent;

// 자식 컴포넌트
function ChildComponent({ data }) {
  console.log('Rendering ChildComponent');
  return (
    <div>
      <p>Name: {data.name}</p>
      <p>Age: {data.age}</p>
    </div>
  );
}

export default ChildComponent;
```

![image](https://github.com/Deep-Dive-React/react-study-archive/assets/42230162/8e307f61-bb0e-4ff8-b26e-171d3679b93e)

이런 식으로 코드를 작성하는 경우 부모 컴포넌트가 전달하는 객체의 참조를 비교하기 때문에 내용은 변경되지만 참조는 그대로이기 때문에 렌더링을 다시하지 않는다고 합니다. 
따라서 
```
const handleChangeData = () => {
  // 새로운 객체를 생성하여 상태를 업데이트
  setData({ ...data, age: 31 });
};
```
이런식으로 spread연산자를 이용하여 이전 상태의 내용을 복사한 후 원하는 변경사항을 반영하여 새로운 객체를 생성하면 다시 렌더링을 할 수 있습니다.

## 원시 타입 vs 객체타입
- 원시 타입 : Boolean, String, Number, null, undefined, Symbol
- 참조 타입 : Object, Array

### null vs undefined
undefined는 '선언됐지만 할당되지 않은 값' 이고
null은 '명시적으로 비어 있음을 나타내는 값' 이라고 합니다.
```
function greet(name) {
    if (name === null) {
        console.log("Name is explicitly set to null.");
    } else if (name === undefined) {
        console.log("Name is not provided.");
    } else {
        console.log("Hello, " + name + "!");
    }
}

greet(); // "Name is not provided." 출력
```
이런 식으로 name과 undefined는 타입이 같지 않다는 것을 알 수 있습니다.

### Symbol
1. 각 Symbol 값은 고유하며 다른 어떤 Symbol 값과도 동일하지 않습니다. 이는 Symbol을 객체의 고유한 식별자로 사용할 수 있게 합니다.
2. 한 번 생성된 Symbol 값은 변경할 수 없습니다. 즉, 생성된 후에는 해당 Symbol 값을 수정할 수 없습니다.
3. 전역 심볼 레지스트리(Global Symbol Registry)를 통해 심볼 값을 전역으로 공유할 수 있습니다. 이를 통해 동일한 심볼 값을 어디서든지 참조할 수 있습니다.
즉 키로써 종종 사용을 한다고 합니다.

자바스크립트 엔진은 call stack과 heap memory 2가지 메모리 공간을 가지고 있다고 합니다.

콜스택: 실행 중인 함수를 추적해 계산을 수행하고 지역변수를 저장하는 공간입니다. 이곳에 원시 타입들이 저장됩니다.
힙 메모리: 참조 타입들이 할당되는 곳입니다. 메모리 누수를 방지하기 위해 js 엔진의 메모리 관리자가 항상 관리하는 공간입니다.

![image](https://github.com/Deep-Dive-React/react-study-archive/assets/42230162/37d23d99-cdec-4078-bc26-2f50effa8dd4)

만약 b에 100을 추가하는 연산을 하게 되면 다음 그림과 같이 되는데요

![image](https://github.com/Deep-Dive-React/react-study-archive/assets/42230162/bd36d790-a688-420c-bfee-97b5c4b7f894)

리액트에서는 Call Stack의 주소값만 가지고 얕은 비교를 하므로 값이 달라지는 것을 알 수 없다고 합니다 :(

# 1.2 함수
### 함수를 정의하는 4가지 방법
#### JS에서 함수는 일급 객체이다!

- 함수 선언문
```
function add(a,b){
  return a+b
}
```
- 함수 표현식
```
const sum = function add(a,b){
  return a+b
}
sum(1,2) //3
add(1,2) // add is not defined?!
```

둘의 차이를 알아 보기 전에 함수 호이스팅이라는 개념을 알아봐야 합니다.
이번에는 함수 호이스팅에 대해서만 다루고 넘어가겠습니다.
JS는 인터프리터 언어인데요 코드가 작성된 순서대로 진행이 됩니다.
```
one()

function one(){
  return 1
}
```
그렇면 이러한 상황에서 위에서부터 코드가 진행이 되는데 함수가 선언이 되지 않았는데 one()이라는 함수를 어떻게 알 수 있을까요?

> 함수 호이스팅은 함수 선언이 컴파일 단계에서 메모리에 할당하는 과정입니다.

함수 선언문은 선언된 위치와 상관없이 해당 스코프의 맨 위로 옮겨지며, 함수 표현식은 호이스팅되지 않습니다. 함수 선언문은 함수의 전체를, 함수 표현식은 변수에 할당되는 함수의 레퍼런스만을 올립니다.
```
//함수 선언문
myFunction(); // "Hello, world!"
function myFunction() {
  console.log("Hello, world!");
}

// 함수 표현식은 호이스팅되지 않음
myFunctionExpression(); // Error: myFunctionExpression is not a function
var myFunctionExpression = function() {
  console.log("Hello, world!");
};
```
함수 표현식과 선언문의 차이는 호이스팅이라고 하면 되겠네요. 각자 선언하고 싶은 상황에 따라 선언하면 된다고 합니다.
저는 개인적으로 표현식을 선언하는 것을 선호하는 것 같아요 ;)

- Function 생성자
```
const add = new Function('a','b','return a+b')
```
처음 보는 신기한 형태이네요

- 화살표 함수
```
const add = (a,b) => {
  return a+b
}
const add =(a,b) => a + b
```
가독성이 뛰어난 방식입니다.
하지만 안좋은 점이 있습니다.
1. constructor를 사용할 수 없다
2. arguments가 존재하지 않는다.
3. this 함수 자체의 바인딩을 갖지 않는다. -> 상위 스코프의 this를 따르게 된다.

간단한 lambda 함수 느낌으로 사용하면 좋을 것 같습니다.

# 1.3 클래스
JS에서 클래스는 ES6때 추가가 됐다고 합니다. 따라서 그 이전에 작성된 것들(함수)로 클래스와 동일하게 표현을 할 수 있다고 해요
babel에서 트랜스 파일하면 서로 바꿀 수 있습니다 :)

객체 지향을 공부해 보신 분들이라면 한번 쯤은 들어보셨을 거 같은 클래스입니다.
그렇다면 어떤 것으로 이루어져 있는지 봐보겠습니다.

- constructor
```
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}
const person = new Person('John', 30);
```
이처럼 객체를 생성해 초기 상태를 설정할 때 사용이 됩니다.

- 프로퍼티
위의 예제에서 name, age에 해당하는 값입니다.
인스턴스가 생성될때 내부에 만들어지는 속성값이라고도 해요.

- getter setter
```
  class Rectangle {
  constructor(width, height) {
    this._width = width;
    this._height = height;
  }

  get area() {
    return this._width * this._height;
  }

  set width(value) {
    this._width = value;
  }

  set height(value) {
    this._height = value;
  }
}

const rect = new Rectangle(10, 20);
console.log(rect.area); // 200
rect.width = 5;
console.log(rect.area); // 100
```
프로퍼티를 받아오거나 가져다 쓸 때 사용합니다.

- 인스턴스 메서드
![image](https://github.com/Deep-Dive-React/react-study-archive/assets/42230162/32060900-3e48-4069-a444-343405fcf29f)

인스턴스 메서드는 프로토타입 메서드라고도 불린다고 합니다.
위의 사진처럼 Person이라는 클래스를 생성하게 되면 Person의 prototype객체도 자동으로 생성이 되는데요
이때 kim이라는 객체와 Person 클래스는 prototype에 의해 연결이 됩니다.

- 정적 메서드
```
class MathUtil {
  static add(x, y) {
    return x + y;
  }

  static subtract(x, y) {
    return x - y;
  }
}

console.log(MathUtil.add(5, 3)); // 8
```
클래스의 인스턴스가 아닌 클래스 자체에 연결된 메서드입니다. 정적 메서드는 인스턴스에 의존하지 않고 클래스의 기능을 사용할 때 유용합니다.
위의 사진에서 프로토 타입이랑 연결이 되지 않다고 생각하면 될 것 같습니다.

- 상속
```
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    console.log(`${this.name} makes a noise.`);
  }
}

class Dog extends Animal {
  speak() {
    console.log(`${this.name} barks.`);
  }
}

const dog = new Dog('Buddy');
dog.speak(); // Buddy barks.
```
객체 지향의 꽃 상속입니다. 상속을 통해 부모 클래스의 기능을 재사용하고 확장하여 새로운 클래스를 만들 수 있습니다.

> JS class는 프로토타입을 기반으로 작동됩니다.

# 1.4 클로저
만약 네이버 메일이 
var islogin = True;
가 되면 메일을 접근할 수 있다고 생각해보자
그렇다면 개발자 도구 켜서 islogin = True로 설정하면 메일을 볼 수 있는것이다...

이러는 것을 방지하기 위해 클로저라는 개념이 도입이 됐다고 한다.

클로저는 함수가 자신이 호출될 때 상황에서 한단계 외부 스코프의 변수와 함수를 기억할 수 있다는 것이다. 2단계 이상의 스코프는 접근할 수 없다고 한다.


## let vs var
- 유효 범위 (Scope)
var: 함수 스코프를 가집니다. 즉, 함수 내에서 선언된 변수는 함수 내에서만 유효합니다.
let: 블록 스코프를 가집니다. 즉, 중괄호 {} 내에서 선언된 변수는 해당 블록 내에서만 유효합니다.

- 호이스팅 (Hoisting)
var 변수는 선언 이전에도 참조할 수 있으며, 이 경우 undefined로 초기화됩니다. 이것을 "호이스팅"이라고 합니다.
let 변수는 선언하기 전에는 참조할 수 없습니다. 따라서 호이스팅이 발생하지 않습니다.

- 재할당 및 재선언
var 변수는 동일한 이름으로 여러 번 선언될 수 있습니다. 이는 변수의 값을 재할당할 때와 관련이 있습니다.
let 변수는 같은 범위에서 한 번만 선언될 수 있습니다. 동일한 이름의 변수를 재선언하면 구문 오류가 발생합니다. 하지만 재할당은 가능합니다.

- 전역 객체 속성
var로 선언된 전역 변수는 전역 객체(window)의 속성이 됩니다. 즉, var로 선언된 변수는 전역 스코프에서 사용할 수 있습니다.
let으로 선언된 변수는 전역 객체(window)의 속성이 되지 않습니다. 따라서 전역 변수로 사용할 때 좀 더 안전합니다.

#### 일반적으로 ES6 이후에는 var보다 let과 const를 더 많이 사용하는 것이 권장됩니다. 이는 블록 스코프와 호이스팅이 제어되는 등 코드의 가독성과 안정성을 향상시키기 때문입니다. const는 변수에 재할당이 필요하지 않을 경우 사용되며, 이로써 코드의 의도를 명확히 전달할 수 있습니다.

선언할때 위의 사항을 고려해서 개발을 진행하도록 해봅시다.

# 1.5 이벤트 루프와 비동기 통신의 이해

> JS는 싱글 스레드를 지원한다.
이 말이 무슨 뜻이냐면 한번의 일밖에 실행하지 못한다는 뜻입니다.
하지만 우리는 JS로 이뤄진 앱을 사용하면서 이런 점을 깨닫지 못할 것 입니다.

그 이유는 바로 call 스택과 이벤트 루프를 사용하기 때문입니다.

## 호출 스택(Call Stack)
호출 스택은 함수 호출을 기록하는 자료구조로, 현재 실행 중인 함수를 포함한 함수 호출의 순서를 추적합니다. 일반적으로 함수가 호출될 때 호출 스택에 해당 함수의 호출 정보가 추가되고, 함수가 종료될 때 호출 스택에서 해당 정보가 제거됩니다.
```
function greet(name) {
  console.log(`Hello, ${name}!`);
}

function sayHello() {
  const userName = 'Alice';
  greet(userName);
}

sayHello();
```
### 실행결과
1. sayHello() 함수 호출
2. sayHello() 함수 내부에서 greet() 함수 호출
3. greet() 함수 실행
4. greet() 함수 종료
5. sayHello() 함수 종료

## 이벤트 루프(Event Loop)
이벤트 루프는 비동기 작업을 관리하고 실행하는 메커니즘으로, 호출 스택과 함께 JavaScript의 동작을 조정합니다. 이벤트 루프는 호출 스택이 비어있을 때까지 대기하다가 비동기 작업이 완료되면 해당 작업을 호출 스택으로 이동시킵니다.
```
console.log('Start');

setTimeout(() => {
  console.log('Inside setTimeout');
}, 0);

console.log('End');
```
### 실행 결과
1. Start
2. End
3. Inside setTimeout

![image](https://github.com/Deep-Dive-React/react-study-archive/assets/42230162/0b23f936-456c-4eee-85a4-2e3141917b0b)
