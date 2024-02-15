## `useFormStatus`
💡 **Canary**

- Reference
    - **`useFormStatus`** 는 마지막으로 form 제출의 상태정보를 제공하는 훅
        
        ```jsx
        const { pending, data, method, action } = useFormStatus();
        
        import { useFormStatus } from "react-dom";
        import action from './actions';
        
        function Submit() {
          const status = useFormStatus();
          return <button disabled={status.pending}>Submit</button>
        }
        
        export default function App() {
          return (
            <form action={action}>
              <Submit />
            </form>
          );
        }
        ```
        
    - Parameters
        - none
    - Returns
        - 아래와 같은 프로퍼티를 갖는 status 객체를 반환한다
            - **`pending`** : boolean 값, true 이면 <form> 이 제출중, 아니면 false
            - **`data`** : 부모 <form> 이 제출하는 데이터를 포함한 formData interface 객체, 활성화된 제출이 없거나 부모 <form> 이 없으면 `null`
            - **`method`** : ‘get’ or ‘post’ 문자열 값. 부모 <form>이 GET 또는 POST 로 제출하는지 나타내는 값 ( 기본값: GET , **메서드 지정가능 - OPTION, DELETE, PUT 등은 HTML 표준 폼에서 직접지원하지 않음**.)
            - **`action`** : 부모 <form> 의 액션 프로퍼티에 전달된 함수. 부모 <form> 이 없거나 지정된 액션 프로퍼티가 없거나 URI value 가 제공되어 있으면 `null`
        - 주의사항
            - `useFormStatus` 훅은 **반드시 <form> 안에 렌더링된 컴포넌트에서만 호출**해야한다
            - `useFormStatus` 는 **부모 <form> 의 상태정보를 리턴**한다. 자식 또는 형제 컴포넌트의 <form> 상태 정보는 반환하지 않는다.
- Usage
    - **form 제출 중 pending 상태 표시**
        
        ```jsx
        // App.js
        import { useFormStatus } from "react-dom";
        import { submitForm } from "./actions.js";
        
        function Submit() {
          const { pending } = useFormStatus();
          return (
            <button type="submit" disabled={pending}>
              {pending ? "Submitting..." : "Submit"}
            </button>
          );
        }
        
        function Form({ action }) {
          return (
            <form action={action}>
              <Submit />
            </form>
          );
        }
        
        export default function App() {
          return <Form action={submitForm} />;
        }
        ```
        
    - **제출되는 form data 읽기**
        
        ```jsx
        // UsernameForm.js
        import {useState, useMemo, useRef} from 'react';
        import {useFormStatus} from 'react-dom';
        
        export default function UsernameForm() {
          const {pending, data} = useFormStatus();
        
          const [showSubmitted, setShowSubmitted] = useState(false);
          const submittedUsername = useRef(null);
          const timeoutId = useRef(null);
        
          useMemo(() => {
            if (pending) {
              submittedUsername.current = data?.get('username');
              if (timeoutId.current != null) {
                clearTimeout(timeoutId.current);
              }
        
              timeoutId.current = setTimeout(() => {
                timeoutId.current = null;
                setShowSubmitted(false);
              }, 2000);
              setShowSubmitted(true);
            }
          }, [pending, data]);
        
          return (
            <>
              <label>Request a Username: </label><br />
              <input type="text" name="username" />
              <button type="submit" disabled={pending}>
                {pending ? 'Submitting...' : 'Submit'}
              </button>
              {showSubmitted ? (
                <p>Submitted request for username: {submittedUsername.current}</p>
              ) : null}
            </>
          );
        }
        
        // App.js
        import UsernameForm from './UsernameForm';
        import { submitForm } from "./actions.js";
        
        export default function App() {
          return (
            <form action={submitForm}>
              <UsernameForm />
            </form>
          );
        }
        ```
