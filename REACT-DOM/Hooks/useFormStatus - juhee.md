# useFormStatus

💡 현재는 Canary와 experimental channel에서만 사용할 수 있다.

- 마지막 양식 제출의 status 정보를 제공하는 Hook이다.

```js
const { pending, data, method, action } = useFormStatus();
```

# 1. Reference
## useFormStatus() 
- 아래 예제에서처럼 status **정보를 얻을(이 hook을 호출할) 컴포넌트가 `<form>` 내부에 렌더링되어야** 한다.

```js
import { useFormStatus } from "react-dom";
import action from './actions';

function Submit() {
  const status = useFormStatus();
  return <button disabled={status.pending}>Submit</button>
}

export default function App() {
  return (
    <form action={action}>
      <Submit />
    </form>
  );
}
```

### Parameters
- 없음

### Returns
#### 1) pending
- 제출중이면 true
- 아니면 false

#### 2) data
- form이 제출하는 데이터를 포함하는 [FormData 인터페이스](https://developer.mozilla.org/en-US/docs/Web/API/FormData)를 구현한 객체
- 활성화된 제출이 없거나 `<form>` 부모가 없으면 null이 된다.

#### 3) method
- get 또는 post (string)
- HTTP 메소드 중 어떤 것으로 제출되는지 알려준다.
- (기본적으로 `<form>`은 GET 메소드를 사용하고, method 속성을 통해 지정할 수 있다.)

#### 4) action
- 부모 `<form>`의 action 속성에 전달된 함수에 대한 참조
- action 속성에 URI값이 제공되었거나 지정되지 않았으면 null이 된다.

# 2. 사용법
## 2-1. 대기 상태 표시하기
- pending 꺼내와서 분기

```js
function Submit() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? "Submitting..." : "Submit"}
    </button>
  );
}
```

## 2-2. 제출된 form 데이터 읽기
- data 꺼내와서 읽기
- data를 활용해서 이미 제출한 데이터를 보여줄 수 있다. 예시 👇

```js
import {useState, useMemo, useRef} from 'react';
import {useFormStatus} from 'react-dom';

export default function UsernameForm() {
  const {pending, data} = useFormStatus();

  const [showSubmitted, setShowSubmitted] = useState(false);
  const submittedUsername = useRef(null);
  const timeoutId = useRef(null);

  useMemo(() => {
    if (pending) {
      submittedUsername.current = data?.get('username');
      if (timeoutId.current != null) {
        clearTimeout(timeoutId.current);
      }

      timeoutId.current = setTimeout(() => {
        timeoutId.current = null;
        setShowSubmitted(false);
      }, 2000);
      setShowSubmitted(true);
    }
  }, [pending, data]);

  return (
    <>
      <label>Request a Username: </label><br />
      <input type="text" name="username" />
      <button type="submit" disabled={pending}>
        {pending ? 'Submitting...' : 'Submit'}
      </button>
      {showSubmitted ? (
        <p>Submitted request for username: {submittedUsername.current}</p>
      ) : null}
    </>
  );
}
```
