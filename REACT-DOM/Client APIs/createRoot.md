# createRoot
- 브라우저의 DOM 노드 안에 React 컴포넌트를 표시하는 루트를 생성하는 API

<br/>

# 1. Reference 
```js
import { createRoot } from 'react-dom/client';

const domNode = document.getElementById('root');
const root = createRoot(domNode);
```

- 루트를 생성한 후, 그 안에 React 컴포넌트를 표시하기 위해 `root.render(<App />;)`를 호출해야 한다.
- 온전히 React만으로 구축된 앱은 보통 루트 컴포넌트에 대한 `createRoot` 호출이 하나만 있다.
- 페이지의 일부에만 React를 사용하는 경우 필요한 만큼 여러 개의 독립적인 루트를 가질 수 있다.

## 1) Parameters
### domNode
- 루트를 생성할 DOM 엘리먼트

### options (optional)
- 루트에 대한 옵션을 가진 객체
- onRecoverableError: 오류로부터 자동으로 복구될 때 호출될 콜백
- identifierPrefix: useId에 의해 생성된 ID에 사용할 문자열 접두사로, 같은 페이지에서 여러 개의 루트를 사용할 때 충돌을 피하는 데 유용하다.

## 2) Returns
- `render` 메서드와 `unmount` 메서드가 있는 객체를 반환한다.

## 3) 주의사항
- 앱이 서버에서 렌더링되는 경우 `createRoot`를 사용할 수 없고, `hydrateRoot`를 사용해야 한다.
- 앱에 `createRoot` 호출이 하나만 있을 가능성이 높다. 프레임워크를 사용하는 경우에는 프레임워크가 대신 호출 해줄걸
- 현재 컴포넌트의 자식이 아닌 DOM 트리의 다른 부분(ex: Modal, Tooltip 등)에 JSX 조각을 렌더링하려는 경우에는 `createPortal`을 사용해야 한다.

<br/>

> 이어서, 리턴되는 객체에 담긴 `render` 메서드와 `unmount` 메서드에 대해 알아보자

# 1-1. root.render(reactNode)
```js
root.render(<App />);
```

- 이걸 호출해서 JSX 조각을 React 루트의 브라우저 DOM 노드에 표시한다.
- `root.render(<App />);`를 호출하면, React는 루트 안에 `<App/>`을 표시하고 그 안의 DOM 관리를 인수한다. 😎

## 1) Parameters
### reactNode
- 표시하려는 React 노드
- 일반적으로 `<App />`과 같은 JSX 조각이 되지만, `createElement()`로 구성된 React 엘리먼트, 문자열, 숫자, null, undefined도 ㄱㄴ

## 2) Returns
- undefined

## 3) 주의사항
- 처음 호출할 때, React는 React 컴포넌트를 렌더링하기 전에 React 루트 안에 있던 **기존 HTML 컨텐츠를 전부 지우고** React 컴포넌트를 그 안에 렌더링한다.
- 서버에서 또는 빌드 중에 React에 의해 생성된 HTML이 루트의 DOM 노드에 포함된 경우, `createRoot` 대신에 이벤트 핸들러를 기존 HTML에 첨부하는 `hydrateRoot()`를 사용해야 한다.
- 같은 루트에 render를 여러 번 호출하면, React는 최신 JSX를 반영하도록 필요에 따라 DOM을 업데이트한다.
    - 이전에 렌더링 된 트리와 비교해서 재사용할 수 있는 부분과 다시 만들어야 하는 부분을 결정한다.
    - 같은 루트에 다시 render를 호출하는 것은 set 함수를 호출하는 것과 비슷하다.
    - 즉, 불필요한 업데이트는 알아서 피한다. 👍


# 1-2. root.unmount()
```js
root.unmount();
```

- React 루트 안에 렌더링된 트리를 파괴한다.
- 온전히 React만으로 작성된 앱에는 보통 `root.unmount`을 호출하지 않는다.
- 이 함수는 React 루트의 DOM 노드 (또는 그 조상 노드)가 다른 코드에 의해 DOM에서 제거될 수 있는 경우에 유용하다.
    - ex: DOM에서 비활성 탭을 제거하는 jQuery 탭 패널은 탭이 제거되면 그 안에 있는 모든 것도 DOM에서 제거될 테니 이때 `root.unmount`를 호출해서 제거된 루트의 컨텐츠 관리를 중지하도록 React에 알려줘야 한다.
    - 그러지 않으면 제거된 루트 내부의 컴포넌트가 구독과 같은 전역 리소스를 정리하고 확보할 수 없다.
- `root.unmount`를 호출하면 루트의 모든 컴포넌트가 unmount되고, 트리의 이벤트 핸들러나 state가 제거되며, 루트 DOM 노드에서 React가 분리된다.
- Parameters와 Returns 없음

## 1) 주의사항
- `root.unmount`를 호출하고 나면 같은 루트에 대해 `root.render`를 다시 호출할 수 없다.
    - unmount된 루트에서 `root.render`를 호출하려고 하면 `Cannot update an unmounted root` 오류가 발생한다.
    - but, 해당 노드의 이전 루트가 unmount 된 후 동일한 DOM 노드에 새로운 루트를 만드는 건 ㄱㄴ (새로 createRoot 해야된다는 뜻)

<br/>

# 2. Usage
## 1) React뿐인 앱 렌더링하기
- 전체 앱에 대해 단일 루트를 생성한다.

```js
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

- 이 코드는 시작할 때 한 번만 실행하면 된다.
- 앱이 온전히 React만으로 작성되었으면 추가적으로 루트를 더 만들거나 `root.render`를 다시 호출할 필요 없다.
    - 이 시점부터 React는 전체 앱의 DOM을 관리하므로, 
        - 컴포넌트를 더 추가하려면 App 컴포넌트 안에 중첩 ㄱㄱ
        - UI 업데이트는 각 컴포넌트의 state를 통해 ㄱㄱ
        - 모달, 툴팁과 같은 추가 컨텐츠를 DOM 노드 외부에 표시해야 하는 경우 portal로 렌더링 ㄱㄱ
- HTML이 비어있으면 앱의 JS 코드가 로드되고 실행될 때까지 사용자에게 빈 페이지가 표시되어서 느리게 느껴질 수 있다.
    - 이 문제를 해결하기 위해 서버에서 or 빌드 중에 컴포넌트로부터 초기 HTML을 생성할 수 있다.
    - 이런 최적화를 기본적으로 수행하는 프레임워크를 사용하는 것을 추천한다.
    - 실행 시점에 따라 이를 SSR(server-side rendering) 또는 SSG(static site generation)라고 한다.
    - SSR이나 SSG를 사용하는 앱은 createRoot 대신 hydrateRoot를 호출해야 한다.
        - 그러면 React는 DOM 노드를 파괴하고 재생성하는 대신 재사용(hydrate)한다.

<br/>

## 2) 부분적으로 React로 작성된 페이지 렌더링하기
- React가 관리하는 각 최상위 UI에 대한 루트를 생성하기 위해 `createRoot`를 여러 번 호출하고, 루트마다 `root.render`를 호출해서 각각 다른 컨텐츠를 표시할 수 있다.

```js
import './styles.css';
import { createRoot } from 'react-dom/client';
import { Comments, Navigation } from './Components.js';

const navDomNode = document.getElementById('navigation');
const navRoot = createRoot(navDomNode); 
navRoot.render(<Navigation />);

const commentDomNode = document.getElementById('comments');
const commentRoot = createRoot(commentDomNode); 
commentRoot.render(<Comments />);
```

- `document.createElement()`를 사용해서 새 DOM 노드를 생성하고 문서에 수동으로 추가할 수도 있다.

```js
const domNode = document.createElement('div');
const root = createRoot(domNode); 
root.render(<Comment />);
document.body.appendChild(domNode); // You can add it anywhere in the document
```

- 이 기능은 React 컴포넌트가 다른 프레임워크로 작성된 앱 내부에 있는 경우에 유용하다.

<br/>

## 3) 루트 컴포넌트 업데이트하기
- 같은 루트에서 `render`를 여러 번 호출할 수 있다.
- 컴포넌트 트리 구조가 이전 렌더링과 일치하면 React는 기존 state를 유지한다.
를 볼 수 있는 괴상한 예시 👇

```js
const root = createRoot(document.getElementById('root'));

let i = 0;
setInterval(() => {
  root.render(<App counter={i} />);
  i++;
}, 1000);
```

이딴식으로 계속 다시 render를 호출해도 컴포넌트 안의 state가 유지된다.

- 하지만 이렇게 render를 여러 번 호출하는 경우는 드문 일이다. 일반적으로는 컴포넌트가 state를 업데이트 한다.

<br/>

# 3. Troubleshooting 
## 1) `Target container is not a DOM element` 오류
- `createRoot`에 전달한 것이 DOM 노드가 아닐 때 나는 오류이다.
- 잘 모르겠으면 콘솔에 로깅해보자

```js
const domNode = document.getElementById('root');
console.log(domNode); // ???
const root = createRoot(domNode);
root.render(<App />);
```

예상되는 이유
- id를 잘못 적었거나
- HTML에서 `<script>` 태그는 그 뒤에 나타나는 DOM 노드를 볼 수 없음


## 2) `Functions are not valid as a React child.` 오류
- `root.render`에 전달한 것이 React 컴포넌트가 아닐 때 나는 오류

## 3) 서버에서 렌더링된 HTML이 처음부터 다시 생성됨
- 서버에서 렌더링된 앱은 `createRoot` 대신 `hydrateRoot` 써야 함!
- 앱이 서버에서 렌더링되어 React에 의해 생성된 초기 HTML을 포함하는 경우, `root.render`를 호출하는 과정에서, 모든 HTML이 삭제되고 모든 DOM 노드가 처음부터 다시 생성된다.
- 이렇게 하면 속도가 느려지고, 포커스와 스크롤 위치가 재설정되며, 그 밖의 다른 사용자 입력들도 손실될 수 있다.
