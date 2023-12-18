
# 2. ref로 DOM 조작하기

React는 DOM을 자동으로 업데이트하므로 컴포넌트가 DOM을 자주 조작할 필요는 없지만,
때로 노드에 포커싱하거나 스크롤, 크기나 위치 측정을 위해 DOM 요소에 접근해야 할 수도 있다.
React에는 이러한 작업을 수행할 수 있는 내장 기능이 없으므로 DOM 노드에 대한 ref가 필요하다.

## 2-1. 노드에 대한 ref 가져오기

DOM 노드를 가져올 JSX 태그에 `ref` 속성으로 참조를 전달한다.

```jsx
<div ref={myRef}>
```

처음에는 myRef.current는 null이 된다.
그리고 이 div에 대한 DOM 노드가 생성되면 이 노드에 대한 참조가 myRef.current에 담긴다.

## 2-2. ref 콜백

ref가 몇 개 필요한지 모를 때!
아래 예시는 목록의 각 항목에 ref를 전달하려 하고 있다.

```jsx
<ul>
  {items.map((item) => {
    // Doesn't work! 작동하지 않습니다!
    const ref = useRef(null);
    return <li ref={ref} />;
  })}
</ul>
```

But, 훅은 컴포넌트의 최상위 레벨에서만 호출해야 하기 때문에 useRef를 반복문 안에서 호출할 수는 없다.

이를 해결하기 위해 ref 속성에 함수를 전달하는 ref 콜백을 사용할 수 있다.

```jsx
import { useRef } from 'react';

export default function CatFriends() {
  const itemsRef = useRef(null);

  function scrollToId(itemId) {
    const map = getMap();
    const node = map.get(itemId);
    node.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      // Initialize the Map on first usage.
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToId(0)}>
          Tom
        </button>
        <button onClick={() => scrollToId(5)}>
          Maru
        </button>
        <button onClick={() => scrollToId(9)}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul>
          {catList.map(cat => (
            <li
              key={cat.id}
              ref={(node) => {
                const map = getMap();
                if (node) {
                  map.set(cat.id, node);
                } else {
                  map.delete(cat.id);
                }
              }}
            >
              <img
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

const catList = [];
for (let i = 0; i < 10; i++) {
  catList.push({
    id: i,
    imageUrl: 'https://placekitten.com/250/200?image=' + i
  });
}
```

## 2-3. 다른 컴포넌트의 DOM 노드에 접근하기

`<input/>` 같은 브라우저 요소를 출력하는 빌트인 컴포넌트에 ref를 넣으면 잘 작동하지만,
컴포넌트에 ref를 넣으려고 하면 null이 반환되고 원하는 동작을 하지 않는다.
콘솔에 이런 오류도 뜸 👇

```
Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?

Check the render method of `MyForm`.
    at MyInput
    at MyForm (https://1e4ad8f7.sandpack-bundler-4bw.pages.dev/App.js:24:38)
```

이유는
기본적으로 컴포넌트가 다른 컴포넌트의 DOM 노드에 접근하는 것을 허용하지 않기 때문이다.

대신, 자신의 DOM 노드를 공개하고 싶은 컴포넌트는 자신의 자식 중 하나에게 ref를 전달한다고 `forwardRef`로 명시할 수 있다.

컴포넌트를 forwardRef를 사용해서 선언하면, props 다음의 두 번째 ref 인자로 전달 받은 ref를 받도록 설정된다.
```jsx
import { forwardRef, useRef } from 'react';

const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

### 내부 DOM 노드에 대한 접근 제한하기

부모로부터 받은 ref에 대한 사용자 정의 인터페이스 제공하기

위 예시에서, 부모 컴포넌트가 MyInput에 focus()를 호출할 수 있게 되었다.
그런데 이렇게 하면 부모 컴포넌트가 MyInput에 대해 다른 작업도 할 수 있게 된다.
그렇지 못하게 제한하고 싶다면 `useImperativeHandle`을 사용하면 된다.

```jsx
const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // Only expose focus and nothing else
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});
```

이렇게 하면 부모 컴포넌트에 대한 ref 값으로 특별한 객체를 제공하도록 React에 지시해서
focus 메서드만 가지게 만든다.
이때 ref 핸들은 DOM 노드가 아닌, `useImperativeHandle` 내부에서 생성한 사용자 정의 객체다.

이를 통해 명시적인 메서드나 속성만을 제공할 수 있다.

## 2-4. ref는 언제 첨부되나요?

React에서 모든 업데이트는 두 단계로 나뉜다.

- 렌더링: 렌더링하는 동안 컴포넌트를 호출하여 화면에 무엇이 표시되어야 하는지 파악한다.
- 커밋: 커밋하는 동안 DOM에 변경 사항을 적용한다.

첫 번째 렌더링을 할 때는 DOM 노드들이 아직 생성되지 않았기 때문에 ref.current는 null이 된다.
그리고 업데이트의 렌더링 중에는, DOM 노드들이 아직 업데이트되지 않았기 때문에 아직 읽기에는 너무 이르다.

따라서 React는 commit 중에 ref.current를 설정한다!
DOM을 업데이트한 후, 즉시 ref.current 값을 해당 DOM 노드로 설정한다.

보통 이벤트 핸들러에서 ref에 접근하는데, 마땅한 특정 이벤트가 없다면 Effect가 필요할 수 있다.


### flushSync
flushSync는 React의 DOM 렌더러 내에서 제공하는 API로, 동기적으로 상태 업데이트를 강제로 수행할 때 사용된다.
일반적인 React의 상태 업데이트는 비동기적으로 동작하는데, 상태 변경을 즉시 반영하고 싶을 때 flushSync를 사용한다.

React에서는 state 업데이트가 큐에 등록되므로 DOM을 즉시 업데이트하지 않아 종종 문제가 발생할 수 있다.
목록에 새 항목을 추가하고 마지막 항목으로 스크롤하려면 아래 예시처럼 flushSync를 사용해서 동기적으로 업데이트를 해줘야 원하는 동작이 가능하다.

```js
flushSync(() => {
  setTodos([ ...todos, newTodo]);
});
listRef.current.lastChild.scrollIntoView();
```

## 2-5. ref를 이용한 DOM 조작 모범 사례

포커싱이나 스크롤처럼 비파괴적인 동작에는 문제가 없다.
그러나 DOM을 수동으로 수정하려 하면 React가 수행하는 변경 사항과 충돌할 위험이 있다..!!

예를 들어, DOM 엘리먼트를 수동으로 제거(`remove()`)하고 setState를 통해 다시 표시하려고 하는 경우 충돌이 발생한다.

따라서 React가 관리하는 DOM 노드를 변경할 때는 주의가 필요하다.
위 상황과 달리 React가 업데이트할 이유가 없는 DOM 요소는 안전하게 수정할 수 있다.
