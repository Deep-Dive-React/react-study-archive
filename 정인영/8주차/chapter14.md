## 14.1 프론트엔드에서 발견할 수 있는 보안 이슈

### 14.1 리액트에서 발생하는 크로스 사이트 스크립팅

XSS (크로스 사이트 스크립팅) 은 웹사이트 개발자가 아닌 제 3자가 웹사이트에 악성 스크립트를 삽입하여 실행할 수 있는 취약점을 의미한다. 이 취약점은 일반적으로 게시판과 같이 사용자가 입력을 할 수 있고, 이 입력을 다른 사용자에게 보여줄 수 있는 경우에 발생한다.

```javascript
<p>사용자가 글을 작성했어요</p>
<script>
 alert("XSS") // 나쁜짓
</script>
```

14.1.1
```javascript
functoin App(){
     return <div dangeroulsySetInnerHTML ={{__html: "blabla"}}></div>
}
```

- dangerouslySetInnerHTML prop

-> 일반적으로 게시판과 같이 사용자나 관리자가 입력한 내용을 브라우저에 표시하는 용도로 사용 

그러나 dangeroulsSetInnerHTML이 인수로 받는 문자열에는 제한이 없다. 즉, 사용에 주의를 해야 하는 prop이다.


14.1.2

`useRef` 로 DOM 에 접근하여 innerHTML 의 값을 바꾸는 행위도 스크립트 삽입 위험성이 존재한다.


14.1.3 

리액트에ㅔ서 XSS 이슈를 피하려면 제 3자가 삽입할 수 있는 HTML은 안전한 HTML 코드로 치환해야 한다. (sanitize, 새니타이즈)

새니타이즈 라이브러리를 npm 에서 다운받아서 사용해보자!

## 14.2 getServerSideProps와 서버컴포넌트를 조심하자.

서버에는 사용자에게 노출되는 안되는 정보들이 존재하기 때문에, 클라이언트 코드로 정보를 전달할때 유의해야 한다.            

getSeverSideProps으로 값을 전달할 때는 최대한 필요한 부분만 전달한다!