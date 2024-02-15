# useFormState
💡 현재는 Canary와 experimental channel에서만 사용할 수 있다.
💡 useFormState을 사용하는 이점을 얻으려면, React 서버 컴포넌트를 지원하는 프레임워크에서 사용해야 한다.

- form 액션의 결과에 기반하여 상태를 업데이트할 수 있게 해주는 Hook이다.

```js
const [state, formAction] = useFormState(fn, initialState, permalink?);
```

# 1. Reference
```js
useFormState(action, initialState, permalink?) 
```

- 컴포넌트 최상위 레벨에서 useFormState를 호출해서, form 액션이 호출될 때 업데이트되는 state를 생성한다.
- useFormState에 form의 액션 함수와 초기값을 전달하면, form에 사용할 새로운 액션과 최신 state를 반환한다.
- 최신 state는 제공한 함수에도 전달된다.

```js
import { useFormState } from "react-dom";

async function increment(previousState, formData) {
  return previousState + 1;
}

function StatefulForm({}) {
  const [state, formAction] = useFormState(increment, 0);
  return (
    <form>
      {state}
      <button formAction={formAction}>Increment</button>
    </form>
  )
}
```

- form의 state는 form이 마지막으로 제출되었을 때 액션에 의해 반환된 값이다.
- form이 아직 제출되지 않았다면, 초기값을 가진다.
- 서버 액션을 사용하면 form을 제출하고 나서 서버의 응답을 hydration이 완료되기 전에 보여줄 수 있다.

## 1-1. Parameters
### fn
- form이 제출될 때 호출할 함수
- 초기 인자로 form의 이전 state를 받는다. 
- 그 다음에는 form 액션이 일반적으로 받는 인자들을 받는다.

### initialState
- State의 초기값
- 처음 호출된 후에는 무시된다.

### permalink (optional)
- form이 수정하는 고유 페이지 URL을 포함하는 문자열
- 동적 콘텐츠를 포함하는 페이지(ex. 피드)에서 점진적 개선을 위해 사용된다.
- 만약 fn이 **서버 액션**이고, JS 번들이 로드되기 전에 폼이 제출되면, 브라우저는 현재 페이지의 URL이 아닌 **지정된 permalink URL로 이동**한다.
- 당연히,, 이 페이지에서도 동일한 form 컴포넌트가 렌더링되도록 해놔야하고, 동일한 액션 fn 및 permalink도 포함시켜야 한다. 👉 React가 상태를 어떻게 전달해야 하는지 알 수 있게 하기 위함
- form이 hydrate되면, 이 매개변수는 더 이상 효과가 없다.

## 1-2. Returns
- 두 개의 값을 가지는 배열을 반환한다.

### current state
- 액션에 의해 반환된 값
- 첫 렌더링에서는 전달한 initialState

### new action
- form 컴포넌트의 액션 속성으로, 또는 form 내의 버튼 컴포넌트의 formAction 속성으로 전달할 수 있는 새로운 액션

## 1-3. 주의사항
### 서버 컴포넌트와 사용
- React 서버 컴포넌트를 지원하는 프레임워크와 사용하면, 클라이언트에서 JS가 실행되기 전에 form을 interactive하게 만들 수 있다.

### 서버 컴포넌트 없이 사용
- 컴포넌트의 로컬 state와 동일하다. (form의 상태가 컴포넌트 내에서만 유지되고 관리되며, form 상태와 관련된 로직은 클라이언트에서 JS가 로드되고 실행된 후에 처리된다.)

### 함수 시그니처의 차이
- 전달된 함수는 첫번째 인자로 이전 state를 받는다. (기존 form 함수는 보통 이벤트 객체만 받음)
- 이는 함수가 useFormState 없이 직접 form 액션으로 사용될 때와 다른 시그니처(인자의 구성)을 갖게 만든다.
- 이로인해 특정 조건 하에만 State를 업데이트하는 등 세밀한 제어가 가능해진다.

# 2. 사용법

- useFormState에서 반환 받은 formAction은 `<form>`의 action prop으로 전달된다.

```js
import { useFormState } from 'react-dom';
import { action } from './actions.js';

function MyComponent() {
  const [state, formAction] = useFormState(action, null);
  // ...
  return (
    <form action={formAction}>
      {/* ... */}
    </form>
  );
}
```

- form이 제출되면 인자로 전달한 action 함수가 호출되고, 이 함수의 반환 값이 form의 새로운 state가 된다.
- 제공한 액션은 첫 번째 인자로 currentState를 받고, 나머지 인자들은 useFormState를 사용하지 않았을 때와 동일하다.

```js
function action(currentState, formData) {
  // ...
  return 'next state';
}
```

