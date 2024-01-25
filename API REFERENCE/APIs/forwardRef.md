### Reference

- `forwardRef(render)` 를 사용하면 컴포넌트가 `ref` 를 사용하여 **부모 컴포넌트에 DOM 노드를 노출할 수 있다**.
    
    ```jsx
    import { forwardRef } from 'react';
    
    const MyInput = forwardRef(function MyInput(props, ref) {
      // ...
    });
    ```
    

### Parameters

- `render` : 컴포넌트의 렌더링 함수. React는 컴포넌트가 부모로부터 받은 props 와 `ref` 를 가지고 이 함수를 호출한다. 반환하는 JSX 는 컴포넌트의 결과물이 된다.
    
    ```jsx
    const MyInput = forwardRef(function MyInput(props, ref) {
      return (
        <label>
          {props.label}
          <input ref={ref} />
        </label>
      );
    });
    ```
    

### Returns

- JSX 에서 렌더링할 수 있는 React 컴포넌트를 반환한다.
`forwardRef` 가 반환하는 컴포넌트는 `ref` props 를 받을수도 있다.

### Caveats

- Strict Mode 에서 React 는 렌더링 함수를 **두 번** 호출한다.
( 렌더링 함수가 순수하다면, 로직에 영향을 미치지 않을것이고, 호출 중 하나의 결과는 무시된다. )

### Render 함수

- **Parameters**
    - `props` : 부모 컴포넌트가 전달한 props
    - `ref` : 부모 컴포넌트가 전달한 `ref` 속성. `ref` 는 객체 or 함수 ( 부모 컴포넌트가 ref 를 전달하지 않을 경우 `null` ) - 받은 ref 를 다른 컴포넌트에 전달하거나 `useImperativeHandle` 에 전달해야 한다.
- **Returns**
    - JSX 에서 렌더링할 수 있는 React 컴포넌트를 반환.

### Usage

- 부모 컴포넌트에 DOM 노드 노출하기
    - 기본적으로 각 컴포넌트의 DOM 노드는 비공개이다.
    그러나 때로는 `forwardRef` 를 통해 부모에 DOM 노드를 노출하는 것이 유용하기도 하다.
        
        ```jsx
        import { forwardRef } from 'react';
        
        const MyInput = forwardRef(function MyInput(props, ref) {
          const { label, ...otherProps } = props;
          return (
            <label>
              {label}
              <input {...otherProps} />
            </label>
          );
        });
        
        function Form() {
          const ref = useRef(null);
        
          function handleClick() {
            ref.current.focus();
          }
        
          return (
            <form>
              <MyInput label="Enter your name:" ref={ref} />
              <button type="button" onClick={handleClick}>
                Edit
              </button>
            </form>
          );
        }
        ```
        
    
- 여러 컴포넌트를 통해 ref 전달하기
    
    ```jsx
    // App.js
    import { useRef } from 'react';
    import FormField from './FormField.js';
    
    export default function Form() {
      const ref = useRef(null);
    
      function handleClick() {
        ref.current.focus();
      }
    
      return (
        <form>
          <FormField label="Enter your name:" ref={ref} isRequired={true} />
          <button type="button" onClick={handleClick}>
            Edit
          </button>
        </form>
      );
    }
    
    // FormField.js
    import { forwardRef, useState } from 'react';
    import MyInput from './MyInput.js';
    
    const FormField = forwardRef(function FormField({ label, isRequired }, ref) {
      const [value, setValue] = useState('');
      return (
        <>
          <MyInput
            ref={ref}
            label={label}
            value={value}
            onChange={e => setValue(e.target.value)} 
          />
          {(isRequired && value === '') &&
            <i>Required</i>
          }
        </>
      );
    });
    
    export default FormField;
    
    // MyInput.js
    import { forwardRef } from 'react';
    
    const MyInput = forwardRef((props, ref) => {
      const { label, ...otherProps } = props;
      return (
        <label>
          {label}
          <input {...otherProps} ref={ref} />
        </label>
      );
    });
    
    export default MyInput;
    ```
    
- DOM 노드 대신 명령형 핸들 노출하기
    - 전체 DOM 노드를 노출하는 대신 보다 제한된 메서드 집합을 사용해 사용자 정의 객체(명령형 핸들)를 노출할 수 있다.
    
    ```jsx
    // MyInput.js
    import { forwardRef, useRef, useImperativeHandle } from 'react';
    
    const MyInput = forwardRef(function MyInput(props, ref) {
      const inputRef = useRef(null);
    
      useImperativeHandle(ref, () => {
        return {
          focus() {
            inputRef.current.focus();
          },
          scrollIntoView() {
            inputRef.current.scrollIntoView();
          },
        };
      }, []);
    
      return <input {...props} ref={inputRef} />;
    });
    
    // App.js
    import { useRef } from 'react';
    import MyInput from './MyInput.js';
    
    export default function Form() {
      const ref = useRef(null);
    
      function handleClick() {
        ref.current.focus();
        // This won't work because the DOM node isn't exposed:
        // DOM 노드가 노출되지 않았기 때문에 작동하지 않습니다:
        // ref.current.style.opacity = 0.5;
      }
    
      return (
        <form>
          <MyInput label="Enter your name:" ref={ref} />
          <button type="button" onClick={handleClick}>
            Edit
          </button>
        </form>
      );
    }
    ```
    

<aside>
💡 ref 를 과도하게 사용하지 마세요 ! ( prop 으로 표현할 수 있다면, prop 사용 ! )

</aside>
