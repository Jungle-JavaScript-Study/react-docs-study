# useCallback
렌더링 간에 함수를 캐싱

- input: 함수, 의존성 배열
- output: 첫 렌더링 시에는 처음에 전달한 함수를 리턴/이후에는 의존성이 바뀌지 않았으면(`Object.is`) 이전과 똑같은 함수, 바뀌었으면 이번 렌더링에서 전달한 함수 리턴

## 컴포넌트 리렌더링 건너뛰기
```javascript
function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => { 
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);  // 2. theme는 함수의 캐싱에 영향을 주지 않음

  return (
    <div className={theme}>  // 0. 컴포넌트 리렌더링 시 자식도 재귀적으로 렌더링 됨
      <ShippingForm onSubmit={handleSubmit} />  // 3. theme가 바뀌어도 리렌더링 되지 않음
    </div>
  );
}

const ShippingForm = memo(function ShippingForm({ onSubmit }) {  // 1. 이전 렌더링과 같은 props를 전달받으면 리렌더링을 건너뜀
  // ...
});
```

- `useCallback`은 성능 최적화를 위해서만 사용해야 함. `useCallback` 없이 코드가 돌아가지 않는다면 다른 문제.
- `useCallback`과 `useMemo`의 차이는 함수 자체를 캐시하는지, 호출한 함수의 결과를 캐시하는지의 차이

### useCallback은 언제 써야 할까?
일반적인 경우에는 `useCallback`을 사용할 이유가 없음.

#### `useCallback`을 사용해야 하는 경우:
 - `memo`로 감싼 컴포넌트에 props로 전달할 때
 - 전달하는 함수가 훅의 의존성으로 사용될 때

#### 이런 원칙을 지키면 메모화가 필요 없어짐:
  - 시각적으로 컴포넌트 내에 다른 컴포넌트가 있으면, JSX 자체를 자식으로 받게 한다. 그러면 래퍼 컴포넌트가 state를 업데이트 해도 자식은 리렌더링되지 않는다.
    ```javascript
    function Card({ children }) {
      return (
        <div className="card">
          {children}
        </div>
      );
    }
    ```
  - 최대한 상위 컴포넌트로 state를 끌어올리지 않는다. 로컬 상태값을 사용하고, 최상위나 전역 상태 라이브러리에 form이나 hover 여부 같은 일시적인 상태를 저장하지 않는다.
  - 렌더링 로직을 순수하게 유지한다. 리렌더링 시 문제가 생기면 메모화 대신 버그 수정이 필요한 것이다.
  - 상태값을 업데이트하기 위한 불필요한 Effect를 제거한다.
  - Effect에서 불필요한 의존성을 제거한다. 메모화 대신 객체/함수를 Effect 내부나 컴포넌트 위부로 이동하는 것을 고려한다.

## 메모된 콜백에서 state 업데이트하기
```javascript
// X: 가능한 적은 의존성을 가지는게 좋음
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos([...todos, newTodo]);
  }, [todos]);
}

// O: 업데이터 함수를 전달하면 todos에 대한 의존성이 필요없음
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos(todos => [...todos, newTodo]);
  }, []);
}
```

## Effect가 너무 자주 발동되지 않게 하기

```javascript
// createOptions는 함수라 렌더링될 때마다 변경되므로 Effect의 의미가 없음
useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]);

// useCallback으로 감싼 createOptions는 roomId 변경 시에만 새롭게 만들어짐
const createOptions = useCallback(() => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }, [roomId]);

// 가장 좋은 방법은 createOptions를 Effect 내부로 이동하는 것
```

## 커스텀 훅 최적화하기
커스텀 훅을 만들 때는 리턴하는 모든 함수를 useCallback으로 감싸는 것이 좋음. 훅을 사용하는 주체가 코드를 원하는 대로 최적화 할 수 있도록.

## 문제 1: 컴포넌트가 렌더링될 때마다 useCallback이 다른 함수를 리턴한다면?
- 의존성 배열을 잊지 않았는지 확인
- 의존성이 이전 렌더링과 달라지지 않았는지 확인

## 문제 2: 루프 안에서 useCallback을 사용하고 싶은데 허용되지 않는다면?
```javascript
// X: 루프 안에서는 useCallback 사용 불가
function ReportList({ items }) {
  return (
    <article>
      {items.map(item => {
        const handleClick = useCallback(() => {
          sendReport(item)
        }, [item]);

        return (
          <figure key={item.id}>
            <Chart onClick={handleClick} />
          </figure>
        );
      })}
    </article>
  );
}

// O: useCallback 사용하고 싶은 부분을 컴포넌트로 분리
function ReportList({ items }) {
  return (
    <article>
      {items.map(item =>
        <Report key={item.id} item={item} />
      )}
    </article>
  );
}

function Report({ item }) {
  const handleClick = useCallback(() => {
    sendReport(item)
  }, [item]);

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
}

// O(대안): 반복되는 컴포넌트 자체를 memo로 감싸면 props가 변경되지 않는 한 Report도 리렌더링되지 않으므로 Chart로 리렌더링 되지 않음
function ReportList({ items }) {
  // ...
}

const Report = memo(function Report({ item }) {
  function handleClick() {
    sendReport(item);
  }

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
});
```
