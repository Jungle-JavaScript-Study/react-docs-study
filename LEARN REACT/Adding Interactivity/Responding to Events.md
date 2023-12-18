## **Responding to Events**

이벤트 핸들러: 이벤트(상호작용)에 반응해서 발생하는 함수, 유일한 인수로 `event` 객체를 받음

### 이벤트 핸들러 추가하는 방법

1. 기명 함수를 JSX 태그에 prop으로 전달 (`onClick={handleClick}`)
    - 일반적으로 컴포넌트 안에 `handle`+EventName 으로 명명
2. JSX에서 인라인으로 기명 함수 정의 (`onClick={function handleClick() {...}}`)
3. JSX에서 인라인으로 익명 함수 전달 (`onClick={() ⇒ {...}}`)
- 이벤트 핸들러는 정의되는 컴포넌트의 props 접근 가능
  
※ 주의 사항 - 호출이 아니라 **전달**해야 React가 이게 이벤트 핸들러구나~하고 기억해뒀다 버튼 클릭 시 호출함<br>
`<button onClick={handleClick()}>`은 잘못된 방식, 이벤트 핸들러로 전달하는게 아니라 렌더링 중에 즉시 호출됨<br>
인라인 코드도 마찬가지로 `onClick={...}`는 렌더링마다 실행됨

### 이벤트 핸들러 props로 전달하기

```jsx
function Button({ clickHandler, children }) {
  return (
    <button onClick={clickHandler}>
      {children}
    </button>
  );
}

// 사용
<Button clickHandler={() => {alert("!")}}>
  Play Movie
</Button>
```

- 컴포넌트 여는 태그와 닫는 태그 사이의 내용이 props의 `children`으로 전달됨
- 디자인 시스템을 사용하는 경우 이런 식으로 이벤트 핸들러를 전달해 모든 컴포넌트에 동일한 스타일을 적용하되 동작은 다르게 할 수 있음
- 웹 접근성을 위해 적절한 태그를 사용해야 함. 원하는 모양은 CSS로 조절 가능<br>
참고) [HTML: 접근성의 좋은 기반 - Web 개발 학습하기 | MDN](https://developer.mozilla.org/ko/docs/Learn/Accessibility/HTML)  
- 컴포넌트 추상화를 위해 이벤트 핸들러 이름을 HTML attribute에는 없는 값으로 작명할 수 있음.<br>예를 들어, `<Toolbar onPlayMovie={...} onUploadImage={...}/>`라고 하면 `Toolbar` 내에서는 해당 핸들러가 무슨 동작을 하는지에 관계 없이 사용할 수 있음. 이는 핸들러가 클릭 이벤트에 붙든, 키보드 이벤트에 붙든 상관 없이 유연하게 동작할 수 있도록 함.

### 이벤트 전파
: `onScroll`을 제외한 모든 이벤트는 이벤트가 발생한 컴포넌트의 상위로 전달되며 이벤트 핸들러를 실행하게 함.

- 이벤트 핸들러에서 `e.stopPropagation()`을 호출함으로써 전파 중지
- 예를 들어 클릭할 수 있는 컴포넌트 내에 클릭할 수 있는 또 따른 컴포넌트가 있을 때, 하위 컴포넌트에서 이벤트 전파를 중지해야 상위의 이벤트핸들러가 실행되지 않음
※  `on*Event*Capture`에 이벤트 핸들러를 전달하면 가장 먼저 하위로 이동하면서 모든 `on*Event*Capture` 핸들러를 호출한 뒤 상위로 전파됨 (분석에 주로 사용, 앱 코드에서는 활용도 낮음)
- `e.preventDefault()`는 이벤트의 기본 브라우저 동작을 중지함(ex. `onSubmit`은 페이지 새로고침)
