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
## state와 ref의 차이점
- ref는 변경되어도 리렌더를 촉발하지 않는다. 따라서 ref는 컴포넌트의 UI에 영향을 미치지 않는 정보를 저장하는 데 적합하다.

### ref를 사용하면...
- 렌더링할 때마다 재설정되는 일반 변수와 달리 정보가 저장된다.
- 정보가 공유되는 외부 변수와 달리 각 컴포넌트에 로컬로 저장된다.

### ref를 읽거나 쓰려면
컴포넌트는 순수 함수처럼 동작해야 하므로,
렌더링 중에 ref를 읽거나 쓰면 안된다!

이렇게 하면 안됨

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

렌더링 중에 뭔가를 읽거나 써야만 한다면, ref 대신 state를 쓰자!
