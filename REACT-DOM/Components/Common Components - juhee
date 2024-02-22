`<div>`같이 모든 내장 브라우저 빌트인 컴포넌트에 대해 알아보자

# React Props
### children
- React 노드
- JSX 내부에 콘텐츠를 넣으면 암묵적으로 children prop으로 지정된다.

### dangerouslySetInnerHTML
- 원시 
- 내부 HTML을 신뢰할 수 없는 경우 XSS 취약점이 발생할 수 있다.

### ref
- useRef나 createRef의 ref 객체 또는 ref 콜백 함수

### suppressContentEditableWarning
- 타입: boolean
- true - 같이 사용하지 않는 children과 
- 콘텐츠를 수동으로 관리하는 텍스트 입력 라이브러리를 빌드할 때 사용된다.

### suppressHydrationWarning
- 

### style
- CSS 스타일이 있는 객체
- 단위를 입력하지 않으면 px로 자동으로 추가된다.


# 표준 DOM Props
### accessKey

### aria-*
- 화면 리더기 같은 접근성을 지정할 때 사용

### contentEditable
- 사용자가 렌더링 된 엘리먼트를 직접 편집할 수 있다.
- 

### data-*
- 

### dir
- 

### draggable
- 

### enterKeyHint
- 

### htmlFor
- 

### hidden

### id
- 타입: 문자열

### is

### onCompositionStart
- 글자를 입력하기 시작할 때 발생한다.

### onCompositionUpdate
- 

### onCompositionEnd
- 

### onContextMenuCapture
- 우클릭 했을 때 나오는 메뉴

### onDragOver
- 드래그된 콘텐츠를 드래그하는 동안 유효한 드롭 대상에서 발생한다.
- 이때, 드롭을 허용하려면 `e.preventDefault()`를 호출해야 한다.
- html 요소의 위에는 다른 요소를 올릴 수 없기 때문에 기본 동작을 막아야 하는 것이다.

### onFocus
- 브라우저의 focus와 달리 버블링이 발생한다.

### onPointerDown
- 포인터가 활성화되면 발생한다.
- onMouseDown은 엘리먼트 위에서만 작동하고, onPointerDown은 document 등 더 큰 범위에서 작동한다. 
- onPointerDown은 가운데 스크롤 버튼과 오른쪽 버튼도 인식한다.

사용자 정의 어트리뷰트의 이름은 소문자여야 한다.

# 주의사항
- children과 dangerouslySetInnerHTML을 동시에 전달할 수 없다.
- 일부 이벤트에서는 브라우저에서는 버블링이 발생하지 않는 이벤트이지만 리액트에서는 버블링이 발생하는 경우가 있다.

# ref 콜백 함수
- useRef에서 반환되는 ref 객체 대신 ref 속성에 함수를 전달할 수 있다.
- 노드를 인수로 써서 함수를 호출하게 된다.

# React 이벤트 객체
- 이벤트 핸들러는 React 이벤트 객체를 받게 되며, '합성 이벤트'라고도 한다.
- 기본 DOM 이벤트와 같은 표준을 준수하지만 일부 브라우저의 불일치를 수정한다.

# 주의사항
- currentTarget, eventPhase, target, type의 값은 React 코드가 예상하는 값을 반영한다.
- 
