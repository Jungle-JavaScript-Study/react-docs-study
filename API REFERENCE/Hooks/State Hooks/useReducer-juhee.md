# useReducer

reducer를 추가하는 훅

# 0. 이거 왜 씀?
- 컴포넌트가 복잡해지면 state가 업데이트되는 다양한 경우를 한 눈에 파악하기 어려워진다.
setState를 어디서 하고 있는지 찾기 어렵다는 뜻
- 이런 경우, 컴포넌트 외부에서 모든 상태 업데이트 로직을 하나의 함수인 reducer에 집중시킬 수 있다.
- 즉, 많은 상태 업데이트와 이벤트 핸들러를 다루는 컴포넌트는 복잡해질 수 있으므로, 리듀서를 사용해서 상태 업데이트 로직을 단일 함수에서 관리하자
- 리듀서는 상태 업데이트 로직을 모아서 관리하므로 코드를 더 깔끔하고 관리하기 쉽게 만들어 준다.

### 단일 함수란?
- 코드의 재사용성과 유지 보수성 향상을 위해서 특정 기능이나 연산을 수행하는 데 필요한 로직을 하나의 함수로 통합하여 구성하는 것

### reducer 이름의 의미
- 배열 메서드인 `reduce()`에서 따서 지은 이름이다.
- `reduce()`는 배열의 값들을 하나의 값으로 누적한다.
- reduce에 전달하는 함수를 reducer라고 부르는데, 이 reducer는 지금까지의 결과와 현재 항목을 받아 다음 결과를 반환한다.
- React의 reducer도 지금까지의 상태와 액션을 받아서 다음 상태를 반환한다.

# 1. Reference

```js
useReducer(reducer, initialArg, init?) 
```

## 1-1. Parameters
### reducer
- state가 업데이트되는 방식을 지정하는 reducer 함수
- 순수 함수여야 한다.
- state와 액션을 인자로 받고, 다음 state를 반환한다.
- state와 액션은 어떤 유형이든 가능하다.

### initialArg
- 초기 state가 계산되는 값
- 어떤 유형이든 가능하다.
- 이 값에서 state를 계산하는 방법은 init 인자에 따라 달라진다.

### (optional) init
- 초기 state 계산 방법을 지정하는 초기화 함수
- 이를 지정하지 않으면 초기 state는 initialArg가 된다.
- 지정하면 `init(initialArg)` 호출 결과가 초기 state가 된다.


## 1-2. Return

두 개의 값을 가진 배열을 반환한다.

### current state
- 현재 state
- 첫 번째 렌더링 중에는 init 유무에 따라서 `init(initialArg)` 또는 `initialArg`

### dispatch function
- state를 다른 값으로 업데이트할 수 있는 함수

이 dispatch function에 대해서 더 자세히 ㄱㄱ

# 2. dispatch function

- useReducer의 두 번째 리턴값인 `dispatch 함수`를 사용하면 state를 다른 값으로 업데이트할 수 있다. (state가 업데이트 되니 리렌더링도 촉발됨!)
- dispatch 함수에 유일한 인수로 `action`을 전달해야 한다.

```js
const [state, dispatch] = useReducer(reducer, { age: 42 });

function handleClick() {
  dispatch({ type: 'incremented_age' });
  // ...
```

- React는 reducer 함수에 현재 `state`와 `dispatch한 액션`을 전달하고, 그 결과를 다음 state로 설정한다.

## Parameters
### action
- 유저가 수행한 작업을 나타내는 객체
- 어떤 데이터 유형이든 가능
- 관용적으로, 보통 type 속성이 있는 객체를 쓴다.
- 다른 속성을 추가해서 추가 정보를 포함해도 됨

## Returns
- 없음!

## 주의사항
- dispatch 함수는 다음 렌더링에 대한 state 변수만 업데이트한다.
따라서, dispatch 함수를 호출한 후 state를 읽으면 이전 값이 표시됨!
- 새로 제공한 값이 Object.is로 비교했을 때 state와 동일하다면 리렌더링을 하지 않는다. (최적화를 위해)
- 여기서도 state 업데이트는 일괄 처리된다.

# 3. Usage
## 3-1. 상태 업데이트하기

- 화면에 표시되는 내용을 업데이트하려면 useState를 쓸 때처럼 setState를 하는 것이 아닌, action을 사용하여 dispatch를 호출한다.

요렇게 👇

```js
function handleClick() {
  dispatch({ type: 'incremented_age' });
}
```

이렇게 하면
1) 현재 state와 액션이 reducer 함수에 전달되고, 
2) reducer가 다음 state를 계산해서 반환한다.
3) reducer가 반환한 값이 다음 state에 저장되고, 
4) 컴포넌트가 렌더링되어서
5) UI가 업데이트된다.

이렇게 useReducer는 useState와 매우 유사하지만, 이벤트 핸들러의 state 업데이트 로직을 컴포넌트 외부의 단일 함수로 옮길 수 있다는 점에서 차이가 있다.

## 3-2. useState VS useReducer

### 1) 코드 크기
- `useState`를 쓰면 초기에 작성해야 할 코드가 덜 필요하다. 간단
- `useReducer`는 `reducer 함수`와 `dispatch action`을 작성해야 한다.
- 여러 이벤트 핸들러에 걸쳐 비슷한 상태 수정 로직이 있다면, 그 로직을 `reducer`에 centralize함으로써 코드를 더 깔끔하게 유지할 수 있다.
즉, `useReducer`는 상태 관리 로직이 복잡하고 여러 이벤트 핸들러에서 상태를 수정할 때 유용하게 사용할 수 있다.

### 2) 가독성
- 상태 업데이트가 간단하면 `useState`가 가독성이 좋다.
- 상태 업데이트 로직이 복잡해지면, `useReducer`로 업데이트 로직이 어떻게 동작하는지(how)와 어떤 일이 일어났는지(what happened)를 깔끔하게 분리할 수 있다.
즉, `useReducer`는 복잡한 상태 업데이트 로직에서 코드의 가독성을 향상시킨다.

### 3) 디버깅
- `useState`로 버그가 발생하면, 어디서 잘못 설정되었는지와 이유를 파악하기 어려울 수 있다. 상태 변경이 여러 곳에서 일어날 수 있고 어디서 발생했는지 명확하지 않을 수 있기 때문!
- `useReducer`를 쓰면 리듀서에 콘솔 로그를 추가하여 모든 상태 업데이트와 그 이유(action)을 명확하게 알 수 있다. 
- 만약 각 action이 올바르다면, 오류는 reducer 로직 자체에 있는 것.. 하지만 이럴 경우 `useState`보다 더 많은 코드를 검토해야 할 수 있다.
즉, `useReducer`는 상태 변경 로직의 버그를 찾고 디버깅하기 더 쉽게 만들어주는 장점이 있지만, `useState`에 비해 더 많은 코드를 살펴봐야 할 수도 있다.

### 4) 테스트
- reducer는 컴포넌트에 의존하지 않는 순수 함수이기 때문에 따로 분리해서 독립적으로 테스트할 수 있다.
- 현실적인 환경에서 테스트하는 것이 좋긴 하지만, 복잡한 상태 업데이트 로직의 경우 초기 상태와 액션에 대해 리듀서가 특정 상태를 반환하는지 확인하는 것이 유용할 수 있다.

### 5) 개인 취향
- 취향 차이임. `useState`와 `useReducer`는 언제든 서로 변환해서 사용할 수 있으며, 동등하다.

## 3-3. reducer 함수 작성하기

reducer 함수는 아래와 같이 선언한다.

```js
function reducer(state, action) {
  // ...
}
```

- 이 안에서 다음 state를 계산해서 반환하는 코드를 작성하면 된다.
- 관례상 `switch문`으로 작성하는 것이 일반적이다.
- 이 reducer 함수에서 반환하는 값이 다음 state가 된다.
- state를 매개변수로 받는 함수이기 때문에 컴포넌트 밖에서 선언해도 되고, 심지어 다른 파일에 분리해놔도 괜찮다.

### 1) reducer는 순수하게 작성할 것
- reducer는 렌더링 도중에 실행되므로 순수해야 한다. 즉, 입력이 같으면 항상 동일한 출력을 생성해야 한다. 외부 세계에 영향을 주는 어떠한 작업도 수행해서는 안 된다.

#### reducer 안에서 하면 안 되는 것
- 네트워크 요청을 보내면 안 됨
- timeout 스케줄링하면 안 됨
- 객체 및 배열을 직접 수정하는 등의 작업 하면 안 됨

### 2) 각 액션은 여러 데이터를 변경하더라도 오직 하나의 유저 상호작용만을 설명하게 작성하자.
- 5개의 필드가 있는 폼에서 재설정을 눌렀을 때, 5개의 개별 액션이 아닌, 하나의 reset_form 액션을 전송하게 하는 것이 합리적이다. 
- 이렇게 작성하면 디버깅에 도움이 됨 👍

## 3-4. dispatch 함수 호출하기

- 액션 type의 이름은 컴포넌트에 로컬로 지정된다.


```js
function Form() {
  const [state, dispatch] = useReducer(reducer, { name: 'Taylor', age: 42 });
  
  function handleButtonClick() {
    dispatch({ type: 'incremented_age' });
  }

  function handleInputChange(e) {
    dispatch({
      type: 'changed_name',
      nextName: e.target.value
    });
  }
  // ...
```

## 3-5. 초기 state 재생성 방지하기

```js
  const [state, dispatch] = useReducer(reducer, createInitialState(username));
```

- state의 초기값은 (위 코드에서는 `createInitialState(username)`) 결과를 초기 렌더링에만 사용하지만, 이후의 모든 렌더링에서도 여전히 이 초기값을 얻는 함수를 호출은 한다.
필요 없는 계산을 하니까 낭비가 되겠지!
- 이 문제를 해결하려면 useReducer 세 번째 인수로 초기화 함수를 전달할 수 있다.

```js
  const [state, dispatch] = useReducer(reducer, username, createInitialState);
```

이렇게 하면 초기화 후에는 초기 state가 다시 생성되지 않는다.

- 위 코드에서는 `createInitialState`가 `usename`을 인수로 받는다.
- 인자가 필요하지 않다면 두 번째 인자로 null을 넘겨주면 된다.

## 3-6. Immer 활용
- `useImmerReducer`를 사용하면 `push` 또는 `arr[i] = ` 방식으로 state를 변이할 수 있다.
- Immer는 변경해도 안전한 특별한 `draft` 객체를 제공한다. 이 `draft` 객체에 변화를 적용하면, Immer는 내부적으로 상태의 복사본을 생성하고 `draft`에 만든 변경 사항을 적용한다.
- state를 반환할 필요가 없다.

# 4. Troubleshooting

## 다음 state 값이 필요한 경우
- reducer를 직접 호출해서 수동으로 계산할 수 있다.

```js
const action = { type: 'incremented_age' };
dispatch(action);

const nextState = reducer(state, action);
console.log(state);     // { age: 42 }
console.log(nextState); // { age: 43 }
```

## dispatch 후 state 일부분이 undefined가 될 때
- 새 state를 반환할 때 기존 필드를 모두 복사했는지(`...state`) 확인하자!

```js
    case 'incremented_age': {
      return {
        ...state, // Don't forget this!
        age: state.age + 1
      };
    }
```

## Too many re-renders
- 매 렌더링 시 무조건 액션을 dispatch하고 있는 경우에 발생할 수 있다.
그러면 렌더링이 일어나면 dispatch로 또 렌더링이 유발되고 또 dispatch... 이렇게 무한 루프에 빠지겠지
- 이벤트 핸들러 지정 시 실수하지 않았는 지 확인하자

```jsx
// 🚩 Wrong: calls the handler during render
return <button onClick={handleClick()}>Click me</button>

// ✅ Correct: passes down the event handler
return <button onClick={handleClick}>Click me</button>

// ✅ Correct: passes down an inline function
return <button onClick={(e) => handleClick(e)}>Click me</button>
```
