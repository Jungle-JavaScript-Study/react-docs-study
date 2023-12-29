# 1. 이벤트 처리
## 1-1. 이벤트 핸들러 추가하기

### 1) 이벤트 핸들러란?
- 클릭, 마우스 호버링, input 포커싱 등의 사용자 상호작용에 반응하여 촉발되는 함수

```jsx
export default function Button() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}

// 인라인으로 정의하는 것도 가능
<button onClick={() => {
  alert('You clicked me!');
}}>
  
// 함수형
<button onClick={function handleClick() {
  alert('You clicked me!');
}}>

```

<br/>

### 2) 전달 ⭕️, 호출 ❌

- 이벤트 핸들러를 전달이 아닌 호출을 하면 렌더링 중에 즉시 함수가 실행되므로 주의!

```jsx
<button onClick={handleClick}> // ⭕️ 전달
<button onClick={handleClick()}> // ❌ 호출 -> 이렇게 하면 안 됨
```

- 인라인으로 작성할 때는 익명 함수로 감싸야 한다.

```jsx
<button onClick={() => alert('...')}> // ⭕️ 이 함수를 '전달'함
<button onClick={alert('...')}> // ❌ 호출 -> 이렇게 하면 클릭시 실행되지 않고 컴포넌트가 렌더링될 때마다 실행된다.
```

<br/>

### 3) 네이밍

- 이벤트 핸들러 네이밍: `handle + 이벤트 이름`
- 이벤트 핸들러를 props로 보낼 경우에는 핸들러 선언은 `handle + 이벤트 이름`로 하고, props로 전달할 때는 `on + 이름`으로 보낸다. (부모에서 선언하고 자식이 사용하게 만들 때는 이 방식을 쓴다.)
- props의 이름을 지을 때는 `onPlayMovie`와 같이 상호작용을 나타내는 이름을 지정하자.

```jsx
// 자식은 onClick으로 받음
function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}

// 부모는 handlePlayClick을 만들어서 onClick으로 보냄
function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`Playing ${movieName}!`);
  }

  return (
    <Button onClick={handlePlayClick}>
      Play "{movieName}"
    </Button>
  );
}
```

<br/>

## 1-2. 이벤트 전파 (Event propagation)
- 이벤트 핸들러는 컴포넌트의 모든 자식들의 이벤트도 포착한다.
- 이벤트가 트리 위로 버블 또는 전파되는 것을 이벤트가 발생한 곳에서 시작하여 트리 위로 올라간다고 말한다.
- 즉, 이벤트는 이벤트가 발생한 곳에서 시작해 상위 트리로 이동한다.
- **예외)** `onScroll`은 전파되지 않는다! `onScroll`은 첨부한 JSX 태그에서만 작동한다. 이 이벤트를 제외한 모든 이벤트는 전파된다.

아래 코드에서 버튼을 누르면 버튼의 `onClick`이 실행되고, 이어서 부모 div의 `onClick`도 실행된다.

```jsx
export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('You clicked on the toolbar!');
    }}>
      <button onClick={() => alert('Playing!')}>
        Play Movie
      </button>
    </div>
  );
}
```

<br/>

## 1-3. 전파 중지하기
- 이벤트 핸들러는 **이벤트 객체**를 유일한 인수로 받는데, 이 이벤트 객체를 사용해 이벤트가 상위 컴포넌트에 도달하지 못하도록 전파를 중지할 수 있다.

```js
e.stopPropagation();
```

<br/>

## 1-4. 캡처 단계 이벤트
- 전파가 중지된 경우에도 하위 요소의 모든 이벤트를 포착하고 싶다면?!
ex) 전파 로직에 관계 없이 모든 클릭을 분석을 위해 로깅하려는 경우

- 이벤트 이름 끝에 `Capture`를 추가하면 된다.
- `onClickCapture`는 이벤트 처리의 "캡처 단계"에서 발생하는 클릭 이벤트를 처리하는 메서드이다.
- Capture 단계에서는 이벤트가 하위에서 상위로 이동하며, Bubble 단계에서는 이벤트가 상위에서 하위로 이동한다.

```jsx
<div onClickCapture={() => { /* 여기 내용이 먼저 실행됨 */ }}>
  <button onClick={e => e.stopPropagation()} />
  <button onClick={e => e.stopPropagation()} />
</div>
```

각 이벤트는 세 단계로 전파된다.
1) 아래로 이동하면서 모든 `onClickCapture`를 호출한다.
2) 클릭한 요소의 `onClick` 핸들러가 실행된다.
3) 상위로 이동하면서 모든 `onClick` 핸들러를 호출한다.

<br/>

## 1-5. 전파의 대안으로 핸들러 전달하기
- 이벤트 전파는 자식 요소에서 발생한 이벤트가 부모 요소로 전달되는 과정이다. 이 이벤트 전파를 이용하지 않고, 부모 컴포넌트에서 전달된 이벤트 핸들러를 **직접 호출**해보자!
- 이 패턴은 이벤트 전파가 복잡해질 경우, 어떤 핸들러가 언제 실행되는지 더 명확하게 파악할 수 있게 해준다.

```jsx
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onClick(); // 👈
    }}>
      {children}
    </button>
  );
}
```

- `onClick`: 부모 컴포넌트에서 전달된 클릭 이벤트 핸들러
- `children`: 버튼 내부에 표시할 내용
- `e.stopPropagation();` : 이벤트 전파를 멈춘다. 즉, 이 이벤트가 부모 요소로 전파되지 않도록 한다.
- `onClick()` : 부모 컴포넌트에서 전달 받은 `onClick`을 호출한다.
- 이렇게 하면 자식 컴포넌트가 부모의 `onClick` 이벤트 핸들러를 호출하기 전에 추가적인 동작을 수행할 수 있다. (`onClick()` 전에 작성하면 되겠지!)


### 언제 씀?
- 특정 이벤트에 대한 세밀한 제어가 필요한 경우 
ex) 버튼 클릭 시 특정 로직을 수행한 후 부모 컴포넌트의 이벤트 핸들러를 호출하고 싶을 때

- 이벤트 전파로 인한 부작용 방지: 특정 DOM 요소에서 이벤트가 발생했을 때 상위 요소로의 이벤트 전파를 원하지 않는 경우 유용하다. 
ex) 중첩된 버튼에서 상위 버튼의 클릭 이벤트가 발생하는 것을 방지하고 싶을 때


<br/>

## 1-6. 기본 동작 방지
- 이벤트에 연결된 기본 브라우저 등록을 막을 수 있다.
ex) `<form>`의 `submit` 이벤트는 내부의 버튼을 클릭할 때 발생하며 전체 페이지를 다시 로드한다.

```js
e.preventDefault()
```

<br/>
