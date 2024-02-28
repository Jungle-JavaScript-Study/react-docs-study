# `<option>`
- 브라우저 내장 컴포넌트로 `<select>` 박스 안에 옵션을 렌더링할 때 사용
- 모든 일반 엘리먼트 props에 더해 `disabled`, `label`, `value`를 지원
  - `disabled`: 옵션을 선택할 수 있는지에 대한 불리언 타입, `true`이면 흐리게 표시됨
  - `label`: 옵션의 의미를 지정하는 문자열 타입, 지정하지 않으면 옵션 내부의 텍스트가 사용됨
  - `value`: 폼에서 `<select>`를 제출할 때 어떤 옵션을 선택했는지 전달할 값
- 리액트에서는 `<option>`에서 `selected` 속성을 사용할 수 없음. 대신 제어되지 않은 select box에서는 `value`를 `<select defaultValue>`에 전달하거나, 제어되는 select box에서는 `<select value>`에 전달해서 같은 동작을 하도록 함.

## 옵션이 포함된 select box 표시하기
- `<select>` 안에 `<option>` 컴포넌트 목록을 넣어 select box 표시
- 각 `<option>`에 `value`를 지정해 폼 제출 시 제출할 데이터를 지정
```javascript
export default function selectBox() {
  return (
    <label>
      Select:
      <select name="selectedOption">
        <option value="yes">Yes</option>
        <option value="no">No</option>
      </select>
    </label>
  );
}
```
