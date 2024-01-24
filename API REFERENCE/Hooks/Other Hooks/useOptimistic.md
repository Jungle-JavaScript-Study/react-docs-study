useOptimistic은 지금은 Canary(가장 최신의 개발 버전)와 실험 채널에서만 사용이 가능하다.

# useOptimistic

useOptimistic은 UI를 낙관적으로 업데이트할 수 있는 훅이다.

# 1. Reference
- 비동기 작업이 진행되는 동안 다른 상태를 표시하는 데 사용하는 훅이다.
- 특정 state를 인자로 받아, 네트워크 요청 같은 비동기 작업이 진행되는 동안 해당 상태의 복사본을 반환한다. 이 복사본은 원래 상태와 다를 수 있다.
- 현재 상태와 작업을 입력 받아서 낙관적 상태를 반환하는 함수를 제공해야 한다.
- 이 상태를 "낙관적" 상태라고 부르는 이유는, 실제 작업이 완료되는 데 시간이 걸리더라도, 수행 결과를 즉시 보여주는 데 주로 사용되기 때문이다.

```js
import { useOptimistic } from 'react';

function AppContainer() {
  const [optimisticState, addOptimistic] = useOptimistic(
    state,
    // updateFn
    (currentState, optimisticValue) => {
      // merge and return new state
      // with optimistic value
    }
  );
}
```

## 1-1. Parameters
### state
- 초기에 반환되며, 어떠한 작업도 대기 중이지 않을 때 반환될 값이다.

### updateFn(currentState, optimisticValue)
- 현재 상태와 addOptimistic에 전달된 낙관적 값(optimisticValue)을 인자로 받아 낙관적 상태를 반환하는 함수
- 순수 함수이어야 한다.


## 1-2. Returns
### optimisticState
- 결과인 낙관적 상태
- 기본적으로, 작업이 대기중이 아닌 경우에는 state와 똑같다.
- 작업이 대기중인 경우에는 updateFn에 의해 반환된 값과 같다.

### addOptimistic
- 낙관적 업데이트를 실행할 때 호출할 dispatch 함수
- 하나의 인자인 optimisticValue(어떤 타입이든 가능)를 받는다.
- state와 optimisticValue를 사용해서 updateFn을 호출한다.


# 2. 예시

메시지를 전송하면 서버에 전송중일 때는 Sending을 표시하고, 서버에서 수신이 확인되면 Sending을 제거하는 예시이다.

```js
function Thread({ messages, sendMessage }) {
  const formRef = useRef();

  // 폼 제출 함수
  async function formAction(formData) {
    // state와 optimisticValue(인자)를 사용해서 updateFn을 호출한다.
    addOptimisticMessage(formData.get("message"));
    formRef.current.reset();
    await sendMessage(formData);
  }
  
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage) => [
      ...state,
      {
        text: newMessage,
        sending: true // 낙관적 value에서는 sending: true
      }
    ]
  );
  
  return (
    <>
      {optimisticMessages.map((message, index) => (
        <div key={index}>
          {message.text}
			// sending이 true인 경우에 노출
			// 서버에서 받은 응답이 아닌 낙관적 value인 경우에 해당
          {!!message.sending && <small> (Sending...)</small>}
        </div>
      ))}
      <form action={formAction} ref={formRef}>
        <input type="text" name="message" placeholder="Hello!" />
        <button type="submit">Send</button>
      </form>
    </>
  );
}
  

export default function App() {
  const [messages, setMessages] = useState([
    { text: "Hello there!", sending: false, key: 1 }
  ]);
  
  // 메시지 전송
  async function sendMessage(formData) {
    const sentMessage = await deliverMessage(formData.get("message"));
    // 전송한 메세지가 messages에 추가된다. sending은 false
    setMessages((messages) => [...messages, { text: sentMessage }]);
  }
  return <Thread messages={messages} sendMessage={sendMessage} />;
}

```
