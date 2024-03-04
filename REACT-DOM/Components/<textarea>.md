# `<textarea>`

- 브라우저의 내장 `<textarea>`를 이용하면 여러 줄의 텍스트 입력을 렌더링할 수 있다.

# Props

- 모든 공통 element의 props를 지원한다.

### controlled 컴포넌트의 Props

- `value` prop을 전달해서 controlled 컴포넌트로 만들 수 있다.
  - `value`를 전달할 땐 반드시 value를 업데이트하는 `onChange` 핸들러도 함께 전달해야 한다.

### uncontrolled 컴포넌트의 Props

- uncontrolled 컴포넌트인 경우에는, 초깃값을 지정하는 `defaultValue`를 전달할 수 있다.

### 공통 Props

- autoComplete: `on | off` - 자동 완성 동작 지정
- autoFocus: `boolean` - true일 경우 마운트 시 초점
- children: 불가능! - 자식 요소를 받지 않는다.
- cols: `number` - 표준 문자 너비를 기준으로 기본 칸 수를 지정한다. 디폴트는 20
- disabled: `boolean` - true일 경우, 입력이 비활성화되고 흐릿하게 표시된다.
- form: `string` - `<textarea>`가 속한 `<form>`의 id를 지정한다. 디폴트는 가장 가까운 상위 form
- maxLength: `number` - 텍스트의 최대 길이
- minLength: `number` - 텍스트의 최소 길이
- name: `string` - 폼 제출 시 해당 textarea의 이름을 지정
- onChange: `Event handler` - controlled 컴포넌트로 사용할 때 필요하다. 사용자에 의해 입력 값이 변경되는 즉시 실행된다. 브라우저의 Input event처럼 동작
- onChangeCapture: 캡쳐 단계에 실행되는 onChange
- onInput: `Event handler` - React에서는 비슷하게 작동하는 onChange를 사용하는 것이 관례
- onInputCapture: 캡쳐 단계에 실행되는 onInput
- onInvalid: `Event handler` - 폼 제출 시 유효성 검사에 실패하면 발생. 빌트인 invalid 이벤트와 달리 이벤트 버블링이 발생한다.
- onInvalidCapture: 캡쳐 단계에 실행되는 onInvalid
- onSelect: `Event handler` - 내부 선택 영역이 변경되면 발생. React에서는 onSelect를 확장하여 empty selection(선택 없이 포커싱만)과 선택에 영향을 줄 수 있는 편집에도 발동된다.
- onSelectCapture: 캡쳐 단계에 실행되는 onSelect
- placeholder: `string`
- readOnly: `boolean`
- required: `boolean` - true일 경우 폼 제출 시 값이 있어야 한다.
- rows: `number` - 표준 문자 높이를 기준으로 기본 줄 수를 지정. default는 2
- wrap: `hard | soft | off` - 폼 제출 시 텍스트를 어떻게 줄바꿈할지 지정

<br/>

# 주의사항

- controlled 컴포넌트이면서 동시에 uncontrolled 컴포넌트일 수는 없다.
- 생명주기 동안 controlled 컴포넌트와 uncontrolled 컴포넌트 사이를 전환할 수 없다.

<br/>

# 사용법

## textarea 크기 재조정

- rows와 cols 속성으로 기본 크기를 정할 수 있지만, 사용자가 재조정할 수 있다.
  - 재조정을 비활성화하려면 CSS로 `resize: none`을 지정하면 됨

## textarea에 label 제공하기

- 보통 `<textarea>`를 `<label>` 태그 안에 위치시킨다.
  - 이렇게 하면 해당 label이 textarea와 연결되어 있음을 의미한다.
- label을 클릭하면 textarea가 포커싱된다.
- textarea에 포커싱하면 스크린 리더가 label 캡션을 읽어주므로, 접근성을 위해 이렇게 하는 것이 필수적이다.
- `<textarea>`를 `<label>` 안에 넣을 수 없는 경우, `<textarea id>`와 `<label htmlFor>`에 동일한 ID를 전달해서 연결한다.
  - 한 컴포넌트에서 여러 인스턴스 간의 충돌을 피하기 위해 `useId`로 생성하자

```js
import { useId } from "react";

export default function Form() {
  const postTextAreaId = useId();
  return (
    <>
      <label htmlFor={postTextAreaId}>Write your post:</label>
      <textarea id={postTextAreaId} name="postContent" rows={4} cols={40} />
    </>
  );
}
```

## textarea 초기값 제공하기

- defaultValue를 사용한다.
  ```js
  <textarea
    name="postContent"
    defaultValue="I really enjoyed biking yesterday!"
    rows={4}
    cols={40}
  />
  ```
- HTML과 달리 초기 텍스트를 자식 요소로 전달하는 것은 지원하지 않는다. ❌
  ```js
  <textarea>Some content</textarea>
  ```

## form 제출시 textarea 값 읽기

- form 데이터를 읽으려면 `new FormData(e.target)`

  ```js
  export default function EditPost() {
    function handleSubmit(e) {
      // Prevent the browser from reloading the page
      e.preventDefault();

      // Read the form data
      const form = e.target;
      const formData = new FormData(form);

      // You can pass formData as a fetch body directly:
      fetch("/some-api", { method: form.method, body: formData });

      // Or you can work with it as a plain object:
      const formJson = Object.fromEntries(formData.entries());
      console.log(formJson);
    }

    return (
      <form method="post" onSubmit={handleSubmit}>
        <label>
          Post title: <input name="postTitle" defaultValue="Biking" />
        </label>
        <label>
          Edit your post:
          <textarea
            name="postContent"
            defaultValue="I really enjoyed biking yesterday!"
            rows={4}
            cols={40}
          />
        </label>
        <hr />
        <button type="reset">Reset edits</button>
        <button type="submit">Save post</button>
      </form>
    );
  }
  ```

## state 변수로 textarea 제어하기

- controlled 컴포넌트로 렌더링하려면 Value prop을 전달한다.
  - 일반적으로 state 변수로 textarea를 제어한다.
- 모든 키 입력에 응답하여 UI의 일부를 다시 렌더링하려는 경우 유용하다.
- onChange 없이 value만 전달하면 입력을 할 수 없고 콘솔 에러남
  - 의도적으로 읽기전용으로 하려면 readOnly를 추가하자!

```js
function NewPost() {
  const [postContent, setPostContent] = useState(""); // Declare a state variable...
  // ...
  return (
    <textarea
      value={postContent} // ...force the input's value to match the state variable...
      onChange={(e) => setPostContent(e.target.value)} // ... and update the state variable on any edits!
    />
  );
}
```

<br/>

# Troubleshooting

## 키를 누를 때마다 커서가 처음으로 이동함

- controlled textarea는 onChange 중에 state 변수를 DOM의 값으로 업데이트한다.
- 이때 `e.target.value`가 아닌 다른 값으로 업데이트할 수 없다.
  ```js
  function handleChange(e) {
    // 🔴 Bug: updating an input to something other than e.target.value
    setFirstName(e.target.value.toUpperCase());
  }
  ```
- 비동기적으로 업데이트하는 것도 안됨
  ```js
  function handleChange(e) {
    // 🔴 Bug: updating an input asynchronously
    setTimeout(() => {
      setFirstName(e.target.value);
    }, 100);
  }
  ```
- 위 두 문제가 아니라면, 키 입력 시마다 textarea가 DOM에서 제거되었다가 다시 추가되는 상황일 수 있다.
  - 렌더링 마다 state를 재설정하는 경우 이런 문제가 발생할 수 있다.
    - 예를 들어, textarea 또는 그 부모 중 하나가 항상 다른 key 속성을 받거나
    - 컴포넌트 정의를 중첩하는 경우 (이렇게 하면 렌더링 마다 내부 컴포넌트가 다시 마운트됨. 원래 이런 짓 하면 안됨)

## A component is changing an uncontrolled input to be controlled

- '컴포넌트가 비제어 입력을 제어하도록 변경하고 있습니다' 오류
- 컴포넌트에 value를 제공하는 경우, 그 값은 생명주기 동안 계속 문자열로 유지되어야 한다.
- React는 컴포넌트를 uncontrolled로 둘지 controlled로 둘지 알 수 없기 때문에 value에 undefined를 전달하고 나중에 string을 전달하는 방법은 쓸 수 없다.
- 제어 컴포넌트는 항상 string value를 받아야 한다. value에 항상 문자열이 오도록 하자! `value={someValue ?? ''}`
