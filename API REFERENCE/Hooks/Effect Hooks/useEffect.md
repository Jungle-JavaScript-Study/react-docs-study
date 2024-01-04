# `useEffect(setup, dependencies?)`
: 컴포넌트를 **외부 시스템과 동기화**할 수 있는 훅

- `setup`: Effect의 로직이 포함된 함수. 클린업함수를 리턴할 수 있음
- **컴포넌트가 DOM에 추가되면(마운트될 때) 셋업 함수 실행-의존성 변경으로 리렌더링 시 이전 값으로 클린업 함수 실행-새 값으로 셋업 함수 실행-컴포넌트가 DOM에서 제거될 때 클린업 함수 실행**
- `dependencies`: 셋업 함수에서 참조된 모든 반응형 값의 목록(props, state, 변수, 함수 모두 포함). `Object.is`로 이전 값과 비교. 의존성 배열이 비어있으면 컴포넌트가 리렌더링될 때마다 Effect가 실행
- 훅이기 때문에 컴포넌트의 **최상위 레벨** 또는 커스텀 훅에서만 호출할 수 있음(반복문/조건문 내 불가)
- 반복문이나 조건문 내에서 사용하고 싶다면 새로운 컴포넌트를 추출하고 state를 그 안으로 옮겨서 사용
- 외부 시스템과 동기화하려는 목적이 아니라면 Effect를 사용하지 않아도 됨
- **Strict 모드에서는 실제 셋업함수 실행 전에 셋업+클린업 함수를 한번 더 실행**함. 이 때 문제가 발생한다면 클린업 함수 구현이 필요한 것
- 컴포넌트 내에서 정의된 객체/함수가 의존성 배열에 들어가면 Effect가 필요 이상으로 자주 실행될 수 있음 → 불필요한 의존성 제거, 업데이터 함수 사용, 비반응형 로직을 Effect 밖으로 추출하는 등의 방법으로 해결
- Effect가 사용자와의 상호작용으로 촉발된게 아니면 Effect 실행 전 브라우저의 화면 페인팅이 먼저 발생함. 만약 Effect가 시각적인 작업 중이고 딜레이가 눈에 띌 정도라면 대신 `useLayoutEffect` 사용
- 상호작용으로 인한 Effect의 경우에도 Effect 내부의 state가 업데이트 되기 전에 리페인팅이 먼저 발생할 수 있음. 이걸 막고 싶다면 `useLayoutEffect` 사용.
- Effect는 클라이언트에서만 실행됨(서버 렌더링 중에는 실행되지 않음)

## 외부 시스템과 연결하기
- 네트워크, 브라우저 API(타이머, 브라우저 이벤트, DOM, intersectionObserver, ...), 써드파티 라이브러리 등은 리액트 외부에서 제어되는 것이므로 외부 시스템이라고 할 수 있음
- 컴포넌트가 페이지에 표시되는 동안 외부 시스템과의 연결을 유지해야 할 때 `useEffect`를 사용

  ```javascript
  import { useEffect } from 'react';
  import { createConnection } from './chat.js';
  
  function ChatRoom({ roomId }) {
    const [serverUrl, setServerUrl] = useState('https://localhost:1234');
  
    useEffect(() => {
    	const connection = createConnection(serverUrl, roomId);
      connection.connect();
    	return () => {
        connection.disconnect();
    	};
    }, [serverUrl, roomId]);
    // ...
  }
  ```
- Strict 모드에서의 Effect 중복 실행은 클린업 함수가 셋업 함수의 작업을 제대로 취소하는지 알아보기 위한 과정이므로 production 모드에서 셋업이 한번 호출되는 것과 dev 모드에서 셋업-클린업-셋업이 호출되는 것 간에 차이가 없어야 함
- 클린업 로직이 셋업 로직을 올바르게 미러링하고 있다면 셋업과 클린업이 자주 실행되더라도 문제되지 않음

## 커스텀 훅으로 Effect 감싸기
Effect는 리액트에서 벗어나야 할 때만 사용하는 것인데, 수동으로 작성해야 하는 경우가 자주 발생한다면 커스텀 훅으로 추출해야 함

```javascript
// useChatRoom.js
function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}

// ChatRoom.jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  // ...
```

## 리액트가 아닌 위젯 제어하기
외부 시스템을 props나 state와 동기화하고 싶을 때 Effect를 사용

```javascript
import { useRef, useEffect } from 'react';
import { MapWidget } from './map-widget.js';

export default function Map({ zoomLevel }) {
  const containerRef = useRef(null);
  const mapRef = useRef(null);

  useEffect(() => {
    if (mapRef.current === null) {
      mapRef.current = new MapWidget(containerRef.current);
    }

    const map = mapRef.current;
    map.setZoom(zoomLevel);
  }, [zoomLevel]);
  // ...
}
```
-> 여기서는 `MapWidget` 클래스가 전달받은 유일한 DOM 노드를 관리하기 때문에 클린업 함수가 필요하지 않음. `Map` 컴포넌트가 트리에서 제거된 후 `MapWidget` 클래스와 DOM 노드 모두 자동으로 가비지컬렉팅 됨

## Effect로 데이터 가져오기
- Effect를 사용해 데이터를 가져올 수 있지만, 프레임워크를 사용하는 경우 프레임워크에 내장된 페치 메커니즘을 사용하는 것이 훨씬 효율적임
- Effect에서 데이터를 페칭하면 반복적이게 되고 나중에 최적화를 추가하기 어려워짐. 커스텀훅 사용을 권장함
- 네트워크 응답이 보낸 순서와 다른 순서로 도착했을 때의 race condition을 방지하기 위해 클린업 함수 필요

  ```javascript
  function Page() {
    const [person, setPerson] = useState('Alice');
    const [bio, setBio] = useState(null);
    useEffect(() => {
      async function startFetching() {
        setBio(null);
        const result = await fetchBio(person);
        if (!ignore) {
          setBio(result);
        }
      }
  
      let ignore = false;
      startFetching();
      return () => {
        ignore = true;
      }
    }, [person]);
  // ...
  ```
- Effect에서 fetch하는 방식의 단점:
  1. 초기 렌더링이 느려짐 - Effect가 서버측에서는 실행되지 않기 때문에 초기의 데이터 없는 state로 앱을 렌더링하고 나서야 데이터 로드가 시작됨.
  2. 네트워크 워터폴 발생 - 상위 컴포넌트 렌더링 후에 상위 컴포넌트의 데이터를 페치하고, 하위 컴포넌트를 렌더링하고, 데이터를 페치하기 시작하기때문에 순차적으로 처리됨. 데이터를 병렬로 페치하는 것보다 훨씬 느림.
  3. 데이터를 미리 로드하거나 캐시할 수 없음 - 컴포넌트가 언마운트됐다가 다시 마운트되면 데이터를 다시 페치해야 함
  4. race condition이 발생하지 않도록 하려면 보일러플레이트 코드가 많아짐
=> 프레임워크를 사용하는 경우 빌트인 데이터 페칭 메커니즘을 사용하고, 프레임워크가 없으면 클라이언트측 캐시를 사용해서 해결 가능.(React Query, useSWR, React Router 6.4+ 등)
=> 이도 저도 마음에 안들면 그냥 Effect에서 데이터 페칭해도 ㄱㅊ

## 반응형 의존성 지정
- Effect의 의존성을 선택적으로 추가할 수 없음(Effect 내에서 사용되는 모든 반응형 값은 의존성으로 선언되어야 함)
- props, state, 컴포넌트 내에서 선언됨 변수와 함수 모두 반응형 값
- 의존성을 제거하려면 의존성이 아니라는 것을 린턴에게 증명해야 함(ex. 변수를 컴포넌트 외부로 이동)
- 의존성 배열에 따른 Effect의 실행:
  1. 의존성이 있을 때 - 초기 렌더링 후와 의존성이 변경되어 리렌더링된 후에 실행
  2. 의존성 배열이 비어있을 때 - 컴포넌트의 props나 state가 변경되어도 다시 실행되지 않음(초기 렌더링 직후에만 한번 실행)
  3. 의존성 배열 자체를 전달하지 않을 때 - 컴포넌트의 모든 렌더링 후마다 실행

## Effect에서 이전 state를 기반으로 새로운 state 업데이트 하기
- Effect 내에서 `setState(state +1)`과 같이 이전 값을 기반으로 새로운 state를 업데이트 한다면 무한렌더링이 발생할 수 있음(Effect 실행-state 업데이트-업데이트된 state가 의존성이므로 Effect 다시 실행-무한 반복)
- 따라서 업데이터 함수를 사용하고 state를 의존성 배열에서 제거: `setState(prev => prev +1)`

## 불필요한 객체/함수 의존성 제거하기
렌더링 중에 생성된 객체/함수가 의존성 배열에 포함되는 경우 매번 다른 값으로 취급되어 매번 Effect가 실행됨(렌더링할때마다 함수가 생성되는 것 자체는 문제없음!)
<br>-> Effect 내에서 객체/함수를 생성(선언)하기 

## Effect에서 최신 props/state 읽기
Effect의 의존성 배열에 추가하지 않고 Effect 내에서 최신 props와 state를 읽고 싶다면 `useEffectEvent` 사용(아직 안정된 버전의 API는 아님)

```javascript
function Page({ url, shoppingCart }) {
  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, shoppingCart.length)
  });

  useEffect(() => {
    onVisit(url);
  }, [url]);
  // ...
}
```
-> `shoppingCart`가 변경되는 경우에는 Effect가 실행되지 않고 `url`이 변경될 때만 실행되므로 Effect를 촉발하지 않고 `shoppingCart`를 읽을 수 있음

## 서버와 클라이언트에 서로 다른 콘텐츠 표시하기
서버사이드 렌더링을 사용할때 클라이언트와 서버에서 다른 콘텐츠를 표시해야 하는 경우(ex. localStorage는 서버에서 접근할 수 없음)에는 서로 다른 콘텐츠를 표시해야 함

```javascript
function MyComponent() {
  const [didMount, setDidMount] = useState(false);

  useEffect(() => {
    setDidMount(true);
  }, []);

  if (didMount) {
    // ... return client-only JSX ...
  }  else {
    // ... return initial JSX ...
  }
}
```
-> 앱이 로드되는 동안에는 초기 렌더링 결과를 보여주고 앱이 로드된 이후 Effect가 실행되면서 `didMount`를 `true`로 설정해 리렌더링 촉발
=> 그러나 네트워크가 느린 경우에는 보던 콘텐츠가 갑자기 바뀌게 되므로 사용성에서 좋지 않음. 웬만하면 조건부 CSS로 해결하기
