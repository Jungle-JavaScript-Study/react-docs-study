## `hydrate`

- React 18 에서 hydrateRoot 로 바뀌었음. React 18에서 `hydrate` 를 사용하면 React 17 을 사용하는 것처럼 동작할 것이라는 경고가 표시됨.
- React 17 이하에서, `hydrate` 는 **`react-dom/server`** 로부터 생성된 React 컴포넌트의 HTML 콘텐츠를 브라우저 DOM 노드에 표시할 수 있도록 해줌
- **Reference**
    - **`hydrate(reactNode, domNode, callback?)`**
        
        ```jsx
        import { hydrate } from 'react-dom';
        
        hydrate(reactNode, domNode);
        ```
        
        React 는 `domNode` 에 있는 HTML 에 연결되고, 그 내부의 DOM 관리를 맡음**. React 로 빌드된 앱은 보통 hydrate 를 루트 컴포넌트에서 딱 한 번 실행함.**
        
- **Parameters**
    - **`reactNode`** : HTML 을 렌더링하기 위해 사용되는 React 노드 ( React 17의 `renderToString(<App/>)` 같은 `ReactDOM Server` 메서드로 렌더링 된 `<App/>` 처럼, 일반적으로 JSX 조각 형태.
    - **`domNode`** : 서버에서 루트 요소로 렌더링 된 DOM 요소
    - `callback?` : 컴포넌트가 hydrate 된 후 호출되는 함수
- **Returns**
    - `null`
- **Caveats**
    - `hydrate` 는 클라이언트에서 렌더링 된 콘텐츠가 서버에서 렌더링 된 콘텐츠와 동일할 것으로 예상함. 텍스트 콘텐츠의 차이 정도는 React 가 조정할 수 있지만, 불일치는 버그로 간주하고 수정해야 함
    - 개발 모드에서 React 는 hydration 중 일어난 불일치에 대해 경고함. 불일치 하는 경우 어트리뷰트 차이가 조정된다는 보장은 없음. - **성능상의 이유로 중요 ( 대부분의 앱에서 불일치가 발생하는 경우는 드물어서, 모든 마크업의 유혀성을 검사하는데 엄청난 비용이 듦 )**
    - 각 앱에서 hydrate 는 한 번만 실행하세요. 프레임워크를 사용하는 경우 프레임워크가 대신 호출하고 있을 것입니다.
    - 앱이 서버에서 렌더링 된 HTML 없이 클라이언트에서만 렌더링 되는 경우엔 `hydrate()` 를 사용할 수 없습니다. 이럴때는 `render()` ( React 17 이하 ) or `createRoot()` (React 18 이상) 을 사용하세요.
- **Usage**
    - **서버에서 렌더링 된 HTML Hydrate 하기**
        - React 에서 **hydration** 은 **React 가 서버 환경에서 미리 렌더링한 HTML 에 연결하는 방식**을 말함. 클라이언트에서 hydration 이 일어나면 React 는 서버에서 생성된 마크업에 이벤트리스너를 연결하고 앱 렌더링을 이어받으려고 함.
        - **`hydrate()`** 더이상 사용하지 않음. **사용할 경우 React 17 처럼 동작할 것이라는 경고**
            
           ![Untitled](https://github.com/SungBeum/carmage/assets/126440955/440f085e-4f8f-41fd-b297-aa510b1aea50)
          
    - **불필요한 hydration 불일치 에러 무시하기**
        - 단일 요소의 어트리뷰트나 텍스트 콘텐츠가 서버와 클라이언트 간 다를 수 밖에 없는 경우 (ex> 타임스탬프) hydration 불일치 경고를 무시하도록 처리할 수 있음.
        - hydration 경고를 무시하려면 **`suppressHydrationWarning={true}`** 를 추가
        - 이 방법은 한 레벨 깊이에서만 작동하며 임시방편일 뿐. 남용 X
    - **서로 다른 클라이언트 및 서버 콘텐츠 처리하기**
        - 서버와 클라이언트에서 의도적으로 다른 것을 렌더링하는 경우, **투 패스 렌더링**을 사용해 볼 수 있음. `isClient` 같은 `state` 를 선언해 `effect` 에서 `true` 로 바꿔 사용하면 됨.
            
            ```jsx
            import { useState, useEffect } from "react";
            
            export default function App() {
              const [isClient, setIsClient] = useState(false);
            
              useEffect(() => {
                setIsClient(true);
              }, []);
            
              return (
                <h1>
                  {isClient ? 'Is Client' : 'Is Server'}
                </h1>
              );
            }
            ```
            
        - 이 방법을 사용할 때에는, 컴포넌트가 두 번 렌더링 되어야 하므로 hydration 속도가 느려짐. **인터넷 속도가 느린 사용자의 경험에 유의하세요.**
