# lazy
- 컴포넌트가 렌더링될 때까지 코드의 로딩을 지연시킨다.
- 컴포넌트의 외부에서 호출해야 한다.
- 렌더링할 리액트 컴포넌트를 리턴한다.
- 스피너를 표시하고싶으면 suspense를 쓰도록..

## parameter
### load 함수
- 프로미스나 thenable(then 메서드를 가지는 프로미스 같은 객체)를 리턴하는 함수
- 컴포넌트 타입으로 resolve되는 값을 리턴해야 한다.
- 이 프로미스가 reject 되면 에러를 던진다.

# 사용법
## Suspense가 있는 레이지 로딩 컴포넌트
- 일반적으로 정적 컴포넌트를 import하지만, lazy를 쓰면 동적으로 import한다.
- 필요할 때만 로드해오므로, 로딩되는 동안 표시할 내용을 지정해줘야 한다.

```jsx
import { lazy } from 'react';

const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));

// ~~ 생략 ~~

<Suspense fallback={<Loading />}>
  <h2>Preview</h2>
  <MarkdownPreview />
 </Suspense>


// ~~ 생략 ~~


```
