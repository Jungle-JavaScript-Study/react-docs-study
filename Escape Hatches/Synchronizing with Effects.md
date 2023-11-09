# Effect와 동기화하기

### 이펙트가 필요한 순간
- 사이드 이펙트: 함수가 결과값 이외에 다른 상태를 변경하는 것. 전역변수 수정, 인자로 받은 변수 변경, 화면이나 파일에 데이터 쓰기 등.
- 리액트 컴포넌트 내부 로직에는 두가지 유형이 있음
  1. 렌더링 코드 - 컴퓨넌트의 최상위 레벨에서 props와 state를 조작하고 화면에 표시한 JSX를 리턴, 순수해야 함(사이드 이펙트 X)
  2. 이벤트 핸들러 - 사용자 작업으로 인해 사이드 이펙트 발생 가능
     
  => 사이드 이펙트라서 렌더링 중에 발생할 수가 없는데 촉발할만한 이벤트가 없다면 Effect에서 다룰 수 있음.<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;즉, Effect는 렌더링 자체로 발생하는 사이드 이펙트를 화면 업데이트 후 커밋이 끝날 때 실행하도록 하는 것.<br>
  &nbsp;&nbsp;&nbsp;※ Effect는 React에서 벗어나 외부 시스템과 동기화할때만 사용할 것(브라우저 API, 서드파티 위젯, 네트워크 등). state 변경만 할때는 X

### Effect 사용법
1. Effect 선언
    - Effect는 매 렌더링 후에 실행됨
       
    - 컴포넌트가 렌더링될 때마다 해당 렌더링이 화면에 반영될때까지 기다린 후 `useEffect` 내부 코드 실행
     
   -> DOM노드에 접근하는 코드를 컴포넌트 내에서 그냥 작성하면 아직 DOM노드가 존재하지 않을 때 접근하기때문에 에러 발생. 가능하다고 해도 렌더링은 순수한 JSX 계산이어야 함(DOM 수정은 사이드 이펙트)<br>
   => Effect는 렌더링이 완료된 후 실행되고, state 변경은 렌더링을 촉발하므로 Effect에서 state를 변경할 경우 무한루프에 빠지게 됨
   
3. 의존성 명시
   - Effect를 모든 렌더링마다가 아니라 필요할때만 실행하려면 의존성을 넣어야 함(키보드 입력마다 외부 서버에 연결하거나 에니메이션을 다시 실행하지 않으려면)
     
   - `useEffect` 호출의 두번째 인자로 의존성 배열을 추가하면 해당 의존성이 달라질 때만 Effect를 다시 실행<br>
     -> Effect 내부의 코드가 변수에 의해 수행할 작업이 결정된다면 해당 변수를 의존성 배열에 추가해야 함
     
   - 빈 배열일때는 컴포넌트 마운트 시에만 실행(의존성 배열이 없는 것과는 다름)
     
   - 의존성 배열의 모든 값이 이전 렌더링 때와 같을 때만 Effect를 재실행하지 않음(일부만 선택 불가, `Object.is`로 비교)
     
   - `useRef`에서 얻은 `ref`나 `useState`에서 얻은 `set`함수는 항상 동일한 객체를 얻는다는 것이 보장되므로 의존성 배열에 추가하지 않아도 됨(`useRef`호출로 리턴받은게 아니라 부모에게서 props로 전달받은 `ref`는 동일하지 않을 위험이 있으므로 추가되는게 맞음) 
5. 클린업 추가
   - 콘솔이 두번씩 찍히는 현상은 Strict Mode에서 리액트가 모든 컴포넌트를 첫 마운트 직후에 한번 더 마운트하기 때문<br>
     -> 버그를 발견하기 위함이므로 끄지 않는게 좋음, Effect를 한번 실행하는 것과 실행-정리-다시 실행하는 것의 동작이 차이가 없어야 함.<br>
     ex.) 마운트 시에 채팅 서버에 연결하는 Effect에 disconnect 과정이 없으면 페이지 이동 시마다 연결이 쌓이게 됨
     
   - 클린업 함수는 Effect가 실행중인 동작을 중지할때 사용(connect-disconnect, subscribe-unsubscribe, fetch-cancle 등)
  
## Effect 사용 패턴
### React가 아닌 위젯 제어
- 상태값과 위젯을 동기화하려고 할때<br>
  ex.)
  ```javascript
  useEffect(() => {
    const map = mapRef.current;
    map.setZoomLevel(zoomLevel);
  }, [zoomLevel]);
  ```
  -> Effect를 같은 값으로 두번 호출해도 상관 없으므로 클린업이 필요하지 않음
  
- 연속으로 두번 호출할 수 없는 API 같은 경우에는 클린업 필요<br>
  ex.)
  ```javascript
  useEffect(() => {
    const dialog = dialogRef.current;
    dialog.showModal();
  
    return () => dialog.close();
  }, []);
  ```

### 이벤트 구독
Effect가 이벤트를 구독하면 클린업 함수에서 구독을 취소해야 한번에 하나의 구독만 활성화 됨<br>
ex.)
```javascript
useEffect(() => {
  const handleScroll = () => {...};
  window.addEventListener('scroll', handleScroll);

  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

### 애니메이션 촉발
Effect가 애니메이션을 촉발하는 경우 클린업에서 초기값으로 재설정해야 함<br>
ex.)
```javascript
useEffect(() => {
  ref.current.style.opacity = 1;

  return () => {
    ref.current.style.opacity = 0;
  };
}, []);
```

### 데이터 페칭
Effect에서 데이터를 페치하면 클린업에서 페치를 중단하거나 무시해야 레이스 컨디션이 발생하지 않음<br>
ex.)
```javascript
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
-> 네트워크 요청을 취소할수는 없으므로 무시해야 userId가 바뀌었을 때 첫번째 요청이 나중에 도착해도 무시됨<br>
=> 요청이 두번 가는게 싫으면 컴포넌트 간에 응답을 캐싱하는 솔루션을 사용하기

사실 Effect에서 `fetch`하는건 클라이언트 사이드 앱에서만 인기있는 방법임. 리액트가 아니어도 마운트 시에 데이터를 페치하는건 단점이 많음.
 - 초기 렌더링이 느려짐 - Effect가 서버측에서는 실행되지 않기 때문에 초기의 데이터 없는 state로 앱을 렌더링하고 나서야 데이터 로드가 시작됨.
 - 네트워크 워터폴 발생 - 상위 컴포넌트 렌더링 후에 상위 컴포넌트의 데이터를 페치하고, 하위 컴포넌트를 렌더링하고, 데이터를 페치하기 시작하기때문에 순차적으로 처리됨. 데이터를 병렬로 페치하는 것보다 훨씬 느림.
 - 데이터를 미리 로드하거나 캐시할 수 없음 - 컴포넌트가 언마운트됐다가 다시 마운트되면 데이터를 다시 페치해야 함.
 - 레이스컨디션이 발생할 수 있음

=> 프레임워크를 사용하는 경우 빌트인 데이터 페칭 메커니즘을 사용하고, 프레임워크가 없으면 클라이언트측 캐시를 사용해서 해결 가능.(React Query, useSWR, React Router 6.4+ 등)<br>
=> 이도 저도 다 안되면 그냥 Effect에서 데이터 페칭해도 되긴 함.....

### 분석 보내기
- 페이지 방문 시 분석 정보를 보내는 코드는 개발 모드에서 분석이 두번씩 전송되지만 그대로 두는게 좋음. 어차피 프로덕션 모드에서는 마운트-언마운트-마운트 하지 않기 때문에.<br>
- 로그를 디버깅하고 싶다면 앱을 스테이징 환경에 배포하거나 Strict 모드를 해제할 수 있음
- Effect 대신 루트 변경 이벤트 핸들러에서 로그 전송하는 것도 방법
- 정확하게 하려면 intersection observer 사용해서 뷰포트에 어떤 컴포넌트가 표시되는지를 추적

### 어플리케이션 초기화는 Effect가 아니야!
애플리케이션이 시작될 때 한번만 실행되어야 하는 로직은 Effect가 아니라 컴포넌트 외부에 넣으면 됨(Effect는 최소한으로 사용하기)<br>
ex.)
```javascript
if (typeof window !== 'undefined') { // 실행 환경이 브라우저인지 확인
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {...}
```

### 제품 구매도 Effect가 아니야!
- Effect에서 제품 구매 POST 요청을 보내면 클린업 함수로도 두번 실행되는걸 막을 수 없음.
- 사용자가 다른 페이지로 갔다가 뒤로가기 버튼을 누르면 Effect가 다시 실행되면서 구매가 진행되는 문제 발생
- 구매는 렌더링으로 발생하는 사이드 이펙트가 아니기때문에 로직을 구매 버튼의 이벤트핸들러로 옮겨야 함.
  
=> 컴포넌트를 두번 마운트하는 것이 앱의 로직을 깨트린다면 버그인거임!!! 사용자 관점에서는 `처음 페이지 방문`-`뒤로가기 버튼으로 페이지 방문`이 똑같이 동작해야 함.

### 추가 주의 사항
- 리액트는 항상 다음 렌더링의 Effect 전에 이전 렌더링의 Effect를 정리함 -> `setTimeout`을 클린업에서 취소하면 항상 하나의 타임아웃만 예약되게 됨
- 각각의 Effect는 해당 렌더링의 상태값을 캡쳐함. 다음 렌더링에서 상태값이 바뀌더라도 해당 Effect는 캡쳐값으로 시행되기 때문에 영향을 받지 않음 => 클로저 개념
