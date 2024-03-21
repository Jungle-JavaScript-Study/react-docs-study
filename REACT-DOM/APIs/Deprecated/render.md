## `render`

- React 18 에서 `render` 는 **`createRoot`** 로 교체되었습니다. React 18 에서 `render` 를 사용하면 앱이 React 17을 실행하는 것처럼 동작하므로 경고를 표시
- **Reference**
    - React 컴포넌트를 브라우저 DOM 요소 내에 표시하려면 `render` 를 호출
    
    ```jsx
    import { render } from 'react-dom';
    
    const domNode = document.getElementById('root');
    render(<App />, domNode);
    ```
    
    - 보통 React 로 완전히 구축된 앱은 `render` 를 최상 컴포넌트와 함께 한 번만 호출함. 페이지의 “일부분”에 React 사용하는 경우 필요한 만큼 `render` 를 호출 할  수 있음.
- **Parameters**
    - **`reactNode`** : 표시하려는 **React node.** 보통 ****`<App>` ****과 같은 JSX 를 사용하지만 **`createElement()`** 로 구성된 React 요소, 문자열, 숫자, null, 또는 undefined 를 전달할 수 있음.
    - **`domNode`** : **DOM 요소**. React 는 전달한 `reactNode` 를 이 DOM 요소 내에 표시합니다. 이후로 React 는 `domNode` 내부의 DOM을 관리하고 React 트리가 변경될 때 업데이트함.
    - **`callback?`** : **함수** 전달하면 React는 컴포넌트가 DOM 에 배치된 후 이를 호출함
- **Returns**
    - `render` 는 일반적으로 `null` 반환. `reactNode` 가 class 컴포넌트인 경우, 해당 컴포넌트의 인스턴스를 반환
- **Caveats**
    - React 18 에서 `render` 는 `createRoot` 로 교체됨. React 18 이상에서는 `createRoot` 를 사용
    - 처음 `render` 를 호출하면 React는 React 컴포넌트를 렌더링하기 전에 해당 `domNode` 내 존재하는 HTML 을 모두 초기화함. 서버에서 혹은 빌드 중에 React 에 의해 생성된 HTML 이 `domNode` 에 포함되어 있다면 기존 HTML 에 이벤트 핸들러를 연결하는 `hydrate()` 를 대신 사용하세요.
    - 동일한 `domNode` 에서 `render` 를 두 번 이상 호출하면 React는 최신 JSX 를 반영하기 위해 필요한 만큼 DOM을 업데이트 함. React 는 이전에 렌더링된 트리와 “매칭”하여 재사용할 수 있는 부분과 다시 만들어야하는 부분을 결정함. 동일한 `domNode` 에 `render` 를 재호출하는 것은 최상단 컴포넌트에서 `set 함수`를 호출하는 것과 유사합니다.
    - 앱 전체가 React 로 구축된 경우, `render` 호출은 앱에서 한 번만 발생할 것입니다. (프레임워크를 사용하는 경우, 이 호출을 대신 수행할 수 있습니다) 자식 컴포넌트가 아니라 DOM 트리의 다른 부분(ex> 모달 or 툴팁)에 JSX 를 렌더링하려면 `render` 대신 `createPortal` 을 사용하세요.
- **Usage**
    - **최상단 컴포넌트 렌더링하기**
        
        ```jsx
        import './styles.css';
        import { render } from 'react-dom';
        import App from './App.js';
        
        render(<App />, document.getElementById('root'));
        
        ```
        
    - **여러 개의 최상단 컴포넌트 렌더링하기**
        - 완전히 React 로 구축된 페이지가 아니라면 React 가 관리하는 최상위 UI 마다 `render` 를 호출하세요.
    - **렌더링 된 트리 업데이트하기**
        - 동일한 DOM 노드에서 `render` 를 여러 번 호출할 수 있음. 이전에 렌더링 된 구조와 컴포넌트 트리가 일치한다면 React는 state를 보존함.
        - `render` 를 여러 번 호출하는 것은 일반적이지 않음. 보통 컴포넌트 내에서 **상태를 업데이트** 함.
