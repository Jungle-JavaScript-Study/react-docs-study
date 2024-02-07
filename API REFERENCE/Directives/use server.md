- 'use server'는 React Server Component를 사용할 경우에만 필요한 기능이다.
- client-side에서 호출할 수 있는 server-side 함수를 표시한다.
- next.js에서는 13.4.0 이상에서 사용할 수 있다.

# 1. Reference
- async 함수의 본문 맨 위에 'use server'를 추가해서 클라이언트에서 호출 가능하게 표시한다.
- 이런 함수를 Server Actions라고 한다.

```js
async function addToCart(data) {
  'use server';
  // ...
}
```

- 클라이언트에서 Server Action을 호출하면, 모든 인수의 직렬화된 복사본을 포함하는 네트워크 요청이 서버로 전송된다. 
- 서버 액션이 값을 반환하는 경우, 반환된 값도 직렬화되어 클라이언트로 반환된다.
- 위 예시처럼 각 함수에 'use server'를 표시하는 대신, 파일의 맨 위에 이 지시어를 추가하여 해당 파일 내의 모든 내보내기(exports)를 서버 액션으로 표시할 수 있다. 

# 2. 주의사항
- 'use server'를 모듈의 맨 처음에 작성할 경우, import문을 포함한 모든 코드보다 위에 있어야 한다. (주석은 ㄱㅊ)
- `''` `""` 가능 / `백틱` 불가능
- server-side에서만 사용할 수 있다.
- return값이 props를 통해 클라이언트 컴포넌트로 전달될 수 있으므로 직렬화 가능한 타입인지 확인해야 한다.
- 클라이언트에서 서버 액션을 import해서 가져오려면, 특정 함수가 아닌 모듈 수준에서 지시어를 사용해야 한다.
- 비동기 함수에서만 사용할 수 있다. (네트워크 호출이 항상 비동적이기 때문)
- 서버 액션으로 전달되는 인수는 신뢰할 수 없는 input으로 취급하고, 모든 변경 사항을 승인해야 한다. (아래의 Security considerations 참조!)
- 서버 액션은 transition에서 호출되어야 한다. (`<form action>`이나 `formAction`에 전달된 서버 액션은 자동으로 transition에서 호출되어서 상관x)
- 서버 액션은 서버 측 상태를 업데이트하는 변화를 위해서 설계된 것으로, 데이터 가져오기에는 권장되지 않는다. 따라서, 서버 액션을 구현하는 프레임워크는 일반적으로 한 번에 하나의 액션을 처리하며 반환 값에 대한 캐싱 방법이 없다.

## 2-1. Security considerations
- 서버 액션으로 전달되는 인수는 클라이언트에서 제어된다.
- 보안을 위해서 항상 이들을 **신뢰할 수 없는 입력**으로 취급하고, 인수를 검증하고 이스케이프해야 한다.
- 서버 액션에서는 항상 유저가 해당 액션을 수행할 권한이 있는지를 확인해야 한다.

# 3. 사용법

## 3-1. form의 Server Action
- 서버 액션의 가장 일반적인 사용은 데이터를 변경(mutate)하는 서버 함수를 호출하는 것이다. 
- React Server Components를 사용하면, form 내에서 서버 액션을 위한 일급 지원(first-class support)을 도입한다. 
- (일급 지원: 다른 기능들과 동등하게 활용할 수 있도록 지원하는 것)

다음은 사용자의 이름을 요청하는 예시이다.

```js
// App.js

async function requestUsername(formData) {
  'use server';
  const username = formData.get('username');
  // ...
}

export default function App() {
  return (
    <form action={requestUsername}>
      <input type="text" name="username" />
      <button type="submit">Request</button>
    </form>
  );
}
```


- 이 폼을 제출하면, requestUsername 서버 함수로 네트워크 요청이 발생한다.
- 폼에서 서버 액션을 호출할 때, React는 폼의 FormData를 서버 액션의 첫 번째 인수로 제공합니다.

폼 액션에 서버 액션을 전달함으로써, React는 폼을 점진적으로 향상시킬 수 있다.
👉 JavaScript 번들이 로드되기 전에도 폼을 제출할 수 있다는 뜻!

### form의 반환값 처리하기
- useFormState 훅을 사용해서 서버 액션의 결과에 따라 UI를 업데이트할 수 있다.

```js
// requestUsername.js
'use server';

export default async function requestUsername(formData) {
  const username = formData.get('username');
  if (canRequest(username)) {
    // ...
    return 'successful';
  }
  return 'failed';
}
```

```js
// UsernameForm.js
'use client';

import { useFormState } from 'react-dom';
import requestUsername from './requestUsername';

function UsernameForm() {
  const [returnValue, action] = useFormState(requestUsername, 'n/a');

  return (
    <>
      <form action={action}>
        <input type="text" name="username" />
        <button type="submit">Request</button>
      </form>
      <p>Last submission request returned: {returnValue}</p>
    </>
  );
}
```


## 3-2. form 외부에서 Server Action 호출하기
- 서버 액션은 공개된 서버 엔드포인트로, 클라이언트 코드 어디에서나 호출할 수 있다.
- 폼 외부에서 서버 액션을 사용할 때는 transition에서 서버 액션을 호출해야 한다.
- transition은 로딩 표시, optimistic state 업데이트, 예기치 않은 오류들의 처리가 가능하다.
```js
import incrementLike from './actions';
import { useState, useTransition } from 'react';

function LikeButton() {
  const [isPending, startTransition] = useTransition();
  const [likeCount, setLikeCount] = useState(0);

  const onClick = () => {
    startTransition(async () => {
      const currentCount = await incrementLike();
      setLikeCount(currentCount);
    });
  };

  return (
    <>
      <p>Total Likes: {likeCount}</p>
      <button onClick={onClick} disabled={isPending}>Like</button>;
    </>
  );
}
```

```js
// actions.js
'use server';

let likeCount = 0;
export default async function incrementLike() {
  likeCount++;
  return likeCount;
}
```
