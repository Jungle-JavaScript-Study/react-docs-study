# `<StrictMode>`
: 개발 중에 컴포넌트에서 흔히 발생하는 버그를 빠르게 발견하게 해주는 모드

- 컴포넌트 트리 내부에 대해 추가 개발 동작과 경고를 활성화할 수 있음
	- 순수하지 않은 렌더링으로 인한 버그를 찾기 위해 한번 더 렌더링
 	- Effect의 클린업 누락으로 인한 버그를 찾기 위해 Effect 한번 더 실행
  - 지원 중단된 API의 사용 여부 확인
- props를 허용하지 않음
- `<StrictMode>`로 감싸진 트래 내부에서 선택적으로 Strict 모드를 해제할 수 있는 방법은 없음(`<StrictMode>` 내의 모든 컴포넌트가 검사된다는 신뢰를 주기 위함)<br>
→ 하나의 프로덕트에 대해 작업하는 팀 간에 이 검사가 가치있는지에 대한 의견이 다르다면 합의하거나 `<StrictMode>`를 트리 아래쪽으로 옮기는 방법밖에 없음


## 전체 앱에 Strict Mode 활성화하기
- 루트 컴포넌트를 `<StrictMode>`로 감싸면 전체 앱에 대해 Strict 모드 적용 가능
- 전체 앱에 Strict 모드 사용하는 것을 권장함(특히 새로 만든 앱의 경우에는 더!)
- `createRoot`를 대신 호출해주는 프레임워크를 사용한다면 어떻게 Strict 모드를 활성화하는지 해당 프레임워크의 문서를 확인
- Strict 모드는 개발 환경에서만 실행되고, production 빌드에는 영향을 주지 않음
- 그러나 production 환경에서는 재현하기 까다로운 버그를 찾는 것을 도와줌<br>
→ 사용자가 버그를 신고하기 전에 수정할 수 있다!


## 앱의 일부에서 Strict 모드 사용하기
일부만 감싸면 앱의 일부분에만 적용 가능


## 개발 중에 이중 렌더링으로 발견된 버그 고치기
- 리액트에 작성하는 모든 컴포넌트는 순수 함수여야 함(=모든 컴포넌트가 항상 동일한 입력(props, state, context)에서 동일한 JSX를 리턴해야 함)
- Strict 모드에서는 이 규칙을 위반하여 예측할 수 없는 동작을 하고 버그를 일으키는, 의도하지 않은 "불순한 코드"를 찾기 위해 개발 모드에서 일부 함수를 두번 호출함(순수해야 하는 함수만!)
	- 컴포넌트 함수 본문(최상위 로직만, 이벤트핸들러 내부 코드는 비포함)
  - `useState`, `set` 함수, `useMemo`, `useReducer`에 전달한 함수
  - `constructor`, `render`, `shouldComponentUpdate` 같은 일부 클래스 컴포넌트 메서드
- ex.) 배열로 구성된 데이터를 바로 수정하면 버그가 발생하지만, Strict 모드가 꺼져 있을 때는 이를 바로 확인할 수 없음<br>
→ Strict 모드에서는 초기 렌더링 시에도 렌더링이 이중으로 되기 때문에 수정이 두 번 반영되어 에러가 바로 보임<br>
⇒ 배열을 복사(`arr.slice()`)하여 복사본을 수정하면 해결 가능

※ React DevTools가 설치돼있으면 두번째 렌더링에서의 `console.log`가 흐릿하게 표시됨(설정으로 조절 가능)


## 개발 중에 Effect를 재실행해서 발견되는 버그 고치기
- Strict 모드에서는 개발 환경의 모든 Effect에 대해 셋업+클린업 사이클을 한번 더 실행함(셋업→클린업→셋업)
- ex.) 채팅방 연결 버그: Effect 내에서 채팅방을 연결할 때 클린업에서 연결을 끊지 않으면 연결 수가 계속 늘어남


## Strict 모드가 발생시키는 지원 중단 경고 고치기
- `<StrictMode>` 트리 내부의 어떤 컴포넌트든 다음과 같은 지원 중단된 API를 사용하면 경고를 표시함
	- `findDOMNode`
	- `UNSAFE_componentWillMount` 같은 `UNSAFE_` 클래스 라이프 사이클 메서드
  - 레거시 컨텐스트 API(`childContextTypes`, `contextTypes`, `getChildContext`)
  - 레거시 문자열 ref(`this.refs`)
- 사실 주로 오래된 클래스 컴포넌트에서 사용하던 API들이라 요즘엔 잘 없음.. 