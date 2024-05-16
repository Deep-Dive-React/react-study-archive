# Chapter 11

# 2. 리액트 서버 컴포넌트

## 2-1. 기존 리액트 컴포넌트와 서버 사이드 렌더링의 한계

1. 자바스크립트 번들 크기가 0인 컴포넌트를 만들 수 없다.
2. 백엔드 리소스에 대한 직접적인 접근이 불가능하다.
3. 자동 코드 분할(code split)이 불가능하다.
4. 연쇄적으로 발생하는 클라이언트와 서버의 요청을 대응하기 어렵다.
5. 추상화에 드는 비용이 증가한다.

## 2-2. 서버 컴포넌트란?

<aside>
➕ 요즘 IT, 새로 등장한 ‘서버 컴포넌트’ 이해하기: `https://yozm.wishket.com/magazine/detail/2271/`
카카오페이, 

`React 18`: 리액트 서버 컴포넌트 준비하기: https://tech.kakaopay.com/post/react-server-components/

</aside>

- `서버 컴포넌트`
  - 요청이 오면 서버에서 한 번만 실행되므로 React API 사용이 제한적이다. hook, 렌더링 생명주기 뿐만 아니라 브라우저 API도 사용할 수 없다.
  - 데이터베이스, 내부 서비스, 파일 시스템 등 서버 데이터를 async/await 없이 접근할 수 있다. 컴포넌트 자체가 async한 것이 가능하다.
  - 번들에 포함되지 않는다.
  - **client boundaries**
    - **부모 자식 간에 따라 결정되는 것이 아니라, 호출 루트에 따라 결정된다?**
  - **‘컴포넌트 내에 서버 전용 코드를 실행할 방법이 생겼다.’는 것으로 이해하면 된다?**

리액트는 기본적으로 모든 것을 다 `공용 컴포넌트`로 판단한다. 이는 서버 컴포넌트일 수도, 클라이언트 컴포넌트일 수도 있음을 의미한다. 특정 경우에 한해 클라이언트 컴포넌트로 해석한다.

- `클라이언트 컴포넌트`
  - 서버 컴포넌트를 불러올 수는 없지만, 자식으로 출력하는 것은 가능하다.
    ```tsx
    'use client'

    import ServerComponent from './ServerComponent.server' // ❌
    export default function ClientComponent() {
    	return (
    		<div>
    			<ServerComponent />
    		</div>
    }
    ```
    ```tsx
    // page.js

    import ClientComponent from "./ClientComponent";
    import ServerComponent from "./ServerComponent";

    export default function Page() {
      return (
        <ClientComponent>
          <ServerComponent /> // ⭕️
        </ClientComponent>
      );
    }
    ```
    위의 경우 이미 렌더링된 결과물로 children에 들어간다.
  - `use client`를 작성하면 서버 컴포넌트를 import하는 시점에 에러를 발생시킨다.
## 2-3. 서버 사이드 렌더링과 서버 컴포넌트의 차이
`서버 사이드 렌더링(SSR)`은 클라이언트가 첫 요청을 보냈을 때, 서버에서 리액트 컴포넌트를 HTML로 렌더링하고 이를 클라이언트에게 보낸다. 이때 SSR로 렌더링된 컴포넌트는 클라이언트에서도 다시 렌더링되고, 이 과정에서 필요한 자바스크립트 파일은 번들에 포함되므로, 클라이언트의 번들 사이즈에 영향을 준다.
`서버 컴포넌트`는 서버에서만 렌더링되며, 그 결과가 HTML로 클라이언트에게 전달된다. 하지만 이 컴포넌트는 클라이언트에서 재렌더링되지 않으며, 자바스크립트 번들에 포함되지 않아 번들 사이즈를 줄일 수 있다
**따라서, 서버 사이드 렌더링과 서버 컴포넌트는 모두 서버에서 렌더링을 수행하지만, 클라이언트에서의 렌더링 방식과 번들 사이즈에 영향을 주는 방식이 다르다.**
저자는 이 둘을 대체제가 아닌 상호보하는 개념으로 보아야 한다고 말한다. 서버 컴포넌트를 활용해 서버에서 렌더링할 수 있는 컴포넌트는 서버에서 완성해서 제공받고, 클라이언트 컴포넌트는 서버 사이드 렌더링으로 초기 HTML을 빠르게 전달받을 수 있을 것이다.
## 2-4. 서버 컴포넌트는 어떻게 작동하는가?
- 서버 컴포넌트 데모: https://github.com/reactjs/server-components-demo
- 서버에서 클라이언트로 정보를 보낼 때 **JSON으로 직렬화**하여 스트리밍 형태로 보낸다. 이에 렌더링 속도가 빨라질 수 있다.
  - JSON으로 직렬화 할 수 없는 class나 Date는 전달할 수 없다.
- 컴포넌트들이 하나의 번들러 작업에 포함돼 있지 않고 각 컴포넌트별로 번들링이 되어 있어 필요에 **따라 컴포넌트를 지연해서 받거나 따로 받는 등의 작업이 가능하다**.
# 3. Next.js에서의 리액트 서버 컴포넌트
## 3-1. 새로운 fetch 도입과 getServerSideProps, getStaticProps, getInitialProps의 삭제
- app 디렉터리 내부에서 getServerSideProps, getStaticProps, getInitialProps가 삭제되었고, 데이터 요청은 **fetch**를 기반으로 이뤄진다.
- 이때 해당 fetch 요청에 대한 내용을 서버에서는 렌더링이 한 번 끝날 때까지 캐싱하며, 클라이언트에서는 별도의 지시자나 요청이 없는 이상 해당 데이터를 최대한 캐싱해서 중복된 요청을 방지한다.
## 3-2. 정적 렌더링과 동적 렌더링
정적인 라우팅에 대해서는 기본적인 캐싱이 지원되고, 동적인 라우팅에 대해서는 요청이 올 때마다 컴포넌트를 렌더링한다.
아래는 캐싱과 관련된 fetch 옵션이다.
- `fetch(URL, { **cache: ‘force-cache’** })`: 기본값으로 `getStaticProps`와 유사하게 불러온 데이터를 캐싱해 해당 데이터로만 관리한다.
- `fetch(URL, { **cache: ‘no-store’** })`, `fetch(URL, { **next: { revalidate: 0** }})`: `getServerSideProps`와 유사하게 캐싱하지 않고 매번 새로운 데이터를 불러온다.
- `fetch(URL, { **next: { revalidate: 10 }**})`: 정해진 유효시간 동안 캐싱하고, 이 유효시간이 지나면 캐시를 파기한다.
## 3-3. 캐시와 mutating, 그리고 revalidating
```tsx
// app/page.tsx
export const revalidate = 60;
```
fetch에 revalidate 옵션을 설정하는 것과 동일하다.
이러한 캐시를 무효화하고 싶다면 `router.refresh()`를 호출할 수 있다. 이 작업은 브라우저나 리액트 state에는 영향을 미치지 않는다.
## 3-4. 스트리밍을 활용한 점진적인 페이지 불러오기
이전의 서버사이드 렌더링은 요청받은 페이지를 한번에 내려줬다면, 이제 **스트리밍을 통해 먼저 완성된 페이지부터 점진적으로 보낼 수 있다.** 이는 핵심 웹 지표인 최초 바이트까지의 시간(TTFB)과 최초 콘텐츠풀 페인팅(FCP)를 개선하는 데 도움을 준다.
스트리밍을 활용하는 방법으로 아래 두가지가 있다.
1. 경로에 [loading.tsx](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming#instant-loading-states)를 배치한다
2. [Suspense](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming#streaming-with-suspense)를 배치한다. loading은 Suspense를 기반으로 하기 때문에 작동 원리는 동일하며 더욱 세분화된 제어가 가능하다.
   ```tsx
   import { Suspense } from "react";
   import { PostFeed, Weather } from "./Components";

   export default function Posts() {
     return (
       <section>
         <Suspense fallback={<p>Loading feed...</p>}>
           <PostFeed />
         </Suspense>
         <Suspense fallback={<p>Loading weather...</p>}>
           <Weather />
         </Suspense>
       </section>
     );
   }
   ```
# 4. 웹팩의 대항마, 터보팩(beta)
Next에서는 [SWC](https://nextjs.org/docs/architecture/nextjs-compiler) 사용을 공식적으로 권장하고 있다. Next.13에서는 [터보팩](https://nextjs.org/docs/architecture/turbopack)이 출시되었으며 이는 웹팩 대비 700배, vite 대비 10배 빠르다고 하며, 러스트 기반으로 작성됐기 때문에 가능하다고 소개하고 있다.


# 5. 서버 액션(alpha)
- `서버 액션`: API를 작성하지 않고 함수 수준에서 서버에 직접 접근해 데이터 요청 등을 수행할 수 있는 기능이다.
- 이를 사용하기 위해서는 함수 내부 혹은 파일 상단에 `‘use server’` 지시자를 선언해야 한다.
- 해당 함수는 반드시 `async`여야 한다.

## 5-1. form의 action
- `<form action=”” />` : [action prop](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#action)은 폼 제출을 처리할 URL을 받는다.
-
form.action으로 전달할 서버 함수를 만들어서 전달하자.
```tsx
export default function Page() {
  async function handleSubmit() {
    **'use server'**

    console.log('해당 작업은 서버에서 수행합니다. 따라서 CORS 이슈가 없습니다.')

    const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
      method: 'post',
      body: JSON.stringify({
        title: 'foo',
        body: 'bar',
        userId: 1,
      }),
      headers: {
        'Content-type': 'application/json; charset=UTF-8',
      },
    })

    const result = await response.json()
    console.log(result)
  }
  return (
    <div className="space-y-4">
      <form **action={handleSubmit}**>
        <button type="submit">form 요청 보내보기</button>
      </form>
    </div>
  )
}
```
handleSubmit **이벤트를 발생시키는 것은 클라이언트**지만 실제로 **함수 자체가 수행되는 것은 서버**다.


서버에 찍힌 로그
서버에는 아래와 같은 내용이 미리 빌드돼 있다. `use server` 가 선언된 부분이 서버로 분리되어 서버 액션이 만들어져있다. 그리고 이 서버 액션이 바인딩 된 actions 객체가 클라이언트에서 호출한 것임을 확인할 수 있다.
```tsx
// .next/server/app/server-action/form/page.js

const actions = {
	'ba6087ff74c8604692cadedc93932b0fca2692b6': () => Promise.resolve(/* import() eager */).then(__webpack_require__.bind(__webpack_require__, 5948)).then(mod => mod["$$ACTION_0"]),
}

...

function Page() {
    async function handleSubmit() {
        return $$ACTION_0(handleSubmit.$$bound);
    }
		...
}

async function $$ACTION_0(closure) {
    console.log("해당 작업은 서버에서 수행합니다. 따라서 CORS 이슈가 없습니다.");
    const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
        method: "post",
        body: JSON.stringify({
            title: "foo",
            body: "bar",
            userId: 1
        }),
        headers: {
            "Content-type": "application/json; charset=UTF-8"
        }
    });
    const result = await response.json();
    console.log(result);
}
```
이는 폼과 실제 노출하는 데이터가 연동돼 있을 때 더욱 효과적으로 사용할 수 있다.
아래는 Redis 스토리지 기반으로 서버 액션을 다루는 방법에 관한 예제다. 

Page는 서버 컴포넌트로 data 변수는 서버와 Redis 간의 직접적인 요청을 통해 데이터를 받아와 렌더링한다.

 그리고 폼 제출이 발생하면 서버 액션이 실행되고, 이는 다시 서버에서 Redis로 데이터를 직접 업데이트한다. 그리고 업데이트가 완료되면 `revalidatePath`를 호출해 컴포넌트를 재렌더링한다.
```tsx
import kv from "@vercel/kv";
import { revalidatePath } from "next/cache";

interface Data {
  name: string;
  age: number;
}

export default async function Page({ params }: { params: { id: string } }) {
  const key = `test:${params.id}`;
  const data = await kv.get<Data>(key);

  async function handleSubmit(formData: FormData) {
    "use server";

    const name = formData.get("name");
    const age = formData.get("age");

    await kv.set(key, {
      name,
      age,
    });

    revalidatePath(`/server-action/form/${params.id}`);
  }

  return (
    <div className="space-y-4">
      <h1 className="text-xl font-medium text-gray-400/80">form with data</h1>
      <h2 className="text-l font-medium text-gray-400/80">
        서버에 저장된 정보: {data?.name} {data?.age}
      </h2>

      <div className="space-y-4">
        <ul className="list-disc space-y-2 pl-4 text-sm text-gray-300">
          <li>아래 버튼을 누르면 서버에서 직접 form 요청을 보냅니다.</li>
          <form action={handleSubmit}>
            <li>
              <label htmlFor="name">이름: </label>
              <input
                type="text"
                id="name"
                name="name"
                defaultValue={data?.name}
                placeholder="이름을 입력해주세요."
              />
            </li>

            <li>
              <label htmlFor="age">나이: </label>
              <input
                type="number"
                id="age"
                name="age"
                defaultValue={data?.age}
                placeholder="나이를 입력해주세요."
              />
            </li>

            <li>
              <button type="submit">submit</button>
            </li>
          </form>
        </ul>
      </div>
    </div>
  );
}
```
기존에는 클라이언트 요청을 받아 데이터를 처리하는 API가 필요했다면, 이러한 과정 없이 폼 데이터와 서버 데이터 간의 동기화가 가능한 점이 핵심이다.
## 5-2. input의 submit과 image의 formAction
- `input type=”submit”`, `input type=”image”`에 fromAction prop으로도 서버 액션을 추가할 수 있다. 위의 방법과 동일하다.
## 5-3. startTransition과의 연동
- `startTransition`을 사용해 서버 액션을 실행할 수도 있다.
- `isPending`을 활용해 컴포넌트 단위의 로딩 처리도 가능하다.
```tsx
// Page.tsx

export default async function Page({ params }: { params: { id: string } }) {
  const key = `test:${params.id}`;
  const data = await kv.get<Data>(key);

  return (
    <div className="space-y-4">
      <h1 className="text-xl font-medium text-gray-400/80">form with data</h1>
      <h2 className="text-l font-medium text-gray-400/80">
        서버에 저장된 정보: {data?.name} {data?.age}
      </h2>

      <div className="space-y-4">
        <ul className="list-disc space-y-2 pl-4 text-sm text-gray-300">
          <li>아래 버튼을 누르면 서버에서 직접 form 요청을 보냅니다.</li>
          <li>이 작업은 useTransition을 기반으로 실행됩니다.</li>
          <li>
            <ClientButtonComponent id={params.id} />
          </li>
        </ul>
      </div>
    </div>
  );
}
```
```tsx
// ClientButtonComponent.tsx

"use client";
import { useCallback, useTransition } from "react";
import { updateData } from "#server-action";
import { SkeletonBtn } from "#components/components";

export function ClientButtonComponent({ id }: { id: string }) {
  const [isPending, startTransition] = useTransition();

  const handleClick = useCallback(() => {
    startTransition(() => updateData(id, { name: "기본값", age: 0 }));
  }, []);

  return isPending ? (
    <SkeletonBtn />
  ) : (
    <button onClick={handleClick}>기본값으로 돌리기</button>
  );
}
```
```tsx
"use server";

import kv from "@vercel/kv";
import { revalidatePath } from "next/cache";
import { cookies } from "next/headers";

export async function updateData(
  id: string,
  data: { name: string; age: number }
) {
  const key = `test:${id}`;

  await kv.set(key, {
    name: data.name,
    age: data.age,
  });

  revalidatePath(`/server-action/form/${id}`);
}
```
## 5-4. server mutation이 없는 작업
5-3처럼 server mutation이 필요하다면 서버 액션을 useTransition으로 감싸서 써야하지만, 별도의 server mutation이 없다면 생략해도 된다.
## 5-5. 서버 액션 사용 시 주의할 점
- 서버 액션은 클라이언트 컴포넌트 내에 정의될 수 없다. [useTransition 예제처럼](https://www.notion.so/Modern-React-Deep-Dive-Next-js-13-18-ced979761e5b4e398bb5d9a869c5e3ce?pvs=21) ‘use server’가 선언된 파일에 서버 액션을 작성하고, 이를 import해야 한다.
- 서버 액션을 props 형태로 클라이언트 컴포넌트에 넘길 수도 있다.