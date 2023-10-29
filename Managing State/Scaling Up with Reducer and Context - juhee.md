# 7. Reducer와 Context로 확장하기

Reducer를 사용하면 state 업데이트 로직을 통합할 수 있다.
Context를 사용하면 다른 컴포넌트들에 정보를 전달할 수 있다.
👉 Reducer와 Context를 함게 사용해서 복잡한 state를 관리할 수 있다.

## 7-1. reducer와 context 결합하기

state와 dispatch 함수를 props 전달이 아닌 context에 넣어서 사용해보자!

### 1) Context를 생성한다.
state와 dispatch를 위한 context를 각각 만든다.

```js
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

### 2) State와 dispatch 함수를 context에 넣는다.

컴포넌트에서 1에서 만든 context를 가져와서 value를 넣어주고,
하위 트리에 전달한다. (태그들을 감싸라는 뜻!)

```jsx
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        ...
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

### 3) Context를 사용한다.

필요한 컴포넌트에서 context를 가져와서 사용한다.

state 가져오기

```js
export default function TaskList() {
  const tasks = useContext(TasksContext);
  // ...
```

dispatch 가져오기

```js
export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);
  // ...
  return (
    // ...
    <button onClick={() => {
      setText('');
      dispatch({
        type: 'added',
        id: nextId++,
        text: text,
      });
    }}>Add</button>
    // ...
```

## 7-2. 하나의 파일로 합치기

필수는 아니지만,
reducer와 context를 모두 하나의 파일에 작성해서 컴포넌트를 더 정리할 수 있다.

createContext로 context를 만들어 둔 파일에 reducer를 옮기고
컴포넌트를 새로 선언하자!
예시에서는 컴포넌트 이름을 `TasksProvider`로 지었다.

prop으로 children을 받게 해서 JSX를 전달할 수 있게 만들었다.

context를 사용하기 위한 함수들도 커스텀 훅으로 만들어놔서 밖에서 사용할 수 있게 한다.

```jsx
import { createContext, useContext, useReducer } from 'react';

const TasksContext = createContext(null);

const TasksDispatchContext = createContext(null);

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: 'Philosopher’s Path', done: true },
  { id: 1, text: 'Visit the temple', done: false },
  { id: 2, text: 'Drink matcha', done: false }
];
```

이렇게 분리를 하면 좋은 점
👉 컴포넌트가 데이터를 어디서 가져오는지보다, 무엇을 표시하는지에 집중하게 한다.

