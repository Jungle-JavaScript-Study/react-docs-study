****useReducer****
  - `useReducer` 는 컴포넌트에 reducer 를 추가할 수 있는 React Hook 이다.
      
      ```jsx
      const [state, dispatch] = useReducer(reducer, initialArg, init?)
      ```
      
  - Reference
      - **useReducer(reducer, initialArg, init?)**
          
          ```jsx
          import { useReducer } from 'react';
          
          function reducer(state, action) {
            // ...
          }
          
          function MyComponent() {
            const [state, dispatch] = useReducer(reducer, { age: 42 });
            // ...
          ```
          
          - **Parameters**
              - **reducer** : state 가 업데이트되는 방식을 지정하는 reducer 함수 ( 순수함수, state와 action을 인자로 받아야하고, 다음 state 를 반환해야 한다. )
              - **initialArg** : 초기 state 가 계산되는 값. 모든 유형일 수 있다. 초기 state 를 계산하는 방법은 `init` 인자에 따라 달라진다.
              - optional **init** : 초기 state 계산 방법을 지정하는 초기화 함수. **지정하지 않으면, initialArg 로 설정됨**. **지정하면 init(initialArg) 를 호출한 결과로 설정됨**.
          - **Returns**
              - `useReducer` 는 정확히 두 개의 값을 가진 배열을 반환한다.
                  1. 현재의 state ( `init(initialArg)` or `initalArg` )
                  2. state 를 다른 값으로 업데이트하고 리렌더링을 촉발할 수 있는 `dispatch` **함수**
          - **Caveats**
              - useReducer 는 React Hook 이다. ( 컴포넌트의 최상위 레벨 or 커스텀 훅에서만 호출 가능 )
      - **dispatch function**
          
          ```jsx
          const [state, dispatch] = useReducer(reducer, { age: 42 });
          
          function handleClick() {
            dispatch({ type: 'incremented_age' });
            // ...
          ```
          
          - **Parameters**
              - **action** : 사용자가 수행한 작업. 어떤 데이터유형이든 올 수 있다. **관용적으로 action 은 보통 이를 식별하는 type 속성이 있는 객체**이며, 선택적으로 추가 정보가 있는 다른 속성을 포함할 수 있다.
          - **Returns**
              - void , `dispatch` 함수에는 반환값이 없습니다.
          - **Caveats**
              - `dispatch` 함수는 **다음 렌더링에 대한 state 변수만 업데이트 한다.**
              - 만약 여러분이 제공한 새 값이 [Object.is](http://Object.is) 로 비교했을 때 현재 state 와 동일하다면, **React 는 컴포넌트와 그 자식들을 다시 렌더링하는 것을 건너뛴다.** ( React의 최적화 )
              - React 는 State 업데이트를 일괄처리 한다. **모든 이벤트 핸들러가 실행되고** set 함수를 호출한 후에 화면을 업데이트 한다. ( 드물지만 React가 화면을 더 일찍 업데이트하도록 강제해야하는 경우 `flushSync` 를 사용할 수 있다. )
  - Usage
      - 컴포넌트에 reducer 추가하기
          
          ```jsx
          import { useReducer } from 'react';
          
          function reducer(state, action) {
            if (action.type === 'incremented_age') {
              return {
                age: state.age + 1
              };
            }
            throw Error('Unknown action.');
          }
          
          export default function Counter() {
            const [state, dispatch] = useReducer(reducer, { age: 42 });
          
            return (
              <>
                <button onClick={() => {
                  dispatch({ type: 'incremented_age' })
                }}>
                  Increment age
                </button>
                <p>Hello! You are {state.age}.</p>
              </>
            );
          }
          ```
          
          - `useReducer` 는 `useState` **와 매우 유사하지만 이벤트 핸들러의 state 업데이트 로직을 컴포넌트 외부의 단일 함수로 옮길 수 있다.**
      - reducer 함수 작성하기
          - **관례상 switch 문으로 작성하는 것이 일반적**이다. switch 의 각 case 에 대해 다음 state 를 계산하고 반환해야 한다.
              
              ```jsx
              function reducer(state, action) {
                switch (action.type) {
                  case 'incremented_age': {
                    return {
                      name: state.name,
                      age: state.age + 1
                    };
                  }
                  case 'changed_name': {
                    return {
                      name: action.nextName,
                      age: state.age
                    };
                  }
                }
                throw Error('Unknown action: ' + action.type);
              }
              ```
              
          - `action` 은 어떤 형태든 가질 수 있다.
          관례상 `action`을 식별하는 **type** 프로퍼티가 있는 객체를 전달하는 것이 일반적이다.
              
              ```jsx
              function Form() {
                const [state, dispatch] = useReducer(reducer, { name: 'Taylor', age: 42 });
                
                function handleButtonClick() {
                  dispatch({ type: 'incremented_age' });
                }
              
                function handleInputChange(e) {
                  dispatch({
                    type: 'changed_name',
                    nextName: e.target.value
                  });
                }
                // ...
              ```
              
              <aside>
              💡 `action` 유형 이름은 컴포넌트에 로컬로 지정된다.
              각 `action` 은 **아무리 많은 데이터를 변경하게 되더라도 오직 하나의 상호작용만을 설명해야 한다.**
              
              </aside>
              
          - state 의 모양은 임의적이지만, 일반적으로 객체나 배열이 될 것이다. ( state 는 읽기 전용이므로, state 객체의 배열을 **직접적으로 수정하지 말고**, reducer로부터 **새로운 객체를 반환**해라. )
              
              ```jsx
              // Error
              function reducer(state, action) {
                switch (action.type) {
                  case 'incremented_age': {
                    // 🚩 Don't mutate an object in state like this:
                    state.age = state.age + 1;
                    return state;
                  }
              
              // Good
              function reducer(state, action) {
                switch (action.type) {
                  case 'incremented_age': {
                    // ✅ Instead, return a new object
                    return {
                      ...state,
                      age: state.age + 1
                    };
                  }
              ```
   
            <details>
              
            <summary>
              example ( 기본 예시 - 객체 )
            </summary>
            
              ```jsx
              import { useReducer } from 'react';
              
              function reducer(state, action) {
                switch (action.type) {
                  case 'incremented_age': {
                    return {
                      name: state.name,
                      age: state.age + 1
                    };
                  }
                  case 'changed_name': {
                    return {
                      name: action.nextName,
                      age: state.age
                    };
                  }
                }
                throw Error('Unknown action: ' + action.type);
              }
              
              const initialState = { name: 'Taylor', age: 42 };
              
              export default function Form() {
                const [state, dispatch] = useReducer(reducer, initialState);
              
                function handleButtonClick() {
                  dispatch({ type: 'incremented_age' });
                }
              
                function handleInputChange(e) {
                  dispatch({
                    type: 'changed_name',
                    nextName: e.target.value
                  }); 
                }
              
                return (
                  <>
                    <input
                      value={state.name}
                      onChange={handleInputChange}
                    />
                    <button onClick={handleButtonClick}>
                      Increment age
                    </button>
                    <p>Hello, {state.name}. You are {state.age}.</p>
                  </>
                );
              }
              ```
              
              - `reducer`는 name, age 두 개의 필드가 있는 state 객체를 관리
            </details>
        
            <details>
              
            <summary>
              example ( 기본 예시 - 배열 )
            </summary>
              
              ```jsx
              import { useReducer } from 'react';
              import AddTask from './AddTask.js';
              import TaskList from './TaskList.js';
              
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
              
              export default function TaskApp() {
                const [tasks, dispatch] = useReducer(
                  tasksReducer,
                  initialTasks
                );
              
                function handleAddTask(text) {
                  dispatch({
                    type: 'added',
                    id: nextId++,
                    text: text,
                  });
                }
              
                function handleChangeTask(task) {
                  dispatch({
                    type: 'changed',
                    task: task
                  });
                }
              
                function handleDeleteTask(taskId) {
                  dispatch({
                    type: 'deleted',
                    id: taskId
                  });
                }
              
                return (
                  <>
                    <h1>Prague itinerary</h1>
                    <AddTask
                      onAddTask={handleAddTask}
                    />
                    <TaskList
                      tasks={tasks}
                      onChangeTask={handleChangeTask}
                      onDeleteTask={handleDeleteTask}
                    />
                  </>
                );
              }
              
              let nextId = 3;
              const initialTasks = [
                { id: 0, text: 'Visit Kafka Museum', done: true },
                { id: 1, text: 'Watch a puppet show', done: false },
                { id: 2, text: 'Lennon Wall pic', done: false }
              ];
              ```
              
              - `reducer` 는 배열을 관리한다. - 배열은 **변이 없이** 업데이트 되어야 한다.
            </details>
        
            <details>
              
            <summary>
              example ( Immer 라이브러리 사용 )
            </summary>
              
              ```jsx
              import { useImmerReducer } from 'use-immer';
              import AddTask from './AddTask.js';
              import TaskList from './TaskList.js';
              
              function tasksReducer(draft, action) {
                switch (action.type) {
                  case 'added': {
                    draft.push({
                      id: action.id,
                      text: action.text,
                      done: false
                    });
                    break;
                  }
                  case 'changed': {
                    const index = draft.findIndex(t =>
                      t.id === action.task.id
                    );
                    draft[index] = action.task;
                    break;
                  }
                  case 'deleted': {
                    return draft.filter(t => t.id !== action.id);
                  }
                  default: {
                    throw Error('Unknown action: ' + action.type);
                  }
                }
              }
              
              export default function TaskApp() {
                const [tasks, dispatch] = useImmerReducer(
                  tasksReducer,
                  initialTasks
                );
              
                function handleAddTask(text) {
                  dispatch({
                    type: 'added',
                    id: nextId++,
                    text: text,
                  });
                }
              
                function handleChangeTask(task) {
                  dispatch({
                    type: 'changed',
                    task: task
                  });
                }
              
                function handleDeleteTask(taskId) {
                  dispatch({
                    type: 'deleted',
                    id: taskId
                  });
                }
              
                return (
                  <>
                    <h1>Prague itinerary</h1>
                    <AddTask
                      onAddTask={handleAddTask}
                    />
                    <TaskList
                      tasks={tasks}
                      onChangeTask={handleChangeTask}
                      onDeleteTask={handleDeleteTask}
                    />
                  </>
                );
              }
              
              let nextId = 3;
              const initialTasks = [
                { id: 0, text: 'Visit Kafka Museum', done: true },
                { id: 1, text: 'Watch a puppet show', done: false },
                { id: 2, text: 'Lennon Wall pic', done: false },
              ];
              ```
              
              - Immer 를 사용하면, 반복적인 코드를 줄일 수 있다.
            </details>
      - 초기 state 재생성 방지하기
          
          ```jsx
          // 🚩 Bad Case
          function createInitialState(username) {
            // ...
          }
          
          function TodoList({ username }) {
            const [state, dispatch] = useReducer(reducer, createInitialState(username));
            // ...
          ```
          
          - `createInitialState(username)`의 결과는 초기 렌더링에만 사용되지만, 이후의 모든 렌더링에서도 여전히 이 함수를 호출하게 된다. 이는 큰 배열을 만들거나 값비싼 계산을 수행하는 경우에는 낭비이다.
          - 이 문제를 해결하려면, `useReducer` 세번 째 인수에 **초기화 함수를 전달**할 수 있다.
              
              ```jsx
              // ✅ Good Case
              function createInitialState(username) {
                // ...
              }
              
              // createInitialState 는 username 인수를 받지만, 아무런 정보가 필요 없는 경우에는 null 을 전달
              function TodoList({ username }) {
                const [state, dispatch] = useReducer(reducer, **username, createInitialState**);
                // ...
              
              ```
