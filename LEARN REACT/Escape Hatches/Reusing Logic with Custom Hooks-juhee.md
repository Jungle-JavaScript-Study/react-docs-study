React에서 제공해주는 빌트인 훅 이외에 좀 더 구체적인 목적을 위한 훅이 필요한 경우가 있다.
나만의 훅을 만들어보자!

# 1. Custom Hooks: 컴포넌트간의 로직 공유

사용자가 앱을 사용하는 동안 네트워크 연결이 끊어지면 사용자에게 주의를 주는 컴포넌트를 만들어보자.

이 컴포넌트에 필요한 hook
- 네트워크가 온라인 상태인지 여부를 담는 State
- 전역 onlie/offline 이벤트를 구독하고 state를 업데이트하는 Effect

```jsx
import { useState, useEffect } from 'react';

export default function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}
```

이 로직을 다른 컴포넌트에서도 쓰려면?
UI는 다를지라도 네트워크 연결 상태를 확인하는 로직은 재사용할 수 있을 것이다.

# 2. 커스텀 훅 추출하기

훅을 직접 만들어보자
`useOnlineStatus` 함수를 선언하고 재사용할 로직을 이 함수로 옮긴다.

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}
```

이렇게 하면 반복 없이 로직의 재사용이 가능해진다.
더 중요한 것은, 컴포넌트 내부의 코드가 how가 아닌 what을 설명한다는 것이다 !!
커스텀 훅을 만들면 외부 시스템이나 브라우저 API를 처리하는 방법에 대한 지저분한 세부 사항을 숨길 수 있어서
코드가 구현이 아닌 의도를 표현하게 할 수 있다. 👍

# 3. 훅의 이름은 언제나 use로 시작
React 애플리케이션은 컴포넌트로 빌드된다.
컴포넌트는 빌트인 훅이든 커스텀 훅이든 상관 없이 훅으로 빌드된다.
명명 규칙을 반드시 따라야 한다.

#### 명명 규칙
1. React 컴포넌트의 이름은 대문자로 시작해야 한다. 또한 JSX를 반환해야 한다.
2. 훅의 이름은 use로 시작해야 하고, 그 다음의 첫글자는 대문자여야 한다. 임의의 값을 반환할 수 있다.
(당연히,, 훅을 호출하지 않는 함수는 use 안 써도 됨)

#### 🧐 Why?
이 규칙은 컴포넌트를 보고 state, Effect 등의 React 기능이 어디에 숨어 있는지 항상 알 수 있도록 보장한다.
예를 들어, 컴포넌트 내부에 `useOnlineStatus()`와 같은 함수는 내부에서 다른 훅을 호출하고 있을 가능성이 높다.

덧붙여, [React용으로 구성된 Linter](https://react-ko.dev/learn/editor-setup#linting)는 이 명명 규칙이 적용되어 있다.
use로 시작하지 않는 함수 안에서는 훅을 호출할 수 없다.
오직 훅과 컴포넌트만이 다른 훅을 호출할 수 있다.

원칙적으로, 다른 훅을 호출하지 않는 훅을 만들 수는 있다.
하지만 이러한 패턴은 혼란을 야기하므로 피하도록 하자.
나중에 훅을 추가할 계획이 있는 경우에만 use 접두사를 사용하도록 하자.

# 4. 커스텀 훅은 state 자체가 아닌 상태적인 로직(stateful logic)을 공유한다.

동일한 커스텀 훅을 여러곳에서 쓴다고 해서, 그 안에 있는 State를 공유하는 것은 아니다!!
각각의 훅 호출은 동일한 훅에 대한 다른 모든 호출과 완전히 독립적이다.
state를 공유하려면 state를 끌어올려서 전달해서 사용하자.

# 5. 훅 사이에 반응형 값 전달하기

컴포넌트를 다시 렌더링할 때마다 커스텀 훅 내부의 코드가 다시 실행된다.
따라서 컴포넌트와 마찬가지로 커스텀 훅도 순수해야 한다.
(커스텀 훅의 코드를 컴포넌트 본문의 일부로 생각하자)

커스텀 훅은 컴포넌트와 함께 다시 렌더링되기 때문에 항상 최신 props와 state를 받는다.
이에 대한 예시를 살펴보자.
아래의 컴포넌트는 서버 URL 또는 선택한 채팅방을 변경한다.

아래는 `serverUrl`이나 `roomId`이 변할 때마다 Effect가 재동기화하는 커스텀 훅이다.

```jsx
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      showNotification('New message: ' + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

아래의 컴포넌트는 props인 `roomId`와 state인 `serverUrl`의 변화에 반응한다.

```jsx
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });

  return (
    <>
      <label>
        Server URL:
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}
```

# 6. 커스텀 훅에 이벤트 핸들러 전달하기

> 🚨 이 섹션에서 설명하는 API는 아직 안정된 버전의 React로 출시되지 않았다.

위에서 살펴본 `useChatRoom` 훅을 다시 살펴보자.

현재는 메시지가 도착했을 때 무엇을 해야 하는지에 대한 로직이 하드코딩 되어 있다.

```jsx
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
// 👇 여기부터
    connection.on('message', (msg) => {
      showNotification('New message: ' + msg);
    });
// 👆 여기까지
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

이 로직을 컴포넌트 안으로 이동시켜보자!

컴포넌트에서 훅에 이벤트를 전달한다.

```jsx
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
    onReceiveMessage(msg) {
      showNotification('New message: ' + msg);
    }
  });
  // ...
```

커스텀 훅에서 이벤트를 사용한다. (의존성 배열에도 추가해줘야 한다.)

```jsx
export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      onReceiveMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl, onReceiveMessage]); // ✅ All dependencies declared
}
```

이렇게만 해도 작동하지만, 커스텀 훅이 이벤트 핸들러를 수락할 때 한 가지 더 개선할 수 있는 부분이 있다.
의존성 배열에 `onReceiveMessage`를 추가하므로 컴포넌트가 다시 렌더링될 때마다 채팅이 다시 연결된다 ㅠㅠ

이 이벤트 핸들러를 **Effect Event**로 감싸 의존성에서 제거하자!

```jsx
export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  const onMessage = useEffectEvent(onReceiveMessage);

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      onMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
}
```

# 7. 커스텀 훅 언제 쓰나요?

중복되는 모든 코드를 커스텀 훅으로 추출할 필요는 없다. (약간의 중복은 괜찮다.)
단일 useState 호출을 감싸기 위해 커스텀 훅을 만드는 것은 불필요할 수 있다.

하지만, Effect를 작성할 때는 매번 커스텀 훅으로 감싸는 것이 더 명확할지 고려해야 한다.
Effect를 자주 사용하는 것은 옳지 않으므로, 만약 Effect를 작성한다는 것은 외부 시스템과 동기화하거나 React에 빌트인 API가 없는 작업을 수행하기 위해 React 외부로 나가야 하는 경우일 것이다.
이러한 Effect를 커스텀 훅으로 감싸면 의도와 데이터 흐름 방식을 정확하게 전달하는 데 도움이 된다.

데이터 흐름을 명시적으로 만드는 예시를 살펴보자.
외부 api로부터 두 개의 목록을 받아오는 컴포넌트를 만들어보자.

```js
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]);

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]);
```

위 코드는 반복되는 내용이 많지만, Effect는 서로 분리하여 유지하는 것이 맞다.
서로 다른 두 가지를 동기화하기 때문이다.
대신, 공통 로직을 커스텀 훅으로 추출하여 단순화할 수 있다.

```jsx
function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    if (url) {
      let ignore = false;
      fetch(url)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setData(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [url]);
  return data;
}
```

이제 위에 추출해 낸 `useData` 커스텀 훅을 사용하여 아래처럼 간결하게 쓸 수 있다.

```jsx
function ShippingForm({ country }) {
  const cities = useData(`/api/cities?country=${country}`);
  const [city, setCity] = useState(null);
  const areas = useData(city ? `/api/areas?city=${city}` : null);
  // ...
```

이렇게 커스텀 훅을 추출해내면 데이터 흐름을 명시적으로 만들 수 있다.
`url을 입력하면 data를 가져온다.` -> 아주 명시적이죠?

이렇게 useEffect를 숨기면 `ShippingForm` 컴포넌트에서 불필요한 의존성을 추가하는 것을 방지할 수 있다.
이상적으로는 시간이 지나면 앱의 Effect 대부분이 커스텀 훅에 포함될 것이다!

## 7-1. 커스텀 훅은 구체적인 고수준 사용 사례에 집중해야 한다.

useEffect API 자체에 대한 대안 및 편의 래퍼 역할을 하는 커스텀 생명주기 훅 같은 건 만들지 말자..

```
이런것들 ㄴㄴ
❌ useMount(fn)
❌ useEffectOnce(fn)
❌ useUpdateEffect(fn)
```

Why?
이런 커스텀 생명주기 훅은 React 패러다임에 잘 맞지 않는다..
저따위 훅을 만들면 의존성 배열에 넣어야 할 것을 빼먹어도 linter가 경고를 해주지 못한다. (linter는 직접적인 useEffect 호출만 확인한다.)
그러니 Effect를 쓸 거면 그냥 useEffect를 가져다 쓰자.

좋은 커스텀 훅은 호출 코드가 수행하는 작업을 제한하여 보다 선언적으로 만든다.
커스텀 훅이 추상적일 경우, 장기적으로 많은 문제를 야기할 가능성이 높다.

## 8. 커스텀 훅은 더 나은 패턴으로 마이그레이션하는데 도움을 준다.

Effect는 탈출구이다. (React를 벗어나야 할 때, 쓸만한 빌트인 솔루션이 없을 때 사용한다.)
React팀의 장기적인 목표는 앱에서 useEffect와 같은 훅의 사용을 최소화하는 것이다.
이는 더 구체적인 문제에 대해 더 구체적인 해결책을 제공함으로써 달성될 수 있다.
Effect를 커스텀 훅으로 감싸두면 이런 솔루션이 제공될 때 코드를 더 쉽게 업그레이드할 수 있다.

### 커스텀 훅으로 Effect를 감싸는 것이 좋은 이유를 정리해보자면,
1. Effect와의 데이터 흐름을 명확하게 만들 수 있다.
2. 컴포넌트가 Effect의 정확한 구현 방법보다는 의도에 집중할 수 있다.
3. 리액트가 새로운 기능을 추가할 때, 컴포넌트 코드에 손을 대지 않고 Effect를 쉽게 제거할 수 있다.

디자인 시스템을 만드는 것처럼 앱의 컴포넌트에서 공통 기능을 사용자 정의 훅으로 추출해서 컴포넌트의 코드가 의도에 집중할 수 있고 Effect를 자주 작성하는 것을 피할 수 있다.

## 8-1. 데이터 페칭 빌트인 솔루션
아직 작업중임!
아래와 같이 데이터 페칭을 할 수 있게 될 것이다.

```jsx
import { use } from 'react'; // Not available yet! 
                             // 아직 동작하지 않습니다!
function ShippingForm({ country }) {
  const cities = use(fetch(`/api/cities?country=${country}`));
  const [city, setCity] = useState(null);
  const areas = city ? use(fetch(`/api/areas?city=${city}`)) : null;
  // ...
```

이 솔루션이 적용되면 데이터 페칭하는 useEffect를 커스텀 훅으로 빼두는 것이 나중에 마이그레이션하는 데 도움이 될 것이다.

