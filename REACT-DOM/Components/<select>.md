# `<select>`
- select box를 표시하는 내장 컴포넌트
- select box를 제어하려면 `value` prop 전달, 제어하지 않는다면 `defaultValue` prop 전달 (HTML에서처럼 `selected` 속성을 `<option>`에 전달하는 것은 지원하지 않음)
  - `value`: `<select>` 내부의 `<option>`의 `value`와 일치하는 String 값(어떤 옵션을 선택할지 제어하는 용도), `value`를 사용할거면 `value`를 동기적으로 업데이트하는 `onChange` 핸들러가 반드시 필요하다.
  - `defaultValue`: 초기 선택 옵션을 지정하는 string 값
- 제어 상태와 비제어 상태를 동시에 가질 수는 없다
- 일반 엘리먼트 props에 더해 `<select>`만의 props 지원
  - `autoComplete`: 자동완성 동작 중 하나를 지정하는 string 값
  - `autoFocus`: `true`면 마운트 시에 해당 엘리먼트에 포커스되는 불리언 값
  - `children`: `<select>`는 `<option>`, `<optgroup>`, `<datalist>` 컴포넌트를 children으로 받는다. 이 외에도 결과적으로 허용된 컴포넌트 중 하나를 렌더링하기만 한다면 자신만의 컴포넌트도 전달할 수 있다. 만약 최종적으로 `<option>` 태그를 렌더링하는 자신만의 컴포넌트를 전달한다면, 각각의 `<option>`은 `value`를 가지고 있어야 한다.
  - `disabled`: `true`일 때 select box가 희미하게 표시되고 선택할 수 없게 되는 불리언 값
  - `form`: 이 select box가 속한 `<form>`의 `id`를 지정하는 string 값, 생략하면 가장 가까운 부모 form으로 지정된다.
  - `multiple`: `true`일 때 다중 선택이 허용되는 불리언 값
  - `name`: form과 함게 제출되는 select box의 이름을 지정하는 string 값
  - `onChange`: 제어되는 select box에 필수적인 이벤트 핸들러 함수. 사용자가 다른 옵션을 선택하는 즉시 호출된다. 브라우저의 `input` 이벤트처럼 동작한다.
  - `onInput`: 사용자에 의해 value가 바뀌는 즉시 호출되는 이벤트 핸들러 함수. 리액트에서는 관용적으로 비슷하게 동작하는 `onChange`를 대신 사용한다.
  - `onInvalid`: form 제출에서 input의 유효성 검사에 실패할 때 호출되는 이벤트 핸들러 함수. 내장 `invalid` 이벤트와 다르게 리액트에서는 버블링된다.
  - `onChange`, `onInput`, `onInvalidCapture`: 캡쳐 단계에서 호출되는 버전의 `onChange`, `onInput`, `onInvalid`
  - `requried`: `true`이면 값을 입력해야 form이 제출될 수 있는 불리언 값
  - `size`: `multiple={true}`일 때 처음에 표시할 기본 항목 수를 지정하는 숫자

## select box가 포함된 라벨 제공
- `<label>` 태그 안에 `<select>`를 배치하면 브라우저가 라벨과 select box가 연결되어 있음을 알게 되어 라벨을 클릭했을 때 자동으로 select box에 포커스된다.
- 라벨은 접근성을 위해서도 필수적 (select box에 포커스하면 스크린 리더가 라벨의 캡션을 알림)
- `<select>`를 `<label>` 안에 중첩시킬 수 없다면 `<select id>`와 `<label htmlFor>`에 같은 id를 전달해서 연결할 수 있다 (한 컴포넌트의 여러 인스턴스 간 충돌을 피하려면 `useId` 사용)

```jsx
// 방법 1
function Form1() {
  return (
    <label>
    Pick a fruit:
      <select name="selectedOption">
        <option value="a">a</option>
        <option value="b">b</option>
        <option value="c">c</option>
      </select>
    </label>
  )
}

// 방법 2
function Form2(){
  const selectId= useId();
  return (
    <label htmlFor={selectId}>
      Pick one:
    </label>
    <select id={selectId} name="selectedOption">
      <option value="c">c</option>
      <option value="d">d</option>
      <option value="e">e</option>
    </select>
  )
}
```

## 초기 선택 옵션 제공
- 기본값으로는 첫번째  `<option>`이 선택된다. 다른 옵션을 기본값으로 선택하려면 `<select>` 엘리먼트의 `defaultValue`에 해당하는 `<option>`의 `value`를 전달해야 한다.
- HTML에서 하던 것처럼 `<option>`에 `selected` 속성을 전달하는 방식은 지원하지 않음을 주의!
```jsx
<label>
      Pick a fruit:
      <select name="selectedFruit" defaultValue="orange">
        <option value="apple">Apple</option>
        <option value="banana">Banana</option>
        <option value="orange">Orange</option>
      </select>
    </label>
```

## 다중 선택 활성화
- `<select>`에 `multiple={true}`를 전달하면 다중 선택 가능
- 초기 선택 옵션을 지정할때는 `defaultValue`에 배열 전달
```jsx
function FruitPicker() {
  return (
    <label>
      Pick some fruits:
      <select
        name="selectedFruit"
        defaultValue={['orange', 'banana']}
        multiple={true}
      >
        <option value="apple">Apple</option>
        <option value="banana">Banana</option>
        <option value="orange">Orange</option>
      </select>
    </label>
  );
}
```
<img width="248" alt="image" src="https://github.com/Jungle-JavaScript-Study/react-docs-study/assets/70076564/eecefd13-15f7-44c3-83ed-c9fe7ce2a431">

## form 제출 시 select box의 값 읽기
- 내부에 `<button type="submit">`이 있는 select box 주변에 `<form>`을 추가하면 form의 `onSubmit` 이벤트 핸들러가 호출된다. 브라우저의 기본 동작으로는 양식 데이터를 현재 URL로 보내고 페이지를 새로고침한다.
- `e.preventDefault()`를 호출해 해당 동작을 덮어쓸 수 있다. form 데이터는 `new FormData(e.target)`로 읽는다.
- 이 때 반드시 `<select>`에 `name`을 지정해야 한다. 지정한 `name`은 form 데이터에서 키로 사용된다.<br>
  ex.) `<select name="selectedFruit" />` → `{ selectedFruit: "orange" }`
- `multiple={true}`일 때는 선택한 각 값들이 이름-값 쌍으로 구분되어 `FormData`에 포함된다.<br>
  ex.) `[["selectedFruit", "apple"], ["selectedVegetables", "corn"], ["selectedVegetables", "tomato"]]`
- 기본적으로 `<form>` 내부의 모든 `<button>`은 select box의 값을 제출한다. 의도한 동작이 아니라면, `<button>` 대신 `<button type="button">`으로 커스텀 Button 컴포넌트를 리턴할 수 있다. 그런 다음 폼을 제출해야 하는 버튼에 `<button type="submit">`을 사용해 명확하게 표시한다.

## state 변수로 select box 제어하기
- 제어되는 select box를 사용하려면 `value` prop을 전달해야 한다. 보통 상태값을 선언해 select box를 제어한다.
  <br>→ 매 선택마다 UI 일부를 리렌더링하고 싶을 때 유용
- `onChange` 없이 `value`를 전달하면 옵션을 선택할 수 없으므로 주의! `value`를 전달해 select box를 제어하는 것은 select box가 항상 전달한 그 값을 갖도록 강제하는 것이기 때문에 `onChange`에서 상태값을 동기적으로 업데이트 하지 않으면 select box는 매 keystroke마다 지정한 `value`로 되돌아간다.
