# 5. State 로직을 Reducer로 추출하기

- 여러 개의 state 업데이트가 여러 이벤트 핸들러에 걸쳐 퍼져 있으면 관리하기가 어려워질 수 있다.
이런 경우, 컴포넌트 외부에서 모든 상태 업데이트 로직을 하나의 함수인 reducer에 집중시킬 수 있다.
- 즉, 많은 상태 업데이트와 이벤트 핸들러를 다루는 컴포넌트는 복잡해질 수 있으므로, 리듀서를 사용해서 상태 업데이트 로직을 단일 함수에서 관리하라는 것
리듀서는 상태 업데이트 로직을 모아서 관리하므로 코드를 더 깔끔하고 관리하기 쉽게 만들어 준다.
- 단일 함수란?
코드의 재사용성과 유지 보수성 향상을 위해서 특정 기능이나 연산을 수행하는 데 필요한 로직을 하나의 함수로 통합하여 구성하는 것

### reducer 의미
- 배열 메서드인 `reduce()`에서 따서 지은 이름이다.
- `reduce()`는 배열의 값들을 하나의 값으로 누적한다.
- reduce에 전달하는 함수를 reducer라고 부르는데, 이 reducer는 지금까지의 결과와 현재 항목을 받아 다음 결과를 반환한다.
- React의 reducer도 지금까지의 상태와 액션을 받아서 다음 상태를 반환한다.

## 5-1. reducer로 state 로직 통합하기
- 컴포넌트가 복잡해지면 state가 업데이트되는 다양한 경우를 한 눈에 파악하기 어려워진다.
setState를 어디서 하고 있는지 찾기 어렵다는 뜻
- 복잡성을 줄이고 모든 로직을 접근하기 쉽게 reducer에 모으자

### reducer에서 state로 마이그레이션하기

### 1) State 설정을 action들의 전달로 바꾸기
state를 재설정하던 이벤트 핸들러에서 setState 로직을 지우고
대신 `action`을 전달한다.

- `dispatch` 함수에 인자로 전달하는 객체를 `action`이라고 한다.
- `action`에는 문자열 타입의 `type`을 지정하고 추가적인 정보는 자유롭게 다른 필드로 전달하면 된다.

```js
function handleAddTask(text) {
  dispatch({
    type: 'added',
    id: nextId++,
    text: text,
  });
  
  기존
  // setTasks([
  //   ...tasks,
  //   {
  //     id: nextId++,
  //     text: text,
  //     done: false,
  //   },
  // ]);
}
```

### 2) reducer 함수 작성
이 reducer 함수 안에 state 로직을 작성한다.
- reducer는 state와 action 객체를 매개변수로 가진다.
- 이 함수에서 state를 반환하면 React가 여기서 반환된 것을 state로 설정해준다.
- state를 매개변수로 받기 때문에 컴포넌트 밖에서 선언해도 된다!
- 다른 파일로 분리도 가능

```js
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

### 3) reducer 사용

- useState를 지우고 useReducer를 사용한다.
- useReducer Hook은 `reducer 함수`와 `초기 state`를 인자로 받는다.
- `state 값`과 `dispatch 함수`를 반환한다.

```js
import { useReducer } from 'react';

// const [tasks, setTasks] = useState(initialTasks);
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

## 5-2. useState VS useReducer

### 1) 코드 크기
- `useState`를 쓰면 초기에 작성해야 할 코드가 덜 필요하다. 간단
- `useReducer`는 `reducer 함수`와 `dispatch action`을 작성해야 한다.
- 여러 이벤트 핸들러에 걸쳐 비슷한 상태 수정 로직이 있다면, 그 로직을 `reducer`에 centralize함으로써 코드를 더 깔끔하게 유지할 수 있다.
즉, `useReducer`는 상태 관리 로직이 복잡하고 여러 이벤트 핸들러에서 상태를 수정할 때 유용하게 사용할 수 있다.

### 2) 가독성
- 상태 업데이트가 간단하면 `useState`가 가독성이 좋다.
- 상태 업데이트 로직이 복잡해지면, `useReducer`로 업데이트 로직이 어떻게 동작하는지(how)와 어떤 일이 일어났는지(what happened)를 깔끔하게 분리할 수 있다.
즉, `useReducer`는 복잡한 상태 업데이트 로직에서 코드의 가독성을 향상시킨다.

### 3) 디버깅
- `useState`로 버그가 발생하면, 어디서 잘못 설정되었는지와 이유를 파악하기 어려울 수 있다. 상태 변경이 여러 곳에서 일어날 수 있고 어디서 발생했는지 명확하지 않을 수 있기 때문!
- `useReducer`를 쓰면 리듀서에 콘솔 로그를 추가하여 모든 상태 업데이트와 그 이유(action)을 명확하게 알 수 있다. 
- 만약 각 action이 올바르다면, 오류는 reducer 로직 자체에 있는 것.. 하지만 이럴 경우 `useState`보다 더 많은 코드를 검토해야 할 수 있다.
즉, `useReducer`는 상태 변경 로직의 버그를 찾고 디버깅하기 더 쉽게 만들어주는 장점이 있지만, `useState`에 비해 더 많은 코드를 살펴봐야 할 수도 있다.

### 4) 테스트
- reducer는 컴포넌트에 의존하지 않는 순수 함수이기 때문에 따로 분리해서 독립적으로 테스트할 수 있다.
- 현실적인 환경에서 테스트하는 것이 좋긴 하지만, 복잡한 상태 업데이트 로직의 경우 초기 상태와 액션에 대해 리듀서가 특정 상태를 반환하는지 확인하는 것이 유용할 수 있다.

### 5) 개인 취향
- 취향 차이임. `useState`와 `useReducer`는 언제든 서로 변환해서 사용할 수 있으며, 동등하다.

## 5-3. Reducer 잘 쓰기
### 1) Reducer는 순수해야 한다.
- reducer는 렌더링 도중에 실행되므로 순수해야 한다. 즉, 입력이 같으면 항상 동일한 출력을 생성해야 한다. 외부 세계에 영향을 주는 어떠한 작업도 수행해서는 안 된다.

#### reducer 안에서 하면 안 되는 것
- 네트워크 요청을 보내면 안 됨
- timeout 스케줄링하면 안 됨
- 객체 및 배열을 직접 수정하는 등의 작업 하면 안 됨

### 2) 각 action이 하나의 사용자 상호작용을 설명해야 한다.
- 여러 state에 대한 변경이 이루어지더라도 하나의 사용자 상호작용 당 하나의 action으로 관리하는 것이 합리적이다.

## 5-4. Immer 활용
- `useImmerReducer`를 사용하면 `push` 또는 `arr[i] = ` 방식으로 state를 변이할 수 있다.
- Immer는 변경해도 안전한 특별한 `draft` 객체를 제공한다. 이 `draft` 객체에 변화를 적용하면, Immer는 내부적으로 상태의 복사본을 생성하고 `draft`에 만든 변경 사항을 적용한다.
- state를 반환할 필요가 없다.
