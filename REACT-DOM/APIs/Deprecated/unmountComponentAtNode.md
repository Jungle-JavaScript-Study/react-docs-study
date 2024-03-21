## `unmountComponentAtNode`

- React 18 에서 `unmountComponentAtNode` 는 `**root.unmount()`** 로 교체되었습니다.
- **Reference**
    - **`unmountComponentAtNode(domNode)`** 를 호출해서 DOM 에 마운트 된 React 컴포넌트를 제거하고 관련된 이벤트핸들러와 state를 정리합니다.
        
        ```jsx
        import { unmountComponentAtNode } from 'react-dom';
        
        const domNode = document.getElementById('root');
        render(<App />, domNode);
        
        unmountComponentAtNode(domNode);
        ```
        
- **Parameters**
    - **`domNode`** : **DOM 엘리멘트.** React는 이 엘리먼트에 마운트된 컴포넌트를 제거함.
- **Returns**
    - **`unmountComponentAtNode`** 는 컴포넌트가 마운트 해제되면 **`true`**를 반환하고 그렇지 않으면 **`false`** 를 반환함.
- **Usage**
    - **DOM 엘리먼트에서 React 애플리케이션 제거하기**
        - 때떄로 기존 페이지나 일부만 React로 작성된 페이지에서 React 를 ‘**포함**’하고 싶을 수 있음. 이런 경우 렌더링 된 DOM 노드에서 UI와 state 및 리스너를 모두 제거해서 React 애플리케이션을 ‘**중지**’해야 할 수 있음.
        
        ```jsx
        import './styles.css';
        import { render, unmountComponentAtNode } from 'react-dom';
        import App from './App.js';
        
        const domNode = document.getElementById('root');
        
        document.getElementById('render').addEventListener('click', () => {
          render(<App />, domNode);
        });
        
        document.getElementById('unmount').addEventListener('click', () => {
          unmountComponentAtNode(domNode);
        });
        
        ```
