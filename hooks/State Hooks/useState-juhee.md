# useState

useState는 컴포넌트에 state 변수를 추가할 수 있게 해주는 훅이다.

```js
const [state, setState] = useState(initialState);
```

# 1. Reference
## 1-1. Parameters
### initialState
- 초기에 state를 설정할 값
- 모든 데이터 타입이 허용된다.
- 이 인자는 초기 렌더링 이후에는 무시된다.
- 함수를 전달하면 초기화 함수로 취급한다. 이 함수는 순수해야 하고 인자를 받을 수 없으며 반드시 값을 반환해야 한다. 이 함수는 컴포넌트를 초기화할 때 호출되고 반환값이 state로 저장된다.

## 1-2. Returns
### state
- 현재 state

### set Function
- state를 다른 값으로 업데이트하고 리렌더링을 촉발할 수 있는 설정자 함수
- 매개변수로 state가 될 값을 받는다.

## 1-3. 주의사항
### 컴포넌트의 최상위 레벨에서만 작성하자
- useState는 훅이므로 컴포넌트의 최상위 레벨이나 직접 만든 훅에서만 호출할 수 있다.
- 반복문이나 조건문 안에서는 호출할 수 없으니, 이런 로직이 필요하다면 새 컴포넌트로 빼내고 그 안에 작성해야 한다.

### 초기화 함수를 순수하게 작성하자
- Strict Mode가 켜져있으면 의도치 않은 불순물을 찾기 위해서 초기화 함수가 두 번 호출된다. 초기화 함수가 순수하게 잘 작성했다면 동작에 영향을 미치지 않는다. 두 번의 호출 중 하나의 결과만 반영되고 나머지는 무시된다.


set Function에 대해 더 자세히 ㄱㄱ

# 2. set Function
## 2-1. Parameters
next state 혹은 updater Function이 들어올 수 있다.

### next state
- 다음 state가 될 값
- 인자로 함수를 전달하면, 업데이터 함수로 취급된다.

### updater Function
- 역시나 순수해야 한다.
- 대기 중인 state를 유일한 인수로 사용한다. 
인수의 이름은 state 변수 이름의 첫글자로 지정하는 것이 일반적이다. `age -> a`
- 다음 state가 될 값을 반환해야 한다.
- 반환값은 없다.
- updater 함수는 대기열에 들어가고 컴포넌트가 리렌더링 된다. 다음 렌더링 중에 대기열에 있는 모든 updater를 이전 state에 적용되고 다음 state가 계산된다.

## 2-2. Returns
- 없음

## 2-3. 주의사항
- **다음 렌더링**에 대한 state만 업데이트 한다.
즉, set 함수를 호출해도 이미 실행 중인 코드의 현재 state는 변경되지 않는다.

```js
function handleClick() {
  setName('Robin');
  console.log(name); // Still "Taylor"!
}
```

- Object.is로 판단했을 때 동일한 값이 들어오면 리렌더링이 일어나지 않는다.
- state 업데이트는 일괄처리된다.
- 모든 이벤트 핸들러가 실행되고 set 함수를 호출한 후에 화면을 업데이트한다.
- 단일 이벤트 중에 여러 번 리렌더링 되는 것을 방지하기 위함이다.
- DOM에 접근하기 위해서 바로 리렌더링을 일으켜야 한다면, `flushSync`를 쓰면 된다.
- 렌더링 도중 set함수를 호출하는 것은 현재 렌더링 중인 컴포넌트 내에서만 허용된다.
- 렌더링 도중 set 함수가 호출되면 해당 출력은 버려지고 즉시 새로운 state로 다시 렌더링을 시도한다. 이 패턴은 드물게 사용되지만, 이전 렌더링에서의 정보를 저장하는 데 사용할 수 있다.

# 3. Usage
## 3-1. 객체 및 배열 state 업데이트 
- React에서 state는 읽기 전용이므로, 기존 객체를 **변이하지 않고 교체**해야 한다.

```js
// ❌
obj.x = 10;
setObj(obj);
// ⭕️
setObj({
  ...obj,
  x: 10
});

// ❌
setTodos(prevTodos => {
  prevTodos.push(createTodo());
});
// ⭕️
setTodos(prevTodos => {
  return [...prevTodos, createTodo()];
});
```

## 3-2. 초기 state 다시 생성하지 않기 

초기 state는 한 번 저장되고 다음 렌더링부터는 무시된다.

```js
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos());
  // ...
```

여기서 `createInitialTodos()`의 결과는 초기 렌더링에서만 사용되지만,
여전히 매 렌더링마다 호출은 된다.
이러면 낭비가 발생하겠지!

이를 해결하려면 useState에 호출 결과가 아닌 함수 자체를 전달하면 된다.

```js
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  // ...
```

## 3-3. key로 state 재설정하기

- 목록을 렌더링할 때 쓰는 key 속성은 컴포넌트의 state를 재설정하는 데에도 쓸 수 있다.
- key가 변경되면 컴포넌트가 다시 생성되므로 state가 초기화된다.

```js
export default function App() {
  const [version, setVersion] = useState(0);

  function handleReset() {
    setVersion(version + 1);
  }

  return (
    <>
      <button onClick={handleReset}>Reset</button>
      <Form key={version} />
    </>
  );
}
```

## 3-4. 이전 렌더링에서 얻은 정보 저장하기 

props 변경 등 렌더링에 대한 응답으로 state를 조정해야 하는 경우에 대해 알아보자!!

🤫 But, 대부분의 경우 이 기능은 필요하지 않다.
- 필요한 값을 현재 props나 다른 state에서 계산해서 얻을 수 있다면, state를 제거하고 계산해서 사용한다.
이때 너무 자주 재계산하는 것이 걱정된다면 `useMemo`를 이용하도록!

카운터가 변경된 후 증가했는지 감소했는지를 표시하는 예시를 보자.

```js
export default function CountLabel({ count }) {
  const [prevCount, setPrevCount] = useState(count);
  const [trend, setTrend] = useState(null);
  if (prevCount !== count) {
    setPrevCount(count);
    setTrend(count > prevCount ? 'increasing' : 'decreasing');
  }
```

- 렌더링하는 동안 set 함수를 호출할 때는 이렇게 조건 안에 있어야 한다. 그렇지 않으면 무한 렌더링에 빠지게 되니까ㅠㅠ
- 렌더링 중에 다른 컴포넌트의 set 함수를 호출해서는 안 된다.
- 이 패턴은 이해하기 어려울 수 있으며 피하는 것이 가장 좋지만, Effect에서 업데이트 하는 것보다는 낫다 ㅎㅎ
- 렌더링 도중 set 함수가 호출되면 컴포넌트는 return문으로 종료된 직후 자식을 렌더링 하기 전에 컴포넌트를 리렌더한다.
이렇게 하면 자식 컴포넌트를 두 번 렌더링할 필요가 없고, 나머지 컴포넌트 함수는 계속 실행되고 결과는 버려진다.
- 조건이 모든 훅 호출보다 아래에 있으면 early return을 통해 렌더링을 더 일찍 다시 시작할 수 있다.

# 4. Troubleshooting
## 4-1. Too many re-renders
- 무한 루프를 방지하기 위해 렌더링 횟수가 제한되어 있다.
- 원인을 찾기 위해 콘솔에서 에러 옆에 있는 화살표를 눌러 JS 스택에서 에러의 원인이 되는 특정 set 함수를 찾아보자!
- 아래 예시처럼 이벤트 핸들러 지정에서 실수로 발생하는 경우가 많다.
핸들러에 함수를 전달할 때는 호출이 아닌 함수 자체를 전달하거나 인라인 함수로 작성하도록!!

```js
// 🚩 Wrong: calls the handler during render 
return <button onClick={handleClick()}>Click me</button>

// ✅ Correct: passes down the event handler
return <button onClick={handleClick}>Click me</button>

// ✅ Correct: passes down an inline function
return <button onClick={(e) => handleClick(e)}>Click me</button>
```

## 4-2. 함수를 state로 설정하기
state의 값으로 함수를 설정하려고 할 때, 아래처럼 작성하면 설정은 안되고 대신 호출이 된다.

```js
const [fn, setFn] = useState(someFunction);

function handleClick() {
  setFn(someOtherFunction);
}
```

- 이렇게 함수를 전달하면 someFunction은 초기화 함수로, someOtherFunction은 업데이터 함수가 된다.
그래서 이들을 호출해서 결과를 저장하려고 시도한다.

- 이게 아니라 정말로 함수를 저장하려면 함수 앞에 `() =>` 를 넣어주면 된다.
이렇게 하면 전달한 함수가 값으로 저장된다.

```js
const [fn, setFn] = useState(() => someFunction);

function handleClick() {
  setFn(() => someOtherFunction);
}
```
