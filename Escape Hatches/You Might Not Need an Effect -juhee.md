## Effect가 필요하지 않은 경우
- 렌더링을 위해 데이터를 변환하는 경우
- 사용자 이벤트를 처리하는 경우

한편, 외부 시스템과 동기화를 하려면 Effect가 필요하다.

## props 또는 state에 따라 업데이트하기

아래의 코드는 useEffect가 필요 없다.

```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // ❌ 중복 state 및 불필요한 Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}
```

기존의 props나 state에서 계산할 수 있으면 아래처럼 렌더링 중에 계산하면 되고, state에 넣을 필요도 없다.

```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ Good: 렌더링 과정 중에 계산
  const fullName = firstName + ' ' + lastName;
  // ...
}
```

## 고비용 계산 캐싱하기

useMemo 훅으로 감싸서 캐시(memoize)할 수 있다.

```jsx
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = useMemo(() => {
    // ✅ Does not re-run unless todos or filter change
    // ✅ todos나 filter가 변하지 않는 한 재실행되지 않음
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
  // ...
}
```

아래처럼 한 줄로 바꿀 수도 있다.

```jsx
  const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]);
```

### 계산이 비싼지 확인하는 방법

```jsx
console.time('filter array');
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd('filter array');
```

## prop이 변경되면 모든 state 재설정하기

나쁜 예시
기존 값으로 먼저 렌더링한 다음 새로운 값으로 렌더링하기 때문에 비효율적이다.

```jsx
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 Avoid: Resetting state on prop change in an Effect
  // 🔴 이러지 마세요: prop 변경시 Effect에서 state 재설정 수행
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```

대신 명시적인 키를 전달한다.
키가 변경되면 React가 다른 컴포넌트로 인식한다!

```jsx
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ key가 변하면 이 컴포넌트 및 모든 자식 컴포넌트의 state가 자동으로 재설정됨
  const [comment, setComment] = useState('');
  // ...
}
```

## props가 변경될 때 일부 state만 조정하기

역시나 Effect를 쓰지 말고 렌더링 중에 직접 state를 조정한다.

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: 렌더링 중에 state 조정
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

렌더링 중에 컴포넌트를 업데이트하면 React는 반환된 JSX를 버리고 즉시 렌더링을 다시 시도한다.
이를 피하기 위해서 위 코드에서 `if (items !== prevItems)` 조건을 넣은 것이다.

이러한 패턴이 Effect보다는 효율적이지만, 대부분의 컴포넌트에는 필요하지 않다.
어떤 방법을 쓰든 props나 다른 state를 바탕으로 state를 조정하면 데이터 흐름을 이해하고 디버깅하기 어려워질 것이다.

key로 모든 state를 재설정하거나 렌더링 중에 모두 계산하는 방식이 좋다.

## 이벤트핸들러 간 로직 공유

아래 코드에서의 Effect는 버그를 촉발할 수 있다.
페이지를 불러올 때 isInCart가 true가 되므로
페이지를 새로고침 할 때마다 알림이 계속 등장할 것이다.

```jsx
function ProductPage({ product, addToCart }) {
  // 🔴 Avoid: Event-specific logic inside an Effect
  // 🔴 이러지 마세요: Effect 내부에 특정 이벤트에 대한 로직 존재
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```

## 연쇄 계산
state를 바탕으로 다른 state를 조정하는 Effect를 연쇄적으로 사용하고 싶을 경우
오직 서로를 촉발하기 위해서만 state를 조정하는 Effect 체인은 비효율적이다!!
불필요한 리렌더링이 더 발생할 수 있다.

렌더링 중에 가능한 것을 계산하고 이벤트 핸들러에서 state를 조정하는 것이 좋다.

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ Calculate what you can during rendering
  // ✅ 가능한 것을 렌더링 중에 계산
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // ✅ Calculate all the next state in the event handler
    // ✅ 이벤트 핸들러에서 다음 state를 모두 계산
    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount <= 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }
  ```
  
  이렇게 Effect 체인을 촉발시키지 않는 것이 훨씬 더 효율적이다.
  
## 애플리케이션 초기화하기

최상위 컴포넌트의 Effect에 앱이 로드될 때 한 번만 실행되어야 하는 로직을 배치하고 싶은 경우

```jsx
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ Only runs once per app load
      // ✅ 앱 로드당 한 번만 실행됨
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
```

React에는 외부 저장소를 구독하기 위해 특별히 제작된 훅이 있습니다. Effect를 삭제하고 useSyncExternalStore호출
