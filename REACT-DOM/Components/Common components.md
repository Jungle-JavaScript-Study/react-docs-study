# 공통 컴포넌트(예: `<div>`)
## props
### 모든 컴포넌트에서 지원하는 React props
- `children`: React 노드를 받아 컴포넌트 내부의 콘텐츠를 지정하는데 사용. JSX에서 태그를 중첩해서 사용하는 것은 암묵적으로 `children` prop을 지정한 것
- `ref`: `useRef`/`createRef`에서 사용하는 ref 객체 or ref 콜백 함수 or 레거시 refs. 해당하는 노드의 DOM 엘리먼트가 들어간다.
- `style`: css 스타일링 객체. camelCase로 작성해야 함. 값으로 숫자를 전달하면 자동으로 기본 단위를 리액트가 추가해줌(ex. `width: 100` -> `width: 100px`). 일반적으로는 `className`으로 css를 작성하고 동적으로 스타일링 할 때만 사용할 것을 권장.

### 모든 컴포넌트에서 지원하는 표준 DOM props
- `id`: 컴포넌트의 여러 인스턴스 간 충돌을 피하고자 `useId`로 생성해서 넣어야 한다.
- `onCompositionStart`: 입력 메서드 편집기가 새로운 구성 세션을 시작할 때, 즉 새로운 한 글자가 생성되기 시작할 때 발생한다.
- `onContextMenu`: 일반적으로 마우스 우클릭했을 때 나오는 컨텍스트 메뉴를 열려고 할 때 발생.
- `onDragOver`: 드래그하는 동안 유효한 드롭 대상(드롭존)에서 발생. 기본적으로 HTML 요소는 다른 요소 위에 존재할 수 없기 때문에 여기서 `e.preventDefault()`를 호출해야 드롭할 수 있다.
- `onPointerDown`: 포인터가 활성화 됐을 때 발생. `mouseDown`은 엘리먼트에서만 발생하지만 `pointerDown`은 Document와 Window에서도 가능. 또한 왼쪽 클릭만 인식하는 `onClick`과 다르게, 가운데와 오른쪽 버튼 다운도 인식.

### 주의 사항
- `children`과 `dangerouslySetInnerHTML`을 동시에 전달할 수 없다.
- 일부 이벤트는 브라우저에서는 버블링이 발생하지 않지만, React에서는 버블링이 발생한다.
- 사용자 정의 어트리뷰트를 props로 전달할 수도 있다. (서트파티 라이브러리 사용할 때 유용)

## ref 콜백 함수
- `ref` 속성에 `useRef`에서 리턴하는 ref 객체 대신 함수를 전달할 수 있음. DOM 노드가 화면에 추가될 때 노드를 인수로 `ref` 콜백이 호출되고, 해당 DOM 노드가 제거되면 `null`을 인수로 호출된다. 다른 콜백 함수를 전달할때마다 콜백을 호출하는데, 화살표 함수와 같은 함수들은 모든 렌더링에서 다른 함수이기 때문에 컴포넌트가 리렌더링될 때, 이전 렌더링에서의 함수는 `null`로 호출되고 다음 렌더링의 함수가 DOM 노드로 호출된다.

## React 이벤트 객체
- 이벤트 핸들러는 React 이벤트 객체(=합성 이벤트)를 받는다
- 기본적으로는 기본 DOM 이벤트와 일치하지만 일부는 일치하지 않는다. 기본 브라우저 이벤트 객체가 필요하면 `e.nativeEvent`에서 읽어와야 한다.
- `currentTarget`, `eventPhase`, `target`, `type`의 값은 React가 예상하는 값이기 때문에 기본 브라우저 이벤트의 값과 일치하지 않을 수 있다.

## 사용법
### CSS 스타일 적용하기
- `className`을 사용해 클래스를 지정하면 HTML의 `class` 속성처럼 작동함.
- 리액트에서는 CSS 파일을 추가하는 방법이 지정되어있지 않음. 빌드 도구나 프레임워크를 사용하고 있다면 그쪽의 문서를 참고할 것.
- 동적으로 스타일링 하려면 `style` 어트리뷰트에 객체를 전달(스타일이 변수에 의존하는 경우에만 사용 권장)
  ```javascript
  // 이중 괄호는 "중괄호가 있는 JSX + 일반 객체"의 조합이기 때문
  <img style={{width: user.imageSize}} />
  ```
  - 조건부로 CSS 클래스를 적용하려면 조건부로 생성되는 문자열을 `className`에 전달. 가독성을 위해 `classnames` 라이브러리 사용 가능(조건부 클래스가 여러 개일 때 편리)<br>
    ex.) `className={'row ' + (isSelected ? 'selected': '')}`

### ref로 DOM 노드 조작하기
- `useRef`로 ref를 선언하고 태그에 `ref` 어트리뷰트로 전달해서 브라우저의 DOM 노드 가져오기
- DOM 노드가 화면에 렌더링 되고 나면 `inputRef.current`에 DOM 노드가 할당됨

### dangerously setting inner HTML
- `dangerouslySetInnerHTML`로 HTML 문자열을 엘리먼트에 전달할 수 있지만 XSS 취약점이 있음.
- 신뢰할 수 있고 유해한 정보가 포함되어 있지 않은 데이터를 사용할 때만 `dangerouslySetInnerHTML`을 사용해야 함.

### 포커스 이벤트 처리
- 리액트에서는 포커스 이벤트가 버블링된다.
- 부모 엘리먼트의 바깥 부분에서 발생한 이벤트가 focus 혹은 blur인지 구분하기 위해 `currentTarget`과 `relatedTarget` 사용
- `target`: 이벤트가 발생한 요소. 예를 들어, 클릭 이벤트가 버튼에서 발생했다면 해당 버튼 요소.<br>
  `currentTarget`: 이벤트 버블링이나 캡처링이 발생할 때 이벤트 핸들러가 현재 이벤트를 처리하고 있는 요소.<br>
  `relatedTarget`: 이벤트가 특정 요소로 이동할 때 발생하는 이벤트에서, 이전에 포커스되었던 요소(onFocus) 또는 포커싱이 이동한 요소(onBlur).
