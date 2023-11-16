## Effect의 생명주기

모든 리액트 컴포넌트는 동일한 생명주기를 거친다.

화면에 추가될 때 마운트되고, 새로운 props나 state를 받으면 업데이트되고(보통 상호작용에 의해), 화면에서 제거되면 마운트 해제된다.

Effect의 생명주기는 이것과 분리해서 생각해야 한다.
Effect는 외부 시스템을 현재 props와 state에 어떻게 동기화 하는지를 설명한다.

Effect 코드 바꾸는 행위 -> 동기화 빈도 수에 영향을 끼치는 것

<br />
채팅 서버에 연결하는 예시

```javascript
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

동기화 시작 방법 명시

```
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ...
```

반환되는 클린업 함수는 동기화 중지 방법 지정

```
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ...
```

<br />


때로는 동기화 여러 번 시작하고 중지해야 할 수도 있다.

<br />


## 동기화 두 번 이상 수행돼야 하는 이유

roomId 바뀔 때 동기화 중지 및 시작

## 리액트가 Effect를 재동기화 하는 방법

컴포넌트가 다른 roomId로 다시 렌더링할 때마다 Effect가 다시 동기화됨

다른 화면 가면 마운트 해제 및 동기화 중지 -> 연결 끊음

## Effect 관점에서 생각하기

시작 중지 시작 중지 시작 중지

## 리액트가 Effect의 재동기화 가능 여부를 확인하는 방법

개발 모드에서 React는 즉시 강제로 동기화를 수행하여 Effect가 다시 동기화될 수 있는지 확인함.

도어락 작동하는지 확인하려고 문 열었다 한 번 더 닫는 것과 비슷

## React가 Effect의 재동기화 필요성을 인식하는 방법

의존성 배열에 roomId 포함시켜서 리액트가 알게 함.
의존성 배열 값 하나라도 이전 렌더링 중 전달한 값과 다르면 리액트는 Effect를 다시 동기화함.

## 각각의 Effect는 별도의 동기화 프로세스를 나타냅니다. 

코드의 각 Effect는 별도의 독립적인 동기화 프로세스를 나타내야 함.
프로세스가 동일한지 또는 분리되어 있는지를 고려

## Effect는 ‘반응형 값’에 반응합니다 

의존성은 시간이 지남에 따라 변경될 때만 무언가를 수행함
반응형 값은 의존성에 포함.

## 빈 의존성을 가지고 있는 Effect의 의미 

빈 의존성 배열은 이 Effect가 컴포넌트가 마운트될 때만 채팅방에 연결되고 컴포넌트가 마운트 해제될 때만 연결이 끊어진다는 것을 의미

Effect의 관점에서 생각하면 마운트 및 마운트 해제에 대해 전혀 생각할 필요가 없음. 중요한 것은 Effect가 동기화를 시작하고 중지하는 작업을 지정한 것.

## 컴포넌트 본문에서 선언된 모든 변수는 반응형입니다 

다시 렌더링할 때 변경될 수 있는 건 의존성에 넣자.

<br />

# ? 전역 또는 변이 가능한 값이 의존성이 될 수 있나요 ?

변이 가능한 값(전역 변수 포함)은 반응하지 않습니다.
location.pathname과 같은 변이 가능한 값은 의존성이 될 수 없음.

ref.current와 같이 변이 가능한 값 또는 이 값으로부터 읽은 것 역시 의존성이 될 수 없습니다. useRef가 반환하는 ref 객체 자체는 의존성이 될 수 있지만, current 프로퍼티는 의도적으로 변이 가능.

하지만 이를 변경하더라도 리렌더링을 촉발하지는 않기 때문에, 이는 반응형 값이 아님.


<br />

## React는 모든 반응형 값을 의존성으로 지정했는지 검토합니다 

반응형 값 의존성 배열 포함 안하면 린트 에러남.


## 재동기화를 원치 않는 경우엔 어떻게 해야 하나요? 

의존성을 “선택”할 수는 없음.
의존성에는 Effect에서 읽은 모든 반응형 값이 포함되어야 함. 
린터가 이를 강제함. 
때때로 이로 인해 무한 루프와 같은 문제가 발생하거나 Effect가 너무 자주 다시 동기화될 수 있음.

대신 시도할 수 있는 방법

- Effect가 독립적인 동기화 프로세스를 나타내는지 확인.
  Effect가 아무것도 동기화하지 않는다면 불필요한 것일 수 있음.

- Effect를 반응하는 부분(Effect에 유지)과 반응하지 않는 부분(Effect Event라는 것으로 추출)으로 분리

- 객체와 함수를 의존성으로 사용하지 말자. 렌더링 중에 오브젝트와 함수를 생성한 다음 Effect에서 읽으면 렌더링할 때마다 오브젝트와 함수가 달라짐.