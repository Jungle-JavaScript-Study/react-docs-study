
Effect를 특정 값의 변경에만 반응하도록 하는 방법을 알아보자


# 이벤트 핸들러 vs Effect

- 이벤트 핸들러는 특정 상호작용에 대한 응답으로 실행된다.
- Effect는 동기화가 필요할 때마다 실행된다.
	- props 또는 state와 같은 일부 값이 마지막 렌더링 때와 다른 값으로 변경되면 다시 동기화된다.
- 이벤트 핸들러는 수동으로 촉발되고 Effect는 자동으로 필요할 때 수행된다고 볼 수 있다.

### 반응형 값, 반응형 로직

#### 반응형 값
- 컴포넌트 내부에 선언된 props, state, 변수를 반응형 값이라고 한다.
- 이러한 반응형 값은 리렌더링으로 인해 변경될 수 있다.

#### 반응형 로직
- 이벤트 핸들러 내부의 로직은 반응형이 아니기 때문에 변경에 반응하지 않고 반응형 값을 읽을 수 있다.
- Effect 내부의 로직은 반응형으로, Effect에서 반응형 값을 읽는 경우에는 의존성으로 지정해줘야 한다. 의존성으로 지정한 값이 변경되면 Effect의 로직이 다시 수행된다.

# Effect에서 비반응형 로직 추출하기

반응형 로직과 비반응형 로직을 함께 사용하는 경우를 살펴보자.

사용자가 채팅에 연결할 때 현재 테마(dark / light)에 맞게 알림을 표시하는 상황을 가정해보자.


여기서 theme은 반응형 값이고,
Effect에서 읽는 모든 반응형 값은 의존성으로 선언해야 하므로 의존성 배열에 넣어줬다.

```js
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();
    return () => {
      connection.disconnect()
    };
  }, [roomId, theme]); // ✅ All dependencies declared
  // ...
```

❌ 이렇게 하면 theme만 변경되어도 채팅이 또다시 연결된다! 그러면 안되겠지!

즉, 알림을 띄우는 비반응형 로직 (`showNotification('Connected!', theme);`) 을 주변의 반응형 Effect로부터 분리해내야 한다.

## useEffectEvent

🚨 `useEffectEvent`는 아직 stable 버전의 React로 출시되지 않았다.

비반응형 로직을 useEffectEvent 훅에 넣어주자.

```js
import { useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });
  // ...
```

여기서 `onConnected`는 Effect Event라고 불린다.
Effect 로직의 일부이지만 이벤트 핸들러처럼 동작한다!
이 내부의 로직은 반응형으로 동작하지 않고, 항상 props와 state의 최신값을 확인한다.

이제 Effect 내부에서 `onConnected` Effect Event를 호출할 수 있다.

```js
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

Effect Event는 반응형 이벤트가 아니므로 Effect의 의존성 배열에 onConnected를 넣으면 안 된다는 것에 유의하자!

이 Effect Event는 이벤트 핸들러와 매우 유사하다.
촉발되는 원인이 사용자의 상호작용에 대한 응답이 아닌 Effect에서 사용자가 촉발한다는 점만 다르다.

이렇게 Effect Event를 사용하면 Effect의 반응성과 반응형으로 동작해서는 안 되는 코드를 분리할 수 있다.

## Effect Event로 최신 props와 state 읽기

Effect Event를 사용하면 의존성 관련 린트 경고를 무시하고자 하는 상황들을 해결할 수 있다.

페이지를 방문할 때마다 url과 장바구니에 있는 품목의 수를 기록하는 상황을 가정해보자.

```js
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems);
  }, [url]); // 🔴 React Hook useEffect has a missing dependency: 'numberOfItems'
  // ...
}
```

Effect 내부에서 numberOfItems를 사용하고 있어서 린터가 이것을 의존성에 추가하라고 한다.
그러나 numberOfItems가 변한다고 해서 logVisit이 호출될 필요는 없다.

useEffectEvent를 써서 코드를 분리해보자.

```js
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]); // ✅ All dependencies declared
  // ...
}
```

useEffectEvent 안에 있는 코드는 반응형이 아니기때문에, 로직 안에서 반응형 값(이 예제에서는 numberOfItems)을 사용할 수 있어졌다.

여기서 url을 onVisit의 인자로 전달해주지 않고 logVisit이 바로 읽도록 바꿔도 작동은 잘 된다.

```
  const onVisit = useEffectEvent(() => {
    logVisit(url, numberOfItems);
  });

  useEffect(() => {
    onVisit();
  }, [url]);
```

하지만 url을 Effect Event인 onVisit에 명시적으로 전달하는 것이 좋다.

Why???
onVisit이 url에 대해 반응하기를 원하는 것이기 때문!
기존 코드처럼 Effect Event가 명시적으로 url을 요청하면 (매개변수로 받으면)
Effect에서 전달해줘야 하니 의존성 배열에서 실수로 url을 빼먹을 일이 없다! (린트가 잡아주니까)

💡 Effect 내부에 **비동기 로직**이 있는 경우에 특히 중요하다.

```js
  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    setTimeout(() => {
      onVisit(url);
    }, 5000); // Delay logging visits
  }, [url]);
```

여기서 onVisit 내부의 url은 최신 url이지만 visitedUrl은 원래 이 Effect를 실행하게 만든 url이 그대로 담기게 될 것이다.

## Effect Event의 제한사항

- Effect 내부에서만 호출할 수 있다.
- 다른 컴포넌트나 hook에 전달할 수 없다.
- Effect Event는 Effect 코드의 비반응형 “조각”이므로, 이를 사용하는 Effect 옆에 있어야 한다.

## 그냥 의존성 린터 막으면 안됨?

이렇게 린트 규칙을 무시하는 코드를 종종 볼 수 있다 ㅋㅋ

```
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems);
    // 🔴 Avoid suppressing the linter like this:
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [url]);
  // ...
}
```

useEffectEvent가 안정적인 API가 된 후에는 린터를 억제하지 않는 것이 좋다.

이렇게 규칙을 막는 것의 단점은
코드에 도입한 새로운 반응형 의존성에 Effect가 반응해야 할 때 경고를 받지 못해 버그가 발생할 수 있다.
모든 반응형 값은 의존성으로 지정해줘야 한다.


