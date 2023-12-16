# useReducer

reducer를 추가하는 훅

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
- 첫 번째 렌더링 중에는 init 유무에 따라서 `init(initialArg)` 또는 `initialArg`

### dispatch function
- state를 다른 값으로 업데이트하고 리렌더링을 촉발할 수 있는 dispatch function

이 dispatch function에 대해서 더 자세히 ㄱㄱ

# 2. dispatch function

- useReducer가 반환하는 dispatch 함수를 사용하면 state를 다른 값으로 업데이트하고 다시 렌더링을 촉발할 수 있다.
- dispatch 함수에 유일한 인수로 액션을 전달해야 한다.

```js
const [state, dispatch] = useReducer(reducer, { age: 42 });

function handleClick() {
  dispatch({ type: 'incremented_age' });
  // ...
```

- React는 reducer 함수에 현재 state와 dispatch한 액션을 전달하고, 그 결과를 다음 state로 설정한다.


# 3. Usage

# 4. Troubleshooting
