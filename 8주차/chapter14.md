# 14장|웹사이트 보안을 위한 리액트와 웹페이지 보안 이슈
## 14.1 리액트에서 발생하는 크로스 사이트 스크립팅(XSS)
▪ 크로스 사이트 스크립팅 : 웹사이트 개발자가 아닌 제3자가 웹사이트에 악성 스크립트를 삽입해 실행할 수 있는 취약점
### 14.1.1 dangerouslySetInnerHTML prop
* 특정 브라우저 DOM의 innerHTML을 특정한 내용으로 교체할 수 있는 방법  
* 오직 __html 키를 가지고 있는 객체만 인수로 받을 수 있으며, 이 인수로 넘겨받은 문자열을 DOM에 그대로 표시하는 역할을 한다.  

💢 인수로 받는 문자열에는 제한이 없기 때문에 사용시 주의를 기울여야 하는 prop이며, 여기에 넘겨주는 문자열 값은 한 번 더 검증이 필요하다.

### 14.1.2 useRef를 활용한 직접 삽입
* useRef를 활용하면 직접 DOM에 접근할 수 있으므로 이 DOM에 앞서와 비슷한 방식으로 innerHTML에 보한 취약점이 있는 스크립트를 삽입하면 동일한 문제가 발생한다.

### 14.1.3 리액트에서 XSS 문제를 피하는 방법
_새니타이즈/이스케이프 : 리액트에서 XSS 이슈를 피하는 가장 확실한 방법은 제3자가 삽입할 수 있는 HTML을 환전한 HTML 코드로 한 번 치환하는 것_

* sanitize-html은 허용할 태그와 목록을 일일이 나열하는 **허용 목록(allow list)** 방식을 채택하기 때문에 안전하다.
<img src="./img/2.jpg"></img><br/>
<img src="./img/1.jpg"></img>

* 단순히 보여줄 때뿐만 아니라 사용자가 콘텐츠를 저장할 때도 이스케이프 과정을 한 번 거치는 것이 더 효율적이고 안전하다.

💢 개발자는 자신이 작성한 코드가 아닌 외부에 존재하는 모든 코드를 위험한 코드로 간주하고 이를 적절하게 처리하는 것이 좋다.

## 14.2 getServerSideProps와 서버 컴포넌트를 주의하자
서버에는 일반 사용자에게 노출되면 안 되는 정보들이 담겨 있기 때문에 클라이언트, 즉 브라우저에 정보를 내려줄 때는 조심해야 한다.
* getServerSideProps가 반환하는 props 값은 모두 사용자의 HTML에 기록되고, 또한 전역 변수로 등록되어 스크립트로 충분히 접근할 수 있는 보안 위협에 노출되는 값이 된다.
* 따라서 getServerSideProps가 반환하는 값 또는 서버 컴포넌트가 클라이언트 컴포넌트에 반환하는 props는 반드시 필요한 값으로만 철저하게 제한되어야 한다.

## 14.3 ```<a>``` 태그의 값에 적절한 제한을 둬야 한다
```<a>``` 태그의 href에 javascript:로 시작하는 코드를 넣어둔 경우

▶️ ```<a>``` href로 선언된 URL로 페이지를 이동하는 것을 막고 onClick 이벤트와 같이 별도 이벤트 핸들러만 작동시키기 위한 용도로 주요 사용된다.  
```java
function App() {
    function handleClick() {
        console.log('hello')
    }

    return (
        <>
            <a href="javascript:;" onClick={handleClick}>
                링크
            </a>
        </>
    )
}
```
▶️ 이렇게 하면 ```<a>```의 href가 작동하지 않아 페이지 이동이 일어나지 않는 대신 onClick의 핸들러만 실행되는 것을 볼 수 있다.  

💢 그러나 ```<a>``` 태그는 반드시 페이지 이동이 있을 때만 사용하는 것이 좋으며, 페이지 이동 없이 어떠한 핸들러만 작동시키고 싶다면 ```<a>```보다는 button을 사용하는 것이 좋다.

## 14.4 HTTP 보안 헤더 설정하기
__HTTP 보안 헤더__ : 브라우저가 렌더링하는 내용과 관련된 보안 취약점을 미연에 방지하기 위해 브라우저와 함께 작동하는 헤더  

HTTP 보안 헤더만 효율적으로 사용할 수 있어도 많은 보안 취약점을 방지할 수 있다.  

### 14.4.1 Strict-Transport-Security
__HTTP의 Strict-Transport-Security 응답 헤더__ : 모든 사이트가 HTTPS를 통해 접근해야 하며, 만약 HTTP로 접근하는 경우 이러한 모든 시도는 HTTPS로 변경되게 한다.  

    사용법은 다음과 같다:
    Strinct-Transport-Security: max-age=<expire-time>; includeSubDomains 
* ```<expire-time>```은 이 설정을 브라우저가 기억해야 하는 시간을 의미하며, 초 단위로 기록된다. 
* 이 기간 내에는 HTTP로 사용자가 요청한다 하더라도 브라우저는 이 시간을 기억하고 있다가 자동으로 HTTPS로 요청하게 된다.

### 14.4.2 X-XSS-Protection
__X-XSS-Protection__ : 페이지에서 XSS 취약점이 발견되면 페이지 로딩을 중단하는 헤더  
```
X-XSS-Protection: 0
X-XSS-Protection: 1
X-XSS-Protection: 1; mode=block
X-XSS-Protection: 1; report=<reporting-uri>
```
* 0 : XSS 필터링을 끈다.
* 1 : 기본값으로, XSS 필터링을 켜게 된다. 만약 XSS 공격이 페이지 내부에서 감지되면 XSS 관련 코드를 제거한 안전한 페이지를 보여준다. 
* 1; mode=block : 1과 유사하지만 코드를 제거하는 것이 아니라 아예 접근 자체를 막아버린다.
* 1; report=<reporting-uri>는 크로미움 기반 브로우저에서만 작동하며, XSS 공격이 감지되면 보고서를 report= 쪽으로 보낸다.  

### 14.4.3 X-Frame-Options
__X-Frame-Oprions__ 는 페이지를 frame, iframe, embed, object 내부에서렌더링을 허용할지를 나타낼 수 있다.  

    (ex) 네이버와 비슷한 주소를 가진 페이지가 있고, 공격자는 이 페이지에서 네이버를 iframe으로 렌더링하여 사용자의 개인정보를 탈취하려 하는 경우
    ▶️ X-Frame-Oprtions는 외부에서 자신의 페이지를 위와 같은 방식으로 삽입되는 것을 막아주는 헤더다.

아래 코드와 같이 네이버와 관계없는 제3의 페이지에서 ```<iframe>```으로 네이버 페이지를 삽입해서 실행하면 네이버 페이지가 정상적으로 노출되지 않음을 볼 수 있다.

```
export default function App() {
    return (
        <div className="App">
            <iframe src="https://www.naver.com" />
        </div>
    )
}
```
<img src="./img/3.jpg"></img>
이는 네이버에 X-Frame-Options: deny 옵션이 있기 때문이다. 이 옵션은 제 3의 페이지에서 ```<iframe>```으로 삽입되는 것을 막아준다.
