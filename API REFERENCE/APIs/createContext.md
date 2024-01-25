### Reference

- <code>**createContext(defaultValue)**</code> 를 사용하면 컴포넌트가 제공하거나 읽을 수 있는 **컨텍스트**를 만들 수 있다.
    
    ```jsx
    import { createContext } from 'react';
    
    const ThemeContext = createContext('light');
    ```
    

### Parameters

- `defaultValue` : 컨텍스트를 읽는 컴포넌트 상위 트리에 일치하는 컨텍스트 provider 가 없을 때 컨텍스트의 값 ( 의미 있는 기본값이 없다면 `null` 로 설정 )

### Returns

- 컨텍스트 객체를 반환한다.
- **컨텍스트 객체 자체는 어떤한 정보도 보유하지 않는다.**
- 일반적으로, 상위 컴포넌트에서 `SomeContext.Provider` 를 사용해 컨텍스트 값을 지정하고, 하위 컴포넌트에서 `useContext(SomeContext)` 를 호출해 컨텍스트 값을 읽는다.
- `SomeContext.Provider` : 컴포넌트에 컨텍스트 값을 제공할 수 있다.
    
    ```jsx
    function App() {
      const [theme, setTheme] = useState('light');
      // ...
      return (
        <ThemeContext.Provider value={theme}>
          <Page />
        </ThemeContext.Provider>
      );
    }
    ```
    
    ### Props
    
    - `value` : 해당 provider 내에서 이 컨텍스트를 읽는 모든 컴포넌트에 전달하려는 값(모든 타입 가능)
    provider 내부에서 `useContext(SomeContext)` 를 호출하는 컴포넌트는 그 위에 있는 가장 안쪽에 해당하는 컨텍스트 provider의 value 를 받는다.
- `SomeContext.Consumer` : 컨텍스트 값을 읽는 또다른 방법이지만, 거의 사용되지 않는다.
    
    ```jsx
    function Button() {
      // 🟡 **Legacy way (not recommended)**
      return (
        <ThemeContext.Consumer>
          {theme => (
            <button className={theme} />
          )}
        </ThemeContext.Consumer>
      );
    }
    
    function Button() {
      // ✅ **Recommended way**
      const theme = useContext(ThemeContext);
      return <button className={theme} />;
    }
    ```
    
    - Legacy way 로, 새로 작성된 코드는 `useContext()` 를 이용해 컨텍스트를 읽어야한다.

### Usage

- context 생성하기
    
    ```jsx
    function App() {
      const [theme, setTheme] = useState('dark');
      const [currentUser, setCurrentUser] = useState({ name: 'Taylor' });
    
      // ...
    
      return (
        <ThemeContext.Provider value={theme}>
          <AuthContext.Provider value={currentUser}>
            <Page />
          </AuthContext.Provider>
        </ThemeContext.Provider>
      );
    }
    ```
    
    - 컨텍스트가 유용한 이유는 **컴포넌트에 다른 동적 값을 제공**할 수 있기 때문이다.
    - 만약 전달된 컨텍스트 값이 변경되면, React 는 컨텍스트를 읽는 컴포넌트도 다시 렌더링한다.
- 파일에서 컨텍스트 import 및 export 하기
    
    ```jsx
    // Contexts.js
    import { createContext } from 'react';
    
    export const ThemeContext = createContext('light');
    export const AuthContext = createContext(null);
    
    // Button.js
    import { ThemeContext } from './Contexts.js';
    
    function Button() {
      const theme = useContext(ThemeContext);
      // ...
    }
    
    // App.js
    import { ThemeContext, AuthContext } from './Contexts.js';
    
    function App() {
      // ...
      return (
        <ThemeContext.Provider value={theme}>
          <AuthContext.Provider value={currentUser}>
            <Page />
          </AuthContext.Provider>
        </ThemeContext.Provider>
      );
    }
    ```
    
    - 이는 **컴포넌트 import 및 export** 와 유사한 방식으로 작동한다.  


<aside>
💡 컨텍스트의 값을 변경하는 방법 : 일반적으로는 <strong>없다 !</strong> ( 시간이 지남에 따라 컨텍스트가 변경되도록 하려면 컨텍스트 provider 에 state 를 추가하고 컴포넌트를 감싸야한다. )
</aside>

