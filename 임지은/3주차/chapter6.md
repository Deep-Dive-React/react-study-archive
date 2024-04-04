# 06.리액트 개발 도구로 디버깅하기
## 6.3 리액트 개발 도구 활용하기
### Components
#### 컴포넌트 트리
    컴포넌트 탭에서는 현재 리액트 애플리케이션의 컴포넌트 트리를 확인할 수 있다. 
    가명함수로 선언된 컴포넌트 > 해당 컴포넌트명
    익명함수로 선언된 컴포넌트 > Anonymous로

컴포넌트 구조를 파악하려면 최대한 가명함수로 적는 것이 좋다. 

만약 함수를 기명함수로 바꾸기 어렵다면 함수에 displayName 속성을 추가하는 방법도 있다.
```javascript
const MomoizedComponent = memo(function() {
    return <>MemoizedComponent</>
})

MemoizedComponent.displayName = '메모 컴포넌트입니다.'
```
    displayName과 함수명은 개발 모드에서만 제한적으로 참고하는 것이 좋다.
    왜냐, 빌드 도구가 사용하지 않는 코드로 인식해 삭제하거나 난수화한다.
#### hooks
    useEffect 와 같은 훅은 대부분 익명 함수를 인수로 넘겨주기 때문에, useEffect가 여러개 선언돼 있다면 어떤 훅인지 구별하기 어렵다. 이 경우 기명함수로 선언한다면 개발 도구를 더욱 유용하게 이용할 수 있다. 
