## useEffect의 의존성은 린터가 시키는 대로 해야 함. 
의존성 배열에는 Effecf에서 사용하는 모든 반응형(리렌더링에 따라 값이 바뀔 수 있는) 값이 들어가야 하고, 의존성을 제거하려면 린터에게 이게 반응형 값이 아님을 증명해야 함.

### 의존성 제거하는 법
1. 이벤트 핸들러로 옮겨야 할 코드가 Effect에 있는 지 살펴보기
2. Effect가 서로 관련 없는 여러 가지 일을 동시에 수행하고 있다면 각각의 Effect로 분할하기<br>-> 중복 로직을 줄이고 싶다면 custom hook으로 추출
3. 이전 state를 사용해서 setState 해야 할 때는 업데이터 함수 사용하기
   ```javascript
   setState([...state, newState])                  // X
   setState(prevState => [...prevState, newState]) // O
   ```
## 값의 변경에 반응하지 않고 값을 읽고 싶다면?
### [Effect Event](https://github.com/Jungle-JavaScript-Study/react-docs-study/blob/main/Escape%20Hatches/Separating%20Events%20from%20Effects-juhee.md#useeffectevent) 사용해서 Effect를 반응형 부분과 비반응형 부분으로 나누기
1. 이벤트 핸들러를 props로 받을 때
   ```javascript
   function ChatRoom({ roomId, onReceiveMessage }) {
    const [messages, setMessages] = useState([]);
  
    const onMessage = useEffectEvent(receivedMessage => {
      onReceiveMessage(receivedMessage);
    });
  
    useEffect(() => {
      const connection = createConnection();
      connection.connect();
      connection.on('message', (receivedMessage) => {
        onMessage(receivedMessage);
      });
      return () => connection.disconnect();
    }, [roomId]); // 의존성에서 이벤트 핸들러 제거 가능
   ```
2. 반응형 코드와 비반응형 코드 분리(Effect 내에서 반응형 값을 사용하고 싶지만 Effect를 촉발하고 싶지는 않을 때)

## 내가 생각한 것보다 더 의존성 값이 자주 변경된다면(객체, 함수가 의존성 배열에 들어있을 때)
### 정적 객체나 함수를 컴포넌트 외부로 이동
우리 눈에 보기에는 계속 같은 객체지만 자바스크립트는 내용물이 같은지 아닌지 판별할 수 없음<br>
-> 정적인 객체는 아예 컴포넌트 외부로 빼버리기

### Effect 내에서 동적 객체나 함수 이동
동적 객체나 함수라면 Effect 내에서 정의하기

### 객체에서 원시값 읽기
props로 객체를 받는다면 Effect 외부에서 객체를 분해해와서 사용하기
```javascript
<ChatRoom
  roomId={roomId}
  options={{
    serverUrl: serverUrl,
    roomId: roomId
  }}
/>

function WrongChatRoom({ options }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // X: 부모 컴포넌트가 리렌더링 할 때마다 Effect가 다시 연결됨
  // ...

function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // O: props 객체때문에 Effect가 실행되지 않음
```

### 함수에서 원시값 계산하기
```javascript
function ChatRoom({ getOptions }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = getOptions();
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // O: props로 받은 함수로 Effect가 실행되지 않음
  // ...

```
