# useImperativeHandle

ref로 노출되는 핸들을 직접 정의할 수 있게 해주는 훅이다.

# Reference

```js
import { forwardRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  useImperativeHandle(ref, () => {
    return {
      // ... your methods ...
    };
  }, []);
  // ...
```

## Parameters

### ref
- forwardRef 함수에서 두 번째 인자로 받은 ref

### createHandle
- 노출하려는 ref 핸들을 반환하는 함수
- 인자는 없다.
- 어떤 방식으로 작성하든 상관 없지만, 일반적으로 노출하려는 메서드가 있는 객체를 반환한다.

### (optional) dependencies
- createHandle 코드 내에서 참조하는 모든 반응형 값을 나열한 목록
- 이 반응형 값은 props, state 및 컴포넌트 내에서 직접 선언한 모든 변수와 함수를 포함한다.
- 이것도 역시나 린터가 올바르게 의존성을 지정했는지 확인해줌 👍
- 이 의존성이 변경되거나 인수를 생략하면 createHandle 함수가 다시 실행되고 새로 생성된 핸들이 ref에 할당된다.

## Returns
- undefined를 반환한다.

# Usage
## 부모 컴포넌트에 커스텀 ref 핸들 노출

- 기본적으로 컴포넌트는 자식 컴포넌트의 DOM 노드를 부모 컴포넌트에 노출하지 않는다.
- 따라서 부모 컴포넌트가 자식의 DOM 노드에 접근하려면 forwardRef를 사용해서 선택적으로 참조에 포함시켜줘야 한다.

```js
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  return <input {...props} ref={ref} />;
});
```

- 위 코드에서는 ref로 `<input>`의 DOM 노드를 받게 되는데, 커스텀한 값을 노출할 수 있다.
- 이것을 커스텀하려면 useImperativeHandle을 사용하면 되는 것!

```js
import { forwardRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  useImperativeHandle(ref, () => {
    return {
      // ... your methods ...
    };
  }, []);

  return <input {...props} />;
});
```

- 이렇게 작성하면 이제 ref에 `<input>`이 전달되지 않게 된다.

focus와 scrollIntoView만 노출시켜보자
그러기 위해서는 실제 DOM을 별도의 ref에 유지해야 한다.

```js
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      focus() {
        inputRef.current.focus();
      },
      scrollIntoView() {
        inputRef.current.scrollIntoView();
      },
    };
  }, []);

  return <input {...props} ref={inputRef} />;
});
```

## 사용자 정의 명령 노출

- imperative handle을 통해 노출하는 메서드가 DOM 메서드일 필요는 없다.

아래의 Post 컴포넌트는 imperative handle을 통해 `ScrollAndFocusAddComment` 메서드를 표시한다.

```js
const Post = forwardRef((props, ref) => {
  const commentsRef = useRef(null);
  const addCommentRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      scrollAndFocusAddComment() {
        commentsRef.current.scrollToBottom();
        addCommentRef.current.focus();
      }
    };
  }, []);

  return (
    <>
      <article>
        <p>Welcome to my blog!</p>
      </article>
      <CommentList ref={commentsRef} />
      <AddComment ref={addCommentRef} />
    </>
  );
});
```

# Pitfall
## ref를 과도하게 사용하지 말 것!
- ref는 props로 표현할 수 없는 필수적인 행동에만 써야 한다.
ex) 특정 노드로 스크롤하기, 초점 맞추기, 애니메이션 촉발하기, 텍스트 선택하기 등

- props으로 표현할 수 있는 것은 ref를 사용하지 말아야 한다.
ex) Modal 컴포넌트에서 {open, close}와 같은 imperative handle을 노출하는 대신 isOpen 등의 prop을 사용하고, Effect를 통해 명령형 동작을 노출할 수 있다.
