# useContext
컴포넌트에서 context를 읽고 구독할 수 있게 해주는 hook이다.

# 1. Reference
## 1-1. Parameters
### SomeContext
- `createContext`로 생성한 context
- context 자체에 정보를 가지고 있는 것은 아니고, 컴포넌트에서 제공하거나 읽을 수 있는 정보의 종류를 나타낸다.

## 1-2. Returns
### context
- 컴포넌트에 대한 context 값을 반환한다.
- 호출한 컴포넌트에서 트리상 상위에 있는 가장 가까운 SomeContext.Provider에 전달된 value이다.
- provider가 없는 경우에는 createContext에 전달한 defaultValue가 된다.
- 항상 최신 값이 반환된다.
- context가 변경되면 리액트는 context를 읽는 컴포넌트를 자동으로 리렌더링한다.

## 1-3. 주의사항
- `<Context.Provider>`는 useContext를 호출하는 컴포넌트의 상위에 위치해야 한다. 동일한 컴포넌트에서 반환된 provider의 영향은 받지 않는다.
- 변경된 value를 받는 provider부터 시작해서 해당 context를 사용하는 자식들이 전부 자동으로 리렌더된다.
- symlinks(심볼릭 링크)를 사용하는 등 빌드 시스템이 출력에 중복 모듈을 생성하는 경우 cotnext가 손상되어 값을 읽지 못할 수 있다. context를 통한 전달은 context를 제공하는 데 사용하는 SomeContext와 context를 읽는 데 사용하는 SomeContext가 정확하게 동일한 객체인 경우에만 작동한다.
이를 확인하는 방법은 `window.SomeContext1`와 `window.SomeContext2`를 전역에 할당하고 콘솔에서 `window.SomeContext1 === window.SomeContext2`가 true인지 확인하면 된다. 동일하지 않은 경우 빌드 도구 수준에서 해당 문제를 해결해야 한다.

# 2. 사용법
## 2-1. 데이터 전달하기
- useContext를 호출해서 context를 읽고 구독한다.
- 전달한 context에 대한 context 값을 반환받는다.
- 리액트는 context 값을 결정하기 위해서 컴포넌트 트리를 검색하고, 특정 context에 대해 상위에 존재하는 가장 가까운 provider를 찾는다.

```js
function MyPage() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  );
}
```

- 이렇게 Provider로 감싸진 `<Form />`과 그 자식들은 useContext를 통해 context 값을 읽을 수 있다.

## 2-2. 데이터 업데이트하기
- context를 업데이트하려면 state와 결합해야 한다.
- 부모 컴포넌트에 state를 만들고 이 state를 context 값으로 provider에 전달한다.

```js
  const [theme, setTheme] = useState('dark');
  return (
    <ThemeContext.Provider value={theme}>
      <Form />
    </ThemeContext.Provider>
  );
```

- Provider 내부의 모든 컴포넌트는 현재 theme 값을 받게 된다.
- setTheme으로 theme 값이 바뀌면 Provider 내부의 모든 컴포넌트가 리렌더링된다.

## 2-3. fallback 기본값 지정하기
- 부모 트리에 provider가 존재하지 않으면, useContext는 context를 생성할 때 지정한 기본값이 반환된다.

```js
const ThemeContext = createContext('light');
```

## 2-4. 트리 일부에 대한 context 재정의하기
- 재정의할 트리 일부분을 다른 값의 provider로 감싸주면 해당 부분만 재정의할 수 있다.
- provider는 필요한 만큼 중첩해서 재정의 가능

```jsx
<ThemeContext.Provider value="dark">
  ...
  <ThemeContext.Provider value="light">
    <Footer />
  </ThemeContext.Provider>
  ...
</ThemeContext.Provider>
```

## 2-5. 객체 및 함수 전달 시 리렌더링 최적화

context로 객체와 함수를 전달하는 예시를 보자

```js
function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  function login(response) {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }

  return (
    <AuthContext.Provider value={{ currentUser, login }}>
      <Page />
    </AuthContext.Provider>
  );
}
```

- 전달되고 있는 객체와 함수는 MyApp이 리렌더링될 때마다 다른 객체가 되어 useContext를 호출하는 트리의 모든 컴포넌트가 리렌더된다.
- 이를 방지하기 위해 useCallback, useMemo를 사용하자

```js
import { useCallback, useMemo } from 'react';

function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(() => ({
    currentUser,
    login
  }), [currentUser, login]);

  return (
    <AuthContext.Provider value={contextValue}>
      <Page />
    </AuthContext.Provider>
  );
}
```

