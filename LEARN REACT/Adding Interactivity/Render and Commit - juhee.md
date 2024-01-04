# 3. 렌더링 & 커밋

컴포넌트를 화면에 표시하기 위해서는 그 전에 React에서 렌더링을 해줘야 한다.
React 렌더링 동작에 대해서 알아보자!!

React에 UI를 요청하고 제공 받기 위해서는 아래의 단계를 거치게 된다.

![](https://velog.velcdn.com/images/e_juhee/post/ec57920d-8fb3-442c-97b7-f1dd5cf3411a/image.png)

## 3-1. Trigger a render : 렌더링을 촉발시킨다
👩‍🍳 손님의 주문을 주방으로 전달한다.

컴포넌트 렌더링이 일어나는 데에는 두 가지 이유가 있다.

### 1) 첫 렌더링인 경우
- 앱을 시작하기 위해서는 첫 렌더링을 촉발시켜야 한다.
- 대상 DOM 노드로 `createRoot`를 호출한 다음 컴포넌트로 `render` 메서드를 호출한다. 
(프레임워크나 샌드박스를 쓰면 코드상에서는 보이지 않을 수 있다.)

```js
import Image from './Image.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'))
root.render(<Image />);
```

### 2) state 또는 상위 요소가 업데이트 된 경우
- set 함수로 state를 업데이트해서 추가 렌더링을 촉발시킬 수 있다.
- 컴포넌트의 state를 업데이트하면 자동으로 렌더링이 대기열에 추가된다.

## 3-2. Render : 컴포넌트가 렌더링된다
👩‍🍳 주방에서 음식을 준비한다.

- 렌더링을 촉발시키면 React는 컴포넌트를 호출하여 화면에 표시할 내용을 파악한다.
- 즉, **렌더링은 React에서 컴포넌트를 호출하는 것**이다.
- 첫 렌더링에서는 루트 컴포넌트를 호출한다.
- 이후 렌더링에서는 state 업데이트에 의해 렌더링을 트리거한 함수 컴포넌트를 호출한다.
- 이 과정은 재귀적이다: 업데이트된 컴포넌트가 다른 컴포넌트를 반환하면 React는 그 컴포넌트를 다음에 렌더링하고, 그 컴포넌트 역시 무언가를 반환하면 그 컴포넌트를 다음에 렌더링하는 식으로 재귀적으로 프로세스를 수행한다.
- 언제까지 재귀? 중첩된 컴포넌트가 더 이상 존재하지 않을 때까지, 그리고 React가 화면에 표시해야 할 내용을 정확히 파악할 때까지 계속된다.
- 리렌더링하는 동안 React는 이전 렌더링 이후 변경된 속성을 계산한다. But, 다음 단계인 커밋 단계까지는 이 정보로 **아무런 작업도 수행하지 않는다**!!

### 1) 렌더링은 순수해야 한다 (feat. Strict Mode)
- 동일한 입력, 동일한 출력: 동일한 입력이 주어지면 항상 동일한 JSX를 반환해야 한다는 뜻!
- 자기 일에만 집중한다: 렌더링 전에 존재했던 객체나 변수를 변경해서는 안 된다.
- 그렇지 않으면 혼란스러운 버그와 예측할 수 없는 동작이 발생할 수 있다.
- Strict Mode에서 개발하면 각 컴포넌트의 함수가 두 번씩 호출되어 순수하지 않은 함수로 인한 실수를 알아차리는 데 도움이 된다.

### 2) 성능 최적화
- 업데이트된 컴포넌트 내에 중첩된 모든 컴포넌트를 렌더링하는 이 기본 동작은 업데이트된 컴포넌트가 트리에서 매우 높은 위치에 있는 경우 성능에 좋지 않다.
- 성능 문제가 발생하는 경우 [memo](https://react.dev/reference/react/memo#skipping-re-rendering-when-props-are-unchanged)를 통해 문제를 해결할 수 있다. But, 조급하게 최적화하려 서두를 필요는 없다.

## 3-3. Commit : DOM에 변경사항을 커밋한다
👩‍🍳 테이블에 주문한 요리를 내놓는다.

- 컴포넌트를 렌더링(호출)한 후 React는 DOM을 수정한다.
- 초기 렌더링인 경우 DOM API인 `appendChild()`를 사용해서 생성한 모든 DOM 노드를 화면에 표시한다.
- 리렌더링인 경우 필요한 최소한의 작업(렌더링하는 동안 계산된 것)을 적용하여 DOM이 최신 렌더링 출력과 일치하도록 한다.
- 렌더링 간에 차이가 있는 부분만 DOM 노드를 변경한다. 즉, 컴포넌트가 리렌더링되어도 변경될 필요 없는 부분의 값들은 리셋되지 않는다.

#### Browser paint
- 렌더링이 완료되고 DOM을 업데이트한 후 브라우저는 화면을 다시 그리는데, 이 단계를 브라우저 렌더링이라고 한다.
- 렌더링과의 혼동을 피하기 위해 이 포스팅에서는 브라우저 렌더링을 페인팅이라고 부를 것이다.

<br/>