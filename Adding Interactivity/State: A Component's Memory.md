## **State - A Component's Memory**

- `state`는 **컴포넌트별** 메모리
- 컴포넌트 내에서 사용하는 지역 변수는 렌더링 간에 유지되지 않고, 변경해도 렌더링을 발동시키지 않음<br>
⇒ `useState` 훅이<br>
  1. 렌더링 사이에 데이터를 유지하기 위한 state 변수<br>
  2. 변수를 업데이트하고 React가 컴포넌트를 다시 렌더링하도록 촉발하는 state 설정자 함수<br>
를 제공함
- 배열 구조분해로 state 변수와 state 설정자 함수를 정의
`const [state, setState] = useState();`
- 훅은 리액트가 렌더링 중일 때만 사용할 수 있음 → 컴포넌트의 최상위 레벨이나 커스텀 훅에서만 호출할 수 있음(조건문, 반복문, 중첩 함수에서는 불가능)
⇒ 리액트가 컴포넌트의 호출 순서에 의존해서 state 변수를 기억하기 때문에
- 관련 있는 state들은 하나의 객체로 묶어서 하나의 state로 사용하는 것이 좋음(ex. 폼 필드)
- `state`는 private하기 때문에 동일한 컴포넌트를 두 군데에서 렌더링해도 독립적인 `state`를 가지며, 해당 컴포넌트 내에서만 사용할 수 있음
⇒ 두 컴포넌트 간 `state`를 공유하려면 [`state` 끌어올리기](https://github.com/Jungle-JavaScript-Study/react-docs-study/blame/de72488198685f6b599a1dd759215b9250ac17a1/Quick%20Start/Thinking%20in%20React.md#L9)로 가능
