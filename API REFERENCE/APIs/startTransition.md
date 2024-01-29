# `startTransition(scope)`
: UI를 차단하지 않고 상태값 업데이트

- 상태값 업데이트를 트랜지션으로 표시할 수 있다
- 아무것도 리턴하지 않음
- 트랜지션이 pending중인지 추적할 수 없음<br>⇒ 트랜지션 진행 중일 때 pending 표시를 보여주고 싶다면 `useTransition` 사용
  - `isPending`이 제공되지 않는걸 빼면 `useTransition`과 아주 유사하다<br>
  → `useTransition`을 사용할 수 없을 때 `starttransition` 사용(ex. 컴포넌트 외부에서 사용할 때)
- state의 `set` 함수에 접근 권한을 가지고 있을 때만 업데이트를 트랜지션으로 감쌀 수 있음<br>⇒ props나 커스텀 훅의 리턴값에 대응해서 트랜지션을 사용하고 싶다면 `useDeferredValue` 사용
- 트랜지션으로 표시된 상태 업데이트는 다른 상태 업데이트에 의해 중단됨<br>
  ex. 트랜지션 내에서 차트 업데이트를 하고 나서 차트가 리렌더링 되고 있는 중에 input에 입력을 시작하면 input 상태 업데이트가 처리되고나서 차트 컴포넌트의 렌더링 작업을 다시 시작함

### scope
- 하나 이상의 `set` 함수를 호출해서 state를 업데이트하는 함수
- 반드시 동기식으로 동작해야야 함
- 즉시 호출되고, 실행 중에 예약된 모든 상태 업데이트는 트랜지션으로 표시됨
- 이 업데이트들은 non-blocking이기 때문에 원치 않는 로딩바를 보여주지 않음
- timeout 같은 걸로 추후에 상태값 업데이트를 하려고 하면 트랜지션으로 표시되지 않음

## 상태 업데이트를 논블로킹 트랜지션으로 표시하기
```javascript
import { startTransition } from 'react';

function TabContainer() {
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
}
```
- 트랜지션을 사용하면 느린 기기에서나 리렌더링 중에도 UI 업데이트가 반응형이도록 할 수 있다<br>
  ex. 유저가 탭을 클릭했다 바로 다른 탭을 클릭하면 첫번째 리렌더링이 끝나는걸 기다리지 않고 다른 탭으로 전환할 수 있음
