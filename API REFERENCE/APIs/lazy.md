# `lazy(load)`
: 처음으로 컴포넌트가 렌더링될 때 까지 코드의 로딩을 지연시킴
- 레이지 로딩된 컴포넌트를 정의하려면 컴포넌트 **외부**에서 `lazy` 호출
- 트리에 렌더링할 수 있는 리액트 컴포넌트를 리턴
- lazy 컴포넌트의 코드가 로딩되고 있는 중에 렌더링을 시도하면 일시중단됨<br>→ 로딩되는 동안 로딩 표시를 하려면 <Suspense> 사용

### `load`
: 프로미스나 thenable(`then` 메서드를 가지는 프로미스 같은 객체)을 리턴하는 함수
- 매개변수를 받지 않음
- 리턴한 결과는 함수, `memo`, `forwardRef` 컴포넌트 같이 유효한 컴포넌트 타입으로 resolve 되어야 한다.
- 리턴되는 컴포넌트를 처음으로 렌더링하려고 할때까지 `load`는 호출되지 않는다. 
- 처음으로 `load`가 호출되고 나면 resolve될 때까지 기다렸다가 resolve된 값을 리액트 컴포넌트로 렌더링한다. 리턴된 프로미스와 프로미스의 resolve된 값 모두 캐시되므로 `load`는 두번 이상 호출되지 않는다. 
- 만약 프로미스가 reject되면 가장 가까운 Error Boundary에 그 이유를 `throw`함

## Suspense가 있는 레이지 로딩 컴포넌트
- 일반적으로는 정적으로 컴포넌트를 import하지만 처음 렌더링될 때까지 컴포넌트의 로딩을 지연하려면 `lazy`를 통해 동적으로 import할 수 있음
  ```javascript
  // 정적 import
  import MarkdownPreview from './MarkdownPreview.js';

  // 동적 import
  import { lazy } from 'react';
  const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));
  ```
  (동적 `import()`는 아직 모든 브라우저에서 지원하는 기능이 아니기 때문에 빌드 시 번들러나 프레임워크의 지원이 필요할 수 있음)
- 이렇게 하면 컴포넌트 코드는 필요할 때만 로드되므로 lazy 컴포넌트나 그 부모 중 하나를 `<Suspense>` 경계로 감싸 로딩중에 표시할 내용을 지정해야 함
- 레이지 컴포넌트는 캐시되므로 컴포넌트가 다시 불러와질 때는 로딩 표시가 나타나지 않음

## Troubleshooting: lazy 컴포넌트의 상태값이 예상치 못하게 초기화될 때
다른 컴포넌트 내부에서 `lazy` 컴포넌트를 선언하면 리렌더링시마다 모든 state가 초기화된다.<br>⇒
`lazy` 컴포넌트는 항상 **모듈의 최상위 레벨**에서 선언되어야 한다.
