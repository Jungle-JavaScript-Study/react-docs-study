## `<input/>`

- **`<input>`** 브라우저 내장 컴포넌트를 사용하면 여러 종류의 form input 을 렌더링할 수 있습니다.
- **Props**
    - `<input>` 은 일반적인 엘리먼트 props 를 지원합니다.
    `formAction` prop 은 **Canary**
    - **`formAction` :** 문자열 또는 함수.
        - 부모 `<form action>`의 `type=”submit”` 그리고 `type=”image”` 를 재정의합니다.
        - URL 이 액션에 전달되면, 해당 `form` 표준 HTML `form` 처럼 동작한다.
        - 함수가 `formAction` 에 전달되면, 폼 양식을 처리한다.
    - 아래 둘 중 하나를 전달할 때는 반드시 전달된 값을 업데이트하는 `onChange` 핸들러도 함께 전달해야 한다.(or `readOnly` 를 전달해야 함.)
        - **`checked`** : boolean, 체크박스 input 또는 라디오버튼에서 선택 여부를 제어
        - **`value`** : string, 텍스트 input 의 경우 텍스트를 제어 ( 라디오의 경우, 폼 데이터를 지정 )
    - 아래 props 들은 제어되지 않는 input 들에만 적용
        - **`defaultChecked`** : boolean, `type=”checkbox”` , `type=”radio”` input 에 대한 **초기값**을 지정
        - **`defaultValue`** : string, 텍스트 input 에 대한 **초기값**을 지정
    - 다음 props 들은 제어되지 않는 input 과 제어되는 input 모두 적용
        - **`accept`** : string, `type=”file”` input 에서 허용할 파일 형식을 지정
        - **`alt`** : string, `type=”image”` input 에서 대체 이미지 텍스트를 지정
        - **`capture`** : string, `type=”file”` input 으로 캡처한 미디어(마이크, 비디오, 카메라) 를 지정
        - **`autoComplete`** : string, 자동 완성 동작들 중 가능한 하나를 지정
        - **`autoFocus`** : boolean, `true` 일 경우 React 는 마운트 시 엘리먼트에 포커스를 맞춤
        - **`dirname`** : string, 엘리먼트 방향성에 대한 폼 필드 이름을 지정
        - **`disabled`** : boolean, `true` 일 경우, input 은 상호작용이 불가능해지며 흐릿해짐
        - **`children`** : X , input 은 no Children
        - **`form`** : string, input 이 속하는 <form> 의 id 를 지정 ( 생략 시 가장 가까운 부모 폼으로 설정 )
        - **`formAction`** : string, `type=”submit”` , `type=”image”` 의 부모 `<form action>` 을 덮어 씀
        - **`formEctype`** : string, `type=”submit”` , `type=”image”` 의 부모 `<form enctype>` 을 덮어 씀 (form 데이터를 서버에 제출할 때 사용할 인코딩 방법을 식별하는 문자열)
        - **`formMethod`**  : string, `type=”submit”` , `type=”image”` 의 부모 `<form method>` 을 덮어 씀 (form 데이터를 서버에 제출할 때 사용할 method 를 식별하는 문자열)
        - **`formNoValidate`** : string, `type=”submit”` , `type=”image”` 의 부모 `<form noValidate>` 을 덮어 씀 ( form 데이터를 서버에 제출할 때 validate 를 무시할 수 있도록 하는 불리언 값 )
        - **`formTarget`** : string, `type=”submit”` , `type=”image”` 의 부모 `<form target>` 을 덮어 씀 ( form submit 할 때 Context 를 탐색 )
        - **`height`** : string, `type=”image”` 의 이미지 높이를 지정
        - **`list`** : string, `<datalist>` 의 `id` 를 자동 완성 옵션들로 지정
        - **`max`** : number, 숫자와 날짜 input 들의 최대값을 지정
        - **`maxLength`** : number, 텍스트와 다른 input 들의 최대 길이를 지정
        - **`min`** : number, 숫자와 날짜 input 들의 최솟값을 지정
        - **`minLength`** : number, 텍스트와 다른 input 들의 최소 길이를 지정
        - **`multiple`** : boolean, `type=”file”` , `type=”email”` 에 대해 여러 값을 허용할지 여부 지정
        - **`name`** : string, `form` 과 제출되는 input 의 이름 지정
        - **`onChange`** : event handler function, 제어되는 input 필수 요소**로 input 값을 변경하는 즉시 실행 ( 브라우저 input 이벤트처럼 동작 )
        - **`onChangeCapture`** : **캡처단계**에서 실행되는 `onChange`
        - **`onInput`** : event handler function, 사용자가 값을 변경하는 즉시 실행 ( onChange 를 사용하는 것이 관습 )
        - **`onInputCapture`** : 캡처단계에서 실행되는 `onInput`
        - **`onInvalid`** : event handler function, form 제출 시 input이 유효하지 않을 경우 실행 ( invalid 내장 이벤트와 달리 **버블링 발생** )
        - **`onInvalidCature`** : 캡처 단계에서 실행되는 `onInvalid`
        - **`onSelect`** : event handler function, <input> 내부의 선택 사항이 변경된 후 실행 ( React 는 이벤트를 확장해 **선택사항이 비거나 편집 시 선택사항에 영향을 끼치게 될 때에도 실행** )
        - **`onSelectCapture`** : 캡처 단계에서 실행되는 `onSelect`
        - **`pattern`** : string, `value` 가 일치해야 하는 패턴 지정
        - **`placeholder`** : string, input 값이 비었을 때 흐린 색으로 표시
        - **`readOnly`** : boolean, `true` 일 때, 사용자가 input 을 편집X
        - **`required`** : boolean, `true` 일 때, 폼의 제출할 값을 반드시 제공해야 함
        - **`size`** : number, 너비를 설정하는 것과 비슷하지만 단위는 제어에 따라 다름.
        - **`src`** : string, `type=”image”` input의 이미지 소스를 지정
        - **`step`** : positive number or “any”, 유효한 값 사이의 간격 지정
        - **`type`** : string, input types 중 하나
        - **`width`** : string, `type=”image”` input 의 이미지 너비 지정
    - **Caveat**
        - 체크박스는 `value` 가 아닌 `checked` 가 필요함
        - 텍스트 input 영역은 문자열 `value` prop 을 받을 경우 **제어되는 것으로 취급**
        - 체크박스 또는 라디오버튼이 불리언 `checked` prop 을 받을 경우 **제어되는 것을 취급**
        - input 은 제어되면서 동시에 비제어될 수 없음
        - input 은 생명주기 동안 제어 도는 비제어 상태를 오갈 수 없음
        - 제어되는 input엔 모두 백업 값을 동기적으로 업데이트하는 `onChange` 이벤트 핸들러가 필요
- Usage
    - **다양한 유형의 input 표시**
      <details>
        <summary>ex)</summary>
        
        ```jsx
        export default function MyForm() {
          return (
            <>
              <label>
                Text input: <input name="myInput" />
              </label>
              <hr />
              <label>
                Checkbox: <input type="checkbox" name="myCheckbox" />
              </label>
              <hr />
              <p>
                Radio buttons:
                <label>
                  <input type="radio" name="myRadio" value="option1" />
                  Option 1
                </label>
                <label>
                  <input type="radio" name="myRadio" value="option2" />
                  Option 2
                </label>
                <label>
                  <input type="radio" name="myRadio" value="option3" />
                  Option 3
                </label>
              </p>
            </>
          );
        }
        
        ```
      </details>
            
    - **input에 라벨 제공하기**
        - 일반적으로 모든 `<input>` 은 `<label>` 태그 안에 두게 되는데, 이러면 **해당 라벨이 해당 input과 연관됨을 브라우저가 알 수 있음**. ( 사용자가 라벨을 클릭하면 브라우저는 input 에 자동적으로 포커스를 맞춤 )
        - 스크린리더는 사용자가 연관된 input 에 포커스를 맞출 때  라벨 캡션을 읽게 되므로 접근성을 위해서도 필수
        - `<label>` 안에 `<input>` 을 감쌀 수 없다면, `<input id>` 와 `<label htmlFor>` 에 동일한 ID 를 전달해서 연관성을 부여. ( 한 컴포넌트의 여러 인스턴스 간 충돌을 피하려면 `useId` 사용 )
          <details>
            <summary>ex)</summary>
        
          ```jsx
          import { useId } from 'react';
          
          export default function Form() {
            const ageInputId = useId();
            return (
              <>
                <label>
                  Your first name:
                  <input name="firstName" />
                </label>
                <hr />
                <label htmlFor={ageInputId}>Your age:</label>
                <input id={ageInputId} name="age" type="number" />
              </>
            );
          }
          
          ```
        </details>
            
    - **input에 초깃값 제공하기**
        - `defaultValue` 로 초깃값을 선택적으로 지정 가능. ( 라디오버튼과 체크박스는 `defaultChecnked` )
          <details>
          <summary>ex)</summary>
              
            ```jsx
            export default function MyForm() {
              return (
                <>
                  <label>
                    Text input: <input name="myInput" defaultValue="Some initial value" />
                  </label>
                  <hr />
                  <label>
                    Checkbox: <input type="checkbox" name="myCheckbox" defaultChecked={true} />
                  </label>
                  <hr />
                  <p>
                    Radio buttons:
                    <label>
                      <input type="radio" name="myRadio" value="option1" />
                      Option 1
                    </label>
                    <label>
                      <input
                        type="radio"
                        name="myRadio"
                        value="option2"
                        defaultChecked={true} 
                      />
                      Option 2
                    </label>
                    <label>
                      <input type="radio" name="myRadio" value="option3" />
                      Option 3
                    </label>
                  </p>
                </>
              );
            }
            
            ```
          </details>
            
    - **폼 제출 시 input 값 읽기**
        - inputs 와 `<button type=”submit”>` 바깥을 `<form>` 으로 감싸면 버튼을 클릭했을 때 `<form onSubmit>` 이벤트핸들러가 호출 됨. 
        form 데이터는 `new FormData(e.target)` 으로 읽을 수 있음 ( 기본적인 브라우저 동작 : 현재 URL 에 폼 데이터를 전송한 후 페이지 새로고침 , `e.preventDefault()` 로 덮어쓸 수 있음 )
          <details>
            <summary>ex)</summary>
            
            ```jsx
            export default function MyForm() {
              function handleSubmit(e) {
                // 브라우저가 페이지를 다시 로드하지 못하도록 방지합니다.
                e.preventDefault();
            
                // 폼 데이터를 읽습니다.
                const form = e.target;
                const formData = new FormData(form);
            
                // formData를 직접 fetch body로 전달할 수 있습니다.
                fetch('/some-api', { method: form.method, body: formData });
            
                // 또는 순수 object로 작업할 수 있습니다.
                const formJson = Object.fromEntries(formData.entries());
                console.log(formJson);
              }
            
              return (
                <form method="post" onSubmit={handleSubmit}>
                  <label>
                    Text input: <input name="myInput" defaultValue="Some initial value" />
                  </label>
                  <hr />
                  <label>
                    Checkbox: <input type="checkbox" name="myCheckbox" defaultChecked={true} />
                  </label>
                  <hr />
                  <p>
                    Radio buttons:
                    <label><input type="radio" name="myRadio" value="option1" /> Option 1</label>
                    <label><input type="radio" name="myRadio" value="option2" defaultChecked={true} /> Option 2</label>
                    <label><input type="radio" name="myRadio" value="option3" /> Option 3</label>
                  </p>
                  <hr />
                  <button type="reset">Reset form</button>
                  <button type="submit">Submit form</button>
                </form>
              );
            }
            
            ```
          </details>
            
        
        <aside>
        💡 기본적으로 `<form>` 내부의 어느 `<button>` 이든 폼을 제출함. <strong>커스텀 Button React 컴포넌트</strong>의 경우 `<button>` 대신 `<button type=”button”>` 을 사용. <strong>명시성 부여를 위해</strong> 폼 제출용 버튼은 `<button type=”submit”>` 을 사용
        
        </aside>
        
    - **state 변수로 input 제어하기**
        - **제어되는 input 을 렌더링하려면 `value` (or `checked` ) prop을 전달해야함.** ( 일반적으로 **state 변수**를 선언하여 할 수 있음 )
          <details>
            <summary>ex)</summary>
            
            ```jsx
            import { useState } from 'react';
            
            export default function Form() {
              const [firstName, setFirstName] = useState('');
              const [age, setAge] = useState('20');
              const ageAsNumber = Number(age);
              return (
                <>
                  <label>
                    First name:
                    <input
                      value={firstName}
                      onChange={e => setFirstName(e.target.value)}
                    />
                  </label>
                  <label>
                    Age:
                    <input
                      value={age}
                      onChange={e => setAge(e.target.value)}
                      type="number"
                    />
                    <button onClick={() => setAge(ageAsNumber + 10)}>
                      Add 10 years
                    </button>
                  </label>
                  {firstName !== '' &&
                    <p>Your name is {firstName}.</p>
                  }
                  {ageAsNumber > 0 &&
                    <p>Your age is {ageAsNumber}.</p>
                  }
                </>
              );
            }
            
            ```
          </details>
            
            - 제어되는 컴포넌트에 전달되는 `value` 는 `undefined` or `null` 이면 안됨. **초깃값을 비워둬야하는 경우 빈 문자열(’’)로 초기화** 할 것
   
    - **키보드를 누를 때마다 리렌더링 최적화하기***
        - 제어된 input 을 사용할 때에는 키보드를 누를 때마다 state 를 설정함. (**state 를 포함하는 컴포넌트가 큰 트리를 리렌더링할 경우 속도가 느려질 수 있음**)
          <details>
            <summary>ex)</summary>
              
            ```jsx
            // bad case : 키보드를 누를때마다 모든 페이지 내용을 리렌더링 함
            function App() {
              const [firstName, setFirstName] = useState('');
              return (
                <>
                  <form>
                    <input value={firstName} onChange={e => setFirstName(e.target.value)} />
                  </form>
                  <PageContent />
                </>
              );
            }
            
            // good case : 키보드를 누를때마다 SignupForm 만 리렌더링 함
            function App() {
              return (
                <>
                  <SignupForm />
                  <PageContent />
                </>
              );
            }
            
            function SignupForm() {
              const [firstName, setFirstName] = useState('');
              return (
                <form>
                  <input value={firstName} onChange={e => setFirstName(e.target.value)} />
                </form>
              );
            }
            ```
          </details>

      - 리렌더링을 피할 방법이 없는 경우 (ex> 검색 input 값에 의존하는 경우 ) 에는 `useDeferredValue`를 사용하면 많은 리렌더링 중에도 제어된 input 이 응답하도록 할 수 있음.
