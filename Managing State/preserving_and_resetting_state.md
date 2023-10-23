# 4. State 보존 및 재설정
## 4-1. UI 트리
- 브라우저는 UI를 모델링하기 위해 많은 트리 구조를 사용한다.
(DOM: HTML 요소, CSSOM: CSS 요소, 접근성 트리: HTML 요소에 대한 정보와 관련된 접근성)
- 리액트도 트리 구조를 사용한다.
- JSX로부터 UI 트리를 만들고, React DOM은 이 트리 구조를 바탕으로 브라우저의 DOM 요소를 업데이트한다.

![](https://velog.velcdn.com/images/e_juhee/post/13f761fe-0975-489e-b489-6c91409eab06/image.png)

## 4-2. State는 트리 내부에서 관리된다
- state는 컴포넌트 내에 존재하는 것이 아니고, React가 보유하고 있는 것이다.
그렇겠지.. 같은 컴포넌트를 여러 개 불러와도 각각 상태는 독립적이잖아
- UI 트리 내에서 컴포넌트의 위치를 기반으로 컴포넌트와 state를 연결시킨다.

- 같은 컴포넌트를 같은 위치에 렌더링하는 한 상태를 계속 유지한다.
즉, 컴포넌트의 렌더링을 중지하면(제거하면) 상태도 사라진다. 다시 불러오면 초기값으로 불러온다.

## 4-3. 동일한 위치의 동일한 컴포넌트는 State를 유지한다

```jsx
<div>
  {isFancy ? (
    <Counter isFancy={true} /> 
  ) : (
    <Counter isFancy={false} /> 
  )}
</div>
```

위 경우 Counter 컴포넌트는 항상 div의 첫 번째 자식이므로, isFancy의 값이 바뀌어도 State가 재설정되지 않는다.

## 4-4. 동일한 위치의 다른 컴포넌트는 State를 초기화한다

```jsx
<div>
  {isPaused ? (
    <p>See you later!</p> 
  ) : (
    <Counter /> 
  )}
</div>
```

위치는 동일하지만 서로 다른 유형의 컴포넌트로 전환되므로 State가 소멸된다.
이렇게 다른 컴포넌트를 렌더링하면 하위 트리 전부 State가 재설정된다.

### 그래서 컴포넌트를 중첩해서 정의하면 안 된다

```jsx
import { useState } from 'react';

export default function MyComponent() {
  const [counter, setCounter] = useState(0);

  function MyTextField() {
    const [text, setText] = useState('');

    return (
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
    );
  }

  return (
    <>
      <MyTextField />
      <button onClick={() => {
        setCounter(counter + 1)
      }}>Clicked {counter} times</button>
    </>
  );
}
```

이렇게 컴포넌트 안에 컴포넌트를 정의해두면
매번 새로운 자식 컴포넌트(MyTextField)가 생성되는 것이므로, 자식 컴포넌트의 상태가 유지되지 않는다.
같은 위치에 다른 컴포넌트를 렌더링하는 것이므로 State가 초기화된다.
따라서 컴포넌트를 항상 최상위 수준에서 선언해야 한다.

## 4-5. 동일한 위치에서 State 재설정하기

```jsx
{isPlayerA ? (
  <Counter person="Taylor" />
) : (
  <Counter person="Sarah" />
)}
```

동일한 Counter라는 컴포넌트를 동일한 위치에서 사용하지만 각각 별도의 상태를 가지게 하고싶다면!


### 1) 컴포넌트를 다른 위치에 렌더링한다.

```jsx
{isPlayerA &&
  <Counter person="Taylor" />
}
{!isPlayerA &&
  <Counter person="Sarah" />
}
```

`isPlayerA`가 true일 때는 첫 번째 Counter가, false일 때는 두 번째 Counter가 보여지게 된다.

![](https://velog.velcdn.com/images/e_juhee/post/8134394c-f6d8-4e81-8d16-f8859d10fab3/image.png)


### 2) key로 명시적인 아이덴티티를 부여한다.

```jsx
{isPlayerA ? (
  <Counter key="Taylor" person="Taylor" />
) : (
  <Counter key="Sarah" person="Sarah" />
)}
```

이것이 더 일반적인 방법!

목록을 렌더링할 때 사용하는 key를 사용해서 리액트가 컴포넌트를 구분하게 할 수 있다.
기본적으로 부모 내의 순서에 따라 컴포넌트를 구분하지만
key를 명시하면 순서 대신 key 자체를 사용해서 위치를 결정한다.
따라서 같은 위치에 렌더링하더라도 다른 컴포넌트로 인식한다.

주의: 키는 전역으로 고유하지는 않고 오직 부모 내에서의 위치를 지정하는 데 사용된다.

### 3) 제거된 컴포넌트의 State 보존하기
- 간단한 UI일 경우, 모두 렌더링시켜놓고 제거시키지 않고 CSS로 숨긴다.
- state를 끌어올려서 보관한다. -> 이것이 가장 일반적인 해결책
- localStorage 등을 이용한다.
