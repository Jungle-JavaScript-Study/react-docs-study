- `useId` 는 접근성 속성에 전달할 수 있는 고유 ID 를 생성하기 위한 React Hook
- Reference
    
    ```jsx
    // 컴포넌트의 최상위 수준에서 호출하여 고유한 ID 를 생성합니다.
    const id = useId()
    ```
    
    ```jsx
    import { useId } from 'react';
    
    function PasswordField() {
      const passwordHintId = useId();
      // ...
    ```
    
- Parameters : none
- Returns
    - **고유 ID 문자열을 반환**
- Caveats
    - `useId` 는 훅이므로 컴포넌트 최상단 또는 커스텀훅에서만 호출 가능
    - `useId` 를 **목록에서 키를 생성하기 위해 사용하지 마세요**. 
    ( 키는 데이터에서 생성되어야 한다. )
        - **데이터베이스의 데이터:** 데이터베이스에서 데이터를 가져오는 경우, 고유한 데이터베이스 key/ID를 사용할 수 있습니다.
        - **로컬에서 생성된 데이터:** 데이터가 로컬에서 생성되고 유지되는 경우(예: 메모 작성 앱의 메모), 항목을 만들 때 증분 카운터, `[crypto.randomUUID()](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/randomUUID)` 또는 `[uuid](https://www.npmjs.com/package/uuid)`와 같은 패키지를 사용하세요.
- Usage
    - **[접근성 속성](https://developer.mozilla.org/ko/docs/Web/Accessibility/ARIA)에 대한 고유 ID 생성**
        
        ```jsx
        import { useId } from 'react';
        
        function PasswordField() {
          const passwordHintId = useId();
          return (
            <>
              <label>
                Password:
                <input
                  type="password"
                  aria-describedby={passwordHintId}
                />
              </label>
              <p id={passwordHintId}>
                The password should contain at least 18 characters
              </p>
            </>
          );
        }
        
        // PasswordField 가 화면에 여러번 나타나도 생성된 ID 가 충돌이 없다.
        export default function App() {
          return (
            <>
              <h2>Choose password</h2>
              <PasswordField />
              <h2>Confirm password</h2>
              <PasswordField />
            </>
          );
        }
        ```
        
        <aside>
        💡 [서버 렌더링](https://react-ko.dev/reference/react-dom/server)을 사용하면, **`useId`와 클라이언트에서 동일한 컴포넌트 트리가 필요.** 서버와 클라이언트에서 렌더링한 트리가 정확히 일치하지 않으면 생성된 ID가 일치하지 않음.
        
        </aside>
        
        <aside>
        💡 `useId` 가 `nextId++` 와 같은 전역변수를 증가시키는 것보다 더 나은 이유는 
        `**useId` 는 React 가 서버 렌더링과 함께 동작하도록 보장한다는 것
        (**React 내부에서 `useId`는 호출한 컴포넌트의 “부모 경로”에서 생성되므로 클라이언트와 서버 트리가 동일하면 렌더링 순서와 상관 없이 “부모 경로”가 일치하므로 `useId` 또한 일치할 것)
        
        </aside>
        
    - **여러 관련 요소에 대한 ID 생성**
        - 여러 관련 요소에 ID 를 제공해야 하는 경우, `useId` 를 호출하여 **해당 요소들이 공유하는 접두사**를 생성할 수 있습니다.
            
            ```jsx
            import { useId } from 'react';
            
            export default function Form() {
              const id = useId();
              return (
                <form>
                  <label htmlFor={id + '-firstName'}>First Name:</label>
                  <input id={id + '-firstName'} type="text" />
                  <hr />
                  <label htmlFor={id + '-lastName'}>Last Name:</label>
                  <input id={id + '-lastName'} type="text" />
                </form>
              );
            }
            ```
            
    - **생성된 모든 ID 에 공유 접두사 지정하기**
        - 단일 페이지에서 여러 개의 독립적인 React 어플리케이션을 렌더링하는 경우,
        `createRoot` 또는 `hydrateRoot` 를 호출해 `identifierPrefix` 에 옵션으로 전달하면, 생성된 모든 식별자가 지정한 고유한 접두사로 시작하기 때문에 서로 다른 두 앱에서 생성된 ID 가 충돌하지 않음.
            
            ```jsx
            //index.js
            import { createRoot } from 'react-dom/client';
            import App from './App.js';
            import './styles.css';
            
            const root1 = createRoot(document.getElementById('root1'), {
              identifierPrefix: 'my-first-app-'
            });
            root1.render(<App />);
            
            const root2 = createRoot(document.getElementById('root2'), {
              identifierPrefix: 'my-second-app-'
            });
            root2.render(<App />);
            
            // App.js
            import { useId } from 'react';
            
            function PasswordField() {
              const passwordHintId = useId();
              console.log('Generated identifier:', passwordHintId)
              return (
                <>
                  <label>
                    Password:
                    <input
                      type="password"
                      aria-describedby={passwordHintId}
                    />
                  </label>
                  <p id={passwordHintId}>
                    The password should contain at least 18 characters
                  </p>
                </>
              );
            }
            
            export default function App() {
              return (
                <>
                  <h2>Choose password</h2>
                  <PasswordField />
                </>
              );
            }
            
            // index.html
            <!DOCTYPE html>
            <html>
              <head><title>My app</title></head>
              <body>
                <div id="root1"></div>
                <div id="root2"></div>
              </body>
            </html>
            ```