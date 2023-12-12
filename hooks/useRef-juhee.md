# useRef

```
useRef(initialValue) 
```

컴포넌트의 최상위 레벨에서 useRef를 호출해서 ref를 선언한다.

```
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  const inputRef = useRef(null);
  // ...
```

## Parameters

### initialValue
- ref 객체의 current 프로퍼티에 들어갈 초기 설정 값이다.
- 어떤 유형의 값이든 지정할 수 있다.
- 초기 렌더링 이후부터는 무시된다.

## Returns
- 단일 프로퍼티인 current를 가진 객체를 반환한다.
- current의 값은 처음에는 전달한 initialValue가 되고, 나중에 다른 값으로 바꿀 수 있다.
- 반환 받은 이 객체 ref를 JSX 노드의 ref 속성으로 전달하면 JSX 노드가 current에 담긴다.
- 다음 렌더링에서도 useRef는 동일한 객체를 반환한다.

# 2. 주의사항
- state와 달리 ref.current 프로퍼티는 변이가 가능하다.
단, 렌더링에 사용되는 객체(ex. state의 일부)를 포함하는 경우에는 변이해서는 안된다.
- ref.current가 변경되어도 리렌더가 일어나지 않는다.
- 초기화를 제외하고는 렌더링 중에 ref.current를 읽거나 쓰면 안된다. (컴포넌트의 동작을 예측할 수 없게 되기 때문)
- Stric Mode에서는 컴포넌트 함수가 두 번씩 호출되어 의도하지 않은 동작을 찾는 데 도움을 준다.
ref 객체도 두 번 생성되고 그 중 하나는 버려진다. 컴포넌트 함수는 순수해야 하므로 이것이 로직에 영향을 미치지는 않는다.

# 3. 사용법
## 3-1. ref를 사용하면 다른점
### state VS ref
- ref는 변경되어도 리렌더를 촉발하지 않는다. 따라서 ref는 컴포넌트의 UI에 영향을 미치지 않는 정보를 저장하는 데 적합하다.
### 일반 변수 VS ref
- 렌더링할 때마다 **재설정**되는 일반 변수와 달리 정보가 **유지**된다.
### 외부 변수 VS ref
- 정보가 공유되는 외부 변수와 달리 각 컴포넌트에 로컬로 저장된다.

## 3-2. ref에 올바르게 접근하기
컴포넌트는 순수 함수처럼 동작해야 하므로, 렌더링 중에 ref를 읽거나 쓰면 안된다!
컴포넌트 최상위 레벨에서 ref에 접근하는 코드가 있으면 안 된다는 뜻
(렌더링 중에 읽거나 써야만 한다면, ref 대신 state를 쓰자!)

👇 이렇게 하면 안됨

```
function MyComponent() {
  // ...
  // 🚩 Don't write a ref during rendering
  myRef.current = 123;
  // ...
  // 🚩 Don't read a ref during rendering
  return <h1>{myOtherRef.current}</h1>;
}
```

대신 **이벤트 핸들러**나 **useEffect** 안에서 읽거나 쓰자

```
function MyComponent() {
  // ...
  useEffect(() => {
    // ✅ You can read or write refs in effects
    myRef.current = 123;
  });
  // ...
  function handleClick() {
    // ✅ You can read or write refs in event handlers
    doSomething(myOtherRef.current);
  }
  // ...
}
```

## 3-3. DOM 조작
ref로 DOM을 조작하는 것은 일반적이다. React에는 이를 위한 빌트인 기능도 있다.

### 1) 초기값이 null인 ref 객체를 생성한다.

```jsx
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);
  // ...
```

### 2) 조작하려는 DOM 노드의 JSX에 ref를 전달한다.

```jsx
  return <input ref={inputRef} />;
```

### 3) 끝!
쓰면 됨
React가 DOM 노드를 생성하고 화면에 배치한 후, ref 객체의 current 속성이 이 DOM 노드로 설정된다.

```js
  function handleClick() {
    inputRef.current.focus();
  }
```

노드가 화면에서 제거되면 ref의 current는 다시 null이 된다.

## 3-4. Ref의 내용 재생성 피하기

- React는 초기에 ref 값을 한 번 저장하고 다음 렌더링부터는 이를 무시한다.

```jsx
function Video() {
  const playerRef = useRef(new VideoPlayer());
  // ...
```

이 코드에서 `new VideoPlayer()`의 결과는 초기 렌더링에만 사용되지만, 
**호출 자체는 이후의 모든 렌더링에서도 여전히 계속 이뤄진다.**

useRef가 초기값을 첫 마운트 때만 사용한다는 것은, 이 부분(`new VideoPlayer()`)이 호출되지 않는다는 뜻이 아니다!
렌더링마다 실행은 매번 되어 인스턴스를 매번 생성한다.
만들어놓고 쓰지는 않음..ㅋㅋ
생성하는 데 비용이 발생하므로 좋지 않다.

이 문제를 해결하려면 아래와 같이 초기화하면 된다.

```js
function Video() {
  const playerRef = useRef(null);
  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
  // ...
```

원래 이렇게 렌더링 중에 ref.current를 쓰거나 읽으면 안되지만, 이 경우에는 결과가 항상 동일하고 초기화 중에만 조건이 실행되어 충분히 예측이 가능하므로 괜찮다.

# Troubleshooting
## 커스텀 컴포넌트에 대한 ref 얻기

컴포넌트에 Ref를 전달하려고 아래와 같이 하면

```js
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

이런 오류가 발생한다..

```
Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?
```

왜냐하면,
기본적으로 컴포넌트는 내부의 DOM 노드에 대한 ref를 외부로 노출하지 않기 때문이다.

이 문제를 해결하려면 ref를 가져오려는 컴포넌트를 forwardRef로 감싸야한다.

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(({ value, onChange }, ref) => {
  return (
    <input
      value={value}
      onChange={onChange}
      ref={ref}
    />
  );
});

export default MyInput;
```

이렇게 하면 부모 컴포넌트가 Ref에 접근할 수 있게 된다. 👍
