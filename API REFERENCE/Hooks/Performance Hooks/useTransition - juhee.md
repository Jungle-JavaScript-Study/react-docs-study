# useTransition
UI를 차단하지 않고 state를 업데이트할 수 있는 Hook이다.

```js
const [isPending, startTransition] = useTransition()
```

# 1. Reference
## 1-1. Parameters
- 없음

## 1-2. Returns
### isPending
- isPending: 보류 중인 트랜지션이 있는지 여부를 알려주는 플래그

### startTransition
- startTransition: state 업데이트를 트랜지션으로 표시할 수 있는 함수
- 트랜지션으로 표시한다는 것은 이 업데이트가 React의 우선순위 큐에서 보다 낮은 우선순위를 갖게 되는 것을 의미한다. 즉 덜 긴급한 업데이트가 되는 것!
- 하나 이상의 set 함수를 호출해서 일부 state를 업데이트하는 함수 `scope`를 매개변수로 받는다.
- 리액트는 이 `scope` 함수를 매개변수 없이 즉시 호출하고, 이 함수 호출 중에 동기적으로 예약된 모든 state 업데이트를 트랜지션으로 표시한다.
- 이는 논블로킹이고, 원치 않는 로딩을 표시하지 않는다.
- 반환하는 값은 없다!

## 1-3. 주의사항
- useTransition은 훅이므로 컴포넌트나 커스텀 훅 내부에서만 호출할 수 있다. 다른 곳에서 트랜지션을 시작해야 한다면 startTransition을 사용하자! (근데 얘는 isPending 없음 쩔수탱 ㅜ)
- state의 set 함수에 접근할 수 있는 경우에만 업데이트를 트랜지션으로 감쌀 수 있다. 일부 prop이나 커스텀 훅 값에 대한 응답으로 트랜지션을 시작하려면 useDeferredValue를 쓰자.
- 트랜지션으로 표시된 state 업데이트는 다른 state 업데이트에 의해 중단된다. 예를 들어, 트랜지션 내에서 차트 컴포넌트를 업데이트한 다음, 차트가 다시 렌더링되는 도중에 입력을 시작하면 React는 입력 업데이트를 처리한 후 차트 컴포넌트에서 렌더링 작업을 다시 시작한다.
- 진행 중인 트랜지션이 여러 개 있는 경우, React는 현재 트랜지션을 함께 일괄 처리한다.


# 2. Usage

## 2-1. state 업데이트를 논블로킹 트랜지션으로 표시하기

- 트랜지션을 사용하면 느린 디바이스에서도 사용자 인터페이스 업데이트의 반응성을 유지할 수 있다.
- 트랜지션을 사용하면 리렌더링 도중에도 UI가 반응성을 유지한다! 예를 들어, 탭을 클릭했다가 다른 탭을 클릭하면 첫 번째 리렌더링이 완료될 때까지 기다리지 않고 바로 두 번째 클릭한 다른 탭이 클릭되게 할 수 있다.

```js
function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

## 2-2. input에 트랜지션 사용하기
- input을 제어하는 state 변수에는 트랜지션을 사용할 수 없다.
- 트랜지션은 논블로킹이지만 변경 이벤트에 대한 응답으로 input을 업데이트하는 것은 동기적으로 이루어져야 하기 때문이다.
- input에 대한 응답으로 트랜지션을 실행하려면 두 가지 옵션이 있다.
- 1) 항상 동기적으로 업데이트되는 input의 state와 트랜지션 실행 시 업데이트할 state 변수를 각각 선언한다. 이를 통해 동기 state를 사용해 input을 제어하고, 트랜지션 state 변수를 나머지 렌더링 로직에 전달한다.
- 2) 하나의 state 변수를 가지고, 실제 값보다 지연되는 useDeferredValue를 추가한다. 그러면 새로운 값을 자동으로 따라잡기 위해 논블로킹 리렌더를 촉발한다.

## 2-3. startTransition에 전달하는 함수는 동기식이어야 한다

- startTransition에 전달하는 함수는 동기 함수여야 한다. 
- React는 이 함수를 즉시 실행하여 실행하는 동안 발생하는 모든 state 업데이트를 트랜지션으로 표시한다. 
- 아래와 같이 타임아웃 등으로 수행시킨 비동기 작업 완료 후의 상태 업데이트는 트랜지션으로 표시되지 않는다.

```js
startTransition(() => {
  // ❌ Setting state *after* startTransition call
  setTimeout(() => {
    setPage('/about');
  }, 1000);
});

setTimeout(() => {
  startTransition(() => {
    // ✅ Setting state *during* startTransition call
    setPage('/about');
  });
}, 1000);
```

```js
startTransition(async () => {
  await someAsyncFunction();
  // ❌ Setting state *after* startTransition call
  setPage('/about');
});

await someAsyncFunction();
startTransition(() => {
  // ✅ Setting state *during* startTransition call
  setPage('/about');
});
```
