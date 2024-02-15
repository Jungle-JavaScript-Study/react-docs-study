## `useFormState`
💡 **Canary**  
<aside>
<strong>`useFormState` 는 React Server Components 를 지원하는 프레임워크를</strong> 사용해야 최대한의 이익을 얻을 수 있습니다.
</aside>

- Reference
    - **`useFormState`** 는 form 작업 결과에 따라 상태를 업데이트 할 수 있습니다.
    
    ```jsx
    const [state, formAction] = useFormState(action, initialState, permalink?);
    ```
    
    - Parameters
        - **`action`** : form 을 제출하거나 버튼을 눌렀을 때 호출할 함수
        ( 함수가 호출되면, 처음에는 initialState, 이후에는 이전 반환값 )를 초기 인수로 받은 후 다음 form action이 일반적으로 받는 인수를 받음.
        - **`initialState`** : 초기 상태값. 직렬화가능한 값이어야하고, action 이 처음으로 호출된 이후에는 무시됨.
        - **`permalink?`** : form 이 수정하는 유일한 URL 값이 포함된 문자열. progressive 향상과 함께 동적컨텐츠(ex> feeds) 가 있는 페이지에서 사용하세요.
            - 만약 action 이 서버액션이고 폼이 JavaScript bundle 이 로드되기 전 제출된다면, 브라우저는 현재의 페이지URL 이 아닌 특정 permalink URL 로 이동합니다.
            - React 가 상태를 전달하는 방법을 알 수 있도록 동일한 양식 컴포넌트가 대상 페이지에서 렌더링 되어야함.
            - 폼이 hydrate 되면, 이 파라미터는 아무런 영향을 미치지 않음.

    - Returns
        - 두 값이 있는 배열을 return
            1. 현재 상태. 첫번째 렌더링시에는 initialState, 그 이후에는 action 이 호출되고 반환한 값
            2. 폼 컴포넌트에 action으로 전달할 수 있는 새로운 action
        - 주의사항
            - React 서버 컴포넌트와 함께 사용하면, 클라이언트에서 JavaScript 가 실행되기 전에 form 을 interactive 하게 만들 수 있음. 서버 컴포넌트 없이 사용하면, local 상태와 동일함.
            - `useFormState` 에 전달된 함수는 첫번째 인자로 이전 또는 초기상태라는 추가 인자를 받음.
            그것이 `useFormState` 를 사용하지 않고 폼 액션을 직접 사용할때와의 차이를 만들어냄.
- Usage
    - **form 액션에 의해 반환된 정보 사용**
        
        ```jsx
        import { useFormState } from 'react-dom';
        import { action } from './actions.js';
        
        function MyComponent() {
          const [state, formAction] = useFormState(action, null);
          // ...
          return (
            <form action={formAction}>
              {/* ... */}
            </form>
          );
        }
        
        function action(currentState, formData) {
          // ...
          return 'next state';
        }
        ```
