# 3. Effect와 동기화하기

컴포넌트가 외부 시스템과 동기화해야 하는 경우가 있다.
- state를 기반으로 react가 아닌 컴포넌트를 제어하거나
- 서버 연결을 설정하거나
- 컴포넌트가 화면에 나타날 때 분석 로그를 보내기 위해

Effect를 사용하면 렌더링 이후에 코드를 실행할 수 있어서 이런 일들을 할 수 있다.

## 3-1. Effect가 뭔가요

### 1) 렌더링 코드와 이벤트 핸들러
우선 React 컴포넌트 내부에서 두 가지 유형의 로직을 다시 떠올려보자

#### 렌더링 코드
- 렌더링 코드는 컴포넌트의 최상위에 존재한다.
- 여기서 props와 상태를 가져와 변환하고, 화면에 표시하려는 JSX를 반환한다.
- 이 렌더링 코드는 순수해야 한다. (수학 공식처럼 결과만을 계산해야 한다. 다른 일은 하지 않아야 한다.)

#### 이벤트 핸들러
- 이벤트 핸들러는 컴포넌트 내의 중첩된 함수로, 단지 계산하는 것이 아니라 실제로 작업을 수행한다.
- 특정 사용자 동작에 의해 발생하는 side effect(ex. 프로그램의 상태 변경)를 포함한다.

때로는 이 둘만으로 충분하지 않다.

### 2) ex. 채팅 서버에 연결하기
화면에 표시될 때마다 채팅 서버에 연결해야 하는 ChatRoom 컴포넌트가 있다고 해보자.
서버에 연결하는 것은 순수한 계산이 아닌 side effect이므로 렌더링 중에는 일어날 수 없다.
또한 ChatRoom이 표시되게 하는 특정한 이벤트가 존재하지도 않는다.

#### 🧐 서버에 연결하는 작업이 순수한 계산이 아닌 이유
- 외부 상태에 영향을 받는다.
서버에 연결하려면 네트워크 상태, 서버의 상태, 인증 정보 등 외부 시스템의 상태에 의존해야 한다.
- 부작용을 발생시킨다.
서버에 연결하는 동작은 네트워크를 통해 메시지를 보내고, 서버의 상태를 변경하며, 클라이언트의 상태에도 영향을 준다. 이러한 변경사항은 예측할 수 없으며, 같은 입력 값에 대해서도 외부 조건에 따라 다른 결과를 가져올 수 있다.
- 외부 시스템과의 상호작용을 포함한다.
서버 연결은 파일 시스템 접근, 네트워크 통신 등 외부 시스템과의 상호작용을 필요로 한다. 이러한 상호작용은 함수의 범위를 넘어서는 효과를 발생시킨다.

따라서 순수한 계산이 아닌 이 '서버에 연결하는 작업'을 렌더링하는 동안에 처리할 수 없다. 

#### 🤨 렌더링이 순수한 계산 과정이어야 하는 이유 복습!
- 재진입성 (Reentrancy)
렌더링 과정은 중단되거나 다시 시작될 수 있다.
side effect가 렌더링 과정 중에 발생한다면, 이런 중단과 재시작으로 인해 예측 불가능한 결과가 발생할 수 있겠지!
- 성능 최적화
순수한 렌더링을 통해 React는 불필요한 렌더링을 방지하고, 컴포넌트의 리렌더링을 효율적으로 계산할 수 있는데, 부작용이 포함되면 이러한 최적화가 어려워진다.

이러한 이유로 React는 렌더링과 side effect를 분리하고, useEffect 같은 훅을 제공해서 컴포넌트가 화면에 커밋된 이후에 side effect를 안전하게 처리할 수 있도록 설계되어 있다. 👍

따라서 채팅 서버에 연결하는 작업은 렌더링 과정에서 분리하여 useEffect 같은 훅을 통해 관리해야 한다.
이를 통해 컴포넌트의 렌더링이 예측 가능하고 순수한 계산으로 남아있게 하면서, 필요한 side effect는 안전하게 실행할 수 있다.
즉, useEffect는 컴포넌트가 렌더링된 후에 sideEffect를 실행해 렌더링 과정 자체는 순수함을 유지하도록 해준다.

### 3) Effect
- Effect를 사용하면 특정한 이벤트가 아닌 렌더링 자체에 의해 발생하는 부수 효과를 지정할 수 있다.
- 서버 연결을 설정하는 것은 컴포넌트가 나타나게 한 상호작용이 무엇이든 상관 없이 일어나야 하기 때문에 Effect로 분류된다.
- Effects는 화면이 업데이트된 후 커밋의 끝에 실행된다.
이 시점은 네트워크나 제 3자 라이브러리와 같은 외부 시스템과 React 컴포넌트를 동기화하기에 좋은 시간이다.

## 3-2. Effect가 필요하지 않을 수도 있다
- 당장 컴포넌트에 Effect를 추가하려고 서두르지 말 것!
- Effect는 일반적으로 React 코드에서 **벗어나서** 외부 시스템과 동기화하기 위해 사용된다.
예를 들면 브라우저 API, third-party 위젯, 네트워크 등

#### 무한루프 조심!
기본적으로 Effect는 매 랜더링 후에 실행된다.
따라서 아래와 같은 코드는 무한 루프에 빠지게 된다.

```jsx
const [count, setCount] = useState(0);
useEffect(() => {
  setCount(count + 1);
});
```

동작을 살펴보면, 
렌더링의 결과로 Effect가 실행됨 -> state를 설정해서 렌더링을 촉발함 -> 렌더링이 발생해 Effect가 실행됨 ->  state를 설정해서 렌더링을 촉발함 ...

Effect는 보통 컴포넌트를 외부 시스템과 동기화하는데 사용되어야 한다.
외부 시스템이 없고 다른 state를 기반으로 일부 state를 조정하기만 원한다면, Effect가 필요하지 않을 수도 있다.

## 3-3. Effect 작성 방법

### 1) Effect를 선언한다.

- useEffect를 import하고 컴포넌트의 최상위 레벨에서 호출한다.
- 컴포넌트가 렌더링될 때마다 화면을 업데이트하고 나서 useEffect 내부의 코드가 실행된다.

```js
import { useEffect } from 'react';

function MyComponent() {
  useEffect(() => {

  });
  return <div />;
}
```

외부 시스템과 동기화하는 예시를 살펴보자!
브라우저의 빌트인 태그인 `<video>`를 가지고 있는 `<VideoPlayer>` 컴포넌트가 있다.
이 컴포넌트에 isPlaying prop을 전달해서 재생/일지중지 여부를 제어할 것이다.

```jsx
function VideoPlayer({ src, isPlaying }) {
  // TODO: do something with isPlaying
  return <video src={src} />;
}
```

브라우저의 `<video>` 태그에는 isPlaying prop이 없다.
이를 제어하는 유일한 방법은 DOM 요소에서 play(), pause() 메서드를 호출하는 것이다.
따라서 isPlaying을 play(), pause()와 같은 함수 호출과 동기화시켜야 한다.

❌ 아래의 코드는 렌더링 중에 play(), pause() 함수를 호출하려 하고 있다.
이런 오류가 남 👇

![](https://velog.velcdn.com/images/e_juhee/post/d76f575c-6e17-4baf-b59f-bbf3cde84110/image.png)

```jsx
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  if (isPlaying) {
    ref.current.play();  // Calling these while rendering isn't allowed.
  } else {
    ref.current.pause(); // Also, this crashes.
  }

  return <video ref={ref} src={src} loop playsInline />;
}
```

React에서 렌더링은 DOM 수정과 같은 side effect를 포함하면 안 된다.
또한, 이 시점에는 DOM이 아직 존재하지 않는다. React는 JSX를 반환하기 전까지는 어떤 DOM을 생성할 지 모르기 때문이다.

따라서 useEffect로 감싸 렌더링 계산 밖으로 옮겨야 한다.

```jsx
import { useEffect, useRef } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}
```

### 2) 의존성을 명시한다.
- Effect는 매 렌더링 후에 실행되는 것이 default이다.
- 이것이 원치 않는 동작일 수 있고, 또는 불필요하게 속도가 느려질 수 있다. 외부 시스템과의 동기화가 항상 즉각적으로 필요한 것은 아닐테니..
- 따라서 대부분의 Effect는 매 렌더링 마다가 아닌 필요할 때만 다시 실행해야 한다.

의존성을 지정해서 이를 제어할 수 있다.

아래의 예시에서는 input에 값이 입력되면 state가 업데이트되어 리렌더링이 일어나고 Effect가 실행된다.
다시 말하자면, 값을 입력할 때마다 Effect 내부의 코드가 실행되고 있다!!

```jsx
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('Calling video.play()');
      ref.current.play();
    } else {
      console.log('Calling video.pause()');
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

이런 불필요한 재실행을 막으려면 useEffect 호출의 두 번째 인자로 의존성 배열을 지정하면 된다.

우선 빈 배열을 추가하면 이런 오류가 표시된다.

```jsx
  useEffect(() => {
    // ...
  }, []);
```

![](https://velog.velcdn.com/images/e_juhee/post/2b901be2-d860-494a-ac01-c53475e4737c/image.png)

Effect 내부의 코드가 `isPlaying`에 의존해서 수행할 작업을 결정하는데, 이 의존성이 명시적으로 선언되지 않아서 나는 오류이다.
의존성 배열에 `isPlaying`을 추가해보자!

```jsx
  useEffect(() => {
    if (isPlaying) { // It's used here...
      // ...
    } else {
      // ...
    }
  }, [isPlaying]); // ...so it must be declared here!
```

이제 `isPlaing`이 이전 렌더링 때의 값과 같으면 Effect를 실행하지 않는다.

이 의존성 배열에는 여러 개의 의존성을 포함시킬 수 있다.
여러 개인 경우, 모든 의존성의 값이 이전 렌더링 때와 정확히 동일한 경우에만 Effect 실행을 건너 뛴다.
(`Object.is` 비교로 의존성 값을 비교한다.)

지정한 의존성들이 Effect 내부의 코드를 기반으로, React가 예상하는 것과 일치하지 않으면 lint 오류가 발생한다.
오류에서 요구하는 의존성이 불필요하다면 React가 해당 의존성을 필요로 하지 않도록 Effect 내부의 코드를 수정하도록 하자.

Effect 코드 내부에서 사용되는 값들 중에서 안정적인 정체성을 가지고 있는 경우에만 의존성에서 생략해도 된다. (그렇지 않으면 위에서처럼 lint 에러남!)
어떤 게 안정적인 거지?
- ref 객체 
React는 렌더링할 때마다 동일한 useRef 호출로부터 항상 같은 객체를 반환할 것을 보장한다.
- useState 설정자 함수
(하지만 이 ref가 부모 컴포넌트로부터 prop으로 전달받은 것이라면,
항상 동일한 ref를 전달하는지, 여러 ref 중 하나를 조건부로 전달하는지 알 수 없기 때문에
이것도 안정적이지 않아서 의존성 배열에 추가해줘야 함)

#### `[]` 의존성 배열
의존성 배열이 없는 경우와 비어 있는 경우의 동작은 다르다!!!

```jsx
useEffect(() => {
  // 렌더시마다 실행
});

useEffect(() => {
  // 오직 마운트시에만 실행
}, []);
```

### 3) 필요한 경우 cleanup을 추가한다.
- 일부 Effect는 수행중이던 작업을 중지, 취소, 정리하는 방법을 명시해야 한다.
예를 들어 connect, subscribe, fetch 후에는 disconnect, unsubscribe, cnacel 또는 ignore을 해줘야 함.
- 클린업 함수는 Effect가 다시 실행되기 전에 매번 호출되고, 컴포넌트가 제거(unmount)될 때 마지막으로 한 번 더 호출된다.

## 3-4. Effect 언제 써요?

### 1) React가 아닌 위젯 제어하기
- React로 작성하지 않은 UI 위젯 추가
ex) 지도 컴포넌트를 React 코드의 zoomLevel state와 동기화하려면

```jsx
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

- 브라우저의 빌트인 `<dialog>`의 showModal 메서드 사용
이 메서드는 두 번 호출하면 에러를 던지므로 클린업 함수가 필요하다.

```jsx
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
```

### 2) 이벤트 구독
- Effect가 무언가를 구독한다면, 클린업 함수에서 구독을 취소해줘야 한다.

```jsx
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

### 3) 애니메이션 트리거링
- Effect가 무언가를 애니메이션화한다면, 클린업 함수에서 애니메이션을 초기 값으로 리셋해야 한다.

```jsx
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
                          // 애니메이션 촉발
  return () => {
    node.style.opacity = 0; // Reset to the initial value
                            // 초기값으로 재설정
  };
}, []);
```

- tweening을 지원하는 third-party 애니메이션 라이브러리를 사용하는 경우, 클린업 함수는 타임라인을 초기 상태로 리셋해야 한다.
(tweening: "in-betweening"의 줄임말. 프레임 사이의 중간 상태들을 생성하여 애니메이션을 부드럽게 만드는 기술)

### 4) 데이터 fetch
- Effect에서 무언가를 fetch하면, 클린업 함수에서 fetch를 중단하거나 그 결과를 무시해야 한다.

```jsx
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```

- 이미 발생한 네트워크 요청을 취소할 수는 없으므로, 대신 클린업 함수에서 더 이상 관련이 없는 fetch가 애플리케이션에 계속 영향을 미치지 않도록 하기 위함이다.
ex) 만약 userId가 'Alice'에서 'Bob'으로 변경되면 클린업은 'Alice' 응답이 'Bob' 이후에 도착하더라도 이를 무시하도록 한다.
