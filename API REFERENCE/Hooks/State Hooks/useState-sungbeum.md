****useState****
  - `useState` 는 컴포넌트에 **state 변수**를 추가할 수 있게 해주는 **React 훅**이다.
  
  ```jsx
  const [state, setState] = useState(initialState);
  ```
  
  - Reference
      - **useState(initialState)** : 컴포넌트 최상위 레벨에서 `useState` 를 호출해 state 변수를 선언한다
          
          ```jsx
          import { useState } from 'react';
          
          function MyComponent() {
            const [age, setAge] = useState(28);
            const [name, setName] = useState('Taylor');
            const [todos, setTodos] = useState(() => createTodos());
            // ...
          ```
          
          - 배열 구조 분해를 사용해 `[something, setSomething]` 과 같은 state 변수의 이름을 지정하는 것이 관례이다.
          - **Parameters**(initialState) : 초기에 state 를 설정할 값이다. 모든 데이터 타입이 허용되지만, 함수이면 이를 *초기화함수*(순수해야하고, 인자를 받지 않아야하며, 반드시 어떤 값을 반환해야한다.)로 취급한다. ( 이 인자는 초기 렌더링 이후에는 무시된다. )
          - **Returns** : useState 는 정확히 두 개의 값을 가진 배열을 반환한다.
          - **Caveats(주의사항)** : useState 는 React훅이다. (컴포넌트의 최상위 레벨이나 커스텀훅에서만 호출 가능)
      - **set 함수(setSomething(nextState))** : `useState` 가 반환하는 **set 함수**를 사용하면 state 를 다른 값으로 업데이트하고 리렌더링을 촉발할 수 있다.
          - **Parametes(nextState)** : state 가 될 값. 모든 데이터 타입을 허용하지만, **함수는 업데이터 함수로 취급**된다.(순수하고, 대기중인 state 를 유일한 인수로 사용해야하고, 다음 state 를 반환해야 한다.)
          - **Returns** : void ( set 함수는 반환값이 없다. )
          - **Caveats(주의사항)**
              - set함수는 **다음 렌더링에 대한 state 변수만 업데이트** 한다. ( set 함수를 호출한 후에도 state 변수에는 여전히 호출 전 화면에 있던 **이전 값이 담겨 있다.** )
              - 사용자가 제공한 새로운 값이 [Object.is](http://Object.is) 에 의해 현재 state 와 동일하다고 판정되면, React 는 **컴포넌트와 그 자식들을 리렌더링하지 않는다**.(React 의 최적화)
              - React 는 State 업데이트를 일괄처리 한다. **모든 이벤트 핸들러가 실행되고** set 함수를 호출한 후에 화면을 업데이트 한다. ( 드물지만 React가 화면을 더 일찍 업데이트하도록 강제해야하는 경우 `flushSync` 를 사용할 수 있다. )
  - Usage
      - 컴포넌트에 state 추가하기
          
          <aside>
          💡 set 함수를 호출해도 <strong>이미 실행 중인 코드의 현재 state 는 변경되지 않는다.</strong> - set 함수는 다음 렌더링에서 반환할 `useState` 에만 영향을 준다.
          
          </aside>
          
          - 동일한 컴포넌트에 두 개 이상의 state 변수를 선언할 수 있다. - **각 state 변수는 완전히 독립적**이다.
      - 이전 state 를 기반으로 state 업데이트하기
          - `age` 가 `42` 라고 가정하면, 아래의 핸들러는 `setAge(age + 1)` 를 세 번 호출한다.
              
              ```jsx
              function handleClick() {
                setAge(age + 1); // setAge(42 + 1)
                setAge(age + 1); // setAge(42 + 1)
                setAge(age + 1); // setAge(42 + 1)
              }
              ```
              
              - 문제를 해결하기 위해 setAge 에 업데이터 함수를 전달할 수 있다.
                  
                  ```jsx
                  function handleClick() {
                    setAge(a => a + 1); // setAge(42 => 43)
                    setAge(a => a + 1); // setAge(43 => 44)
                    setAge(a => a + 1); // setAge(44 => 45)
                  }
                  ```
                  
      - 객체 및 배열 state 업데이트
          - state 에는 객체와 배열도 넣을 수 있다. - React 에서 state 는 읽기 전용으로 간주되므로 **기존 객체를 변이하지 않고, 교체를 해야 한다.**
              
              ```jsx
              // 🚩 state 안에 있는 객체를 다음과 같이 변이하지 마세요: 
              form.firstName = 'Taylor';
              
              // ✅ 새로운 객체로 state 교체합니다
              setForm({
                ...form,
                firstName: 'Taylor'
              });
              ```
          <details>
            <summary>
            example ( 중첩 객체 )
            </summary>
              

          ```jsx
          // 중첩객체
          import { useState } from 'react';
          
          export default function Form() {
            const [person, setPerson] = useState({
              name: 'Niki de Saint Phalle',
              artwork: {
                title: 'Blue Nana',
                city: 'Hamburg',
                image: 'https://i.imgur.com/Sd1AgUOm.jpg',
              }
            });
          
            function handleNameChange(e) {
              setPerson({
                ...person,
                name: e.target.value
              });
            }
          
            function handleTitleChange(e) {
              setPerson({
                ...person,
                artwork: {
                  ...person.artwork,
                  title: e.target.value
                }
              });
            }
          
            function handleCityChange(e) {
              setPerson({
                ...person,
                artwork: {
                  ...person.artwork,
                  city: e.target.value
                }
              });
            }
          
            function handleImageChange(e) {
              setPerson({
                ...person,
                artwork: {
                  ...person.artwork,
                  image: e.target.value
                }
              });
            }
          
            return (
              <>
                <label>
                  Name:
                  <input
                    value={person.name}
                    onChange={handleNameChange}
                  />
                </label>
                <label>
                  Title:
                  <input
                    value={person.artwork.title}
                    onChange={handleTitleChange}
                  />
                </label>
                <label>
                  City:
                  <input
                    value={person.artwork.city}
                    onChange={handleCityChange}
                  />
                </label>
                <label>
                  Image:
                  <input
                    value={person.artwork.image}
                    onChange={handleImageChange}
                  />
                </label>
                <p>
                  <i>{person.artwork.title}</i>
                  {' by '}
                  {person.name}
                  <br />
                  (located in {person.artwork.city})
                </p>
                <img 
                  src={person.artwork.image} 
                  alt={person.artwork.title}
                />
              </>
            );
          }
          ```
                        
          </details>
   
          <details>
            <summary>
            example ( Immer 로 간결한 업데이트 로직 작성 )
            </summary>
            
              
          ```jsx
          // Immer 라이브러리 사용
          import { useState } from 'react';
          import { useImmer } from 'use-immer';
          
          let nextId = 3;
          const initialList = [
            { id: 0, title: 'Big Bellies', seen: false },
            { id: 1, title: 'Lunar Landscape', seen: false },
            { id: 2, title: 'Terracotta Army', seen: true },
          ];
          
          export default function BucketList() {
            const [list, updateList] = useImmer(initialList);
          
            function handleToggle(artworkId, nextSeen) {
              updateList(draft => {
                const artwork = draft.find(a =>
                  a.id === artworkId
                );
                artwork.seen = nextSeen;
              });
            }
          
            return (
              <>
                <h1>Art Bucket List</h1>
                <h2>My list of art to see:</h2>
                <ItemList
                  artworks={list}
                  onToggle={handleToggle} />
              </>
            );
          }
          
          function ItemList({ artworks, onToggle }) {
            return (
              <ul>
                {artworks.map(artwork => (
                  <li key={artwork.id}>
                    <label>
                      <input
                        type="checkbox"
                        checked={artwork.seen}
                        onChange={e => {
                          onToggle(
                            artwork.id,
                            e.target.checked
                          );
                        }}
                      />
                      {artwork.title}
                    </label>
                  </li>
                ))}
              </ul>
            );
          }
          ```
          </details>
      - 초기 state 다시 생성하지 않기
          - React는 초기 state를 한 번 저장하고 다음 렌더링부터는 이를 무시한다.
              
              ```jsx
              function TodoList() {
                const [todos, setTodos] = useState(createInitialTodos());
                // ...
              ```
              
              - `createInitialTodos()`의 결과는 초기 렌더링에만 사용되지만, 여전히 **모든 렌더링에서 이 함수를 호출**하게 됩니다. 이는 큰 배열을 생성하거나 값비싼 계산을 수행하는 경우 낭비가 될 수 있다.
              - 이 문제를 해결하려면, `useState` 에 **초기화 함수로 전달**하면 된다. - 함수를 호출한 결과인 `createInitialTodos()`가 아니라 함수 자체인`createInitialTodos`를 전달하면, React는 초기화 중에만 함수를 호출한다.
                  
                  ```jsx
                  function TodoList() {
                    const [todos, setTodos] = useState(createInitialTodos);
                    // ...
                  ```
                  
      - key 로 state 재설정하기
          - 컴포넌트에 다른 key 를 전달해 컴포넌트의 state 를 재설정 할 수도 있다.
              
              ```jsx
              import { useState } from 'react';
              
              export default function App() {
                const [version, setVersion] = useState(0);
              
                function handleReset() {
                  setVersion(version + 1);
                }
              
                return (
                  <>
                    <button onClick={handleReset}>Reset</button>
                    <Form key={version} />
                  </>
                );
              }
              
              function Form() {
                const [name, setName] = useState('Taylor');
              
                return (
                  <>
                    <input
                      value={name}
                      onChange={e => setName(e.target.value)}
                    />
                    <p>Hello, {name}.</p>
                  </>
                );
              }
              
              //이 예제에서는 Reset 버튼이 version state 변수를 변경하고, 이를 Form에 key로 전달한다.
              //key가 변경되면 React는 Form 컴포넌트(및 그 모든 자식)를 처음부터 다시 생성하므로 state가 초기화된다.
              ```
              
      - 이전 렌더링에서 얻은 정보 저장하기
          - 보통은 이벤트 핸들러에서 state 를 업데이트하지만, **렌더링에 대한 응답으로 state를 조정해야 하는 경우도 있다**. ( ex > props 가 변경될 때 state 변수를 변경하고 싶을 수 있다. )
              - 대부분 이 기능은 필요하지 않다.
                  - **필요한 갑을 현재 props 나 다른 state 에서 모두 계산할 수 있는 경우 → 중복되는 state 를 모두 제거해라.**
                  - 전체 컴포넌트 트리의 state 를 재설정하려는 경우 → 컴포넌트에 다른 key 를 전달해라
                  - 가능하다면 이벤트 핸들러의 모든 관련 state 를 업데이트해라.
                  - 위와 같은 경우들이 아니면, **컴포넌트가 렌더링되는 동안 set 함수를 호출해 지금까지 렌더링된 값을 기반으로 state 를 업데이트하는 데 사용하는 패턴이 있을 수 있다.**
                  
                  ```jsx
                  // app.js
                  import { useState } from 'react';
                  import CountLabel from './CountLabel.js';
                  
                  export default function App() {
                    const [count, setCount] = useState(0);
                    return (
                      <>
                        <button onClick={() => setCount(count + 1)}>
                          Increment
                        </button>
                        <button onClick={() => setCount(count - 1)}>
                          Decrement
                        </button>
                        <CountLabel count={count} />
                      </>
                    );
                  }
                  
                  // CountLabel.js
                  import { useState } from 'react';
                  
                  export default function CountLabel({ count }) {
                    const [prevCount, setPrevCount] = useState(count);
                    const [trend, setTrend] = useState(null);
                    if (prevCount !== count) {
                      setPrevCount(count);
                      setTrend(count > prevCount ? 'increasing' : 'decreasing');
                    }
                    return (
                      <>
                        <h1>{count}</h1>
                        {trend && <p>The count is {trend}</p>}
                      </>
                    );
                  }
                  ```
                  
                  - 렌더링하는 동안 set 함수를 호출하는 경우, 그 set 함수는 `prevCount ≠ count` 와 같은 조건안에 있어야하며, 조건 내부에 `setPrevCount(count)` 와 같은 호출이 있어야 한다.
                  - 또한 이 방식은 현재 렌더링 중인 컴포넌트의 state 만을 업데이트 할 수 있다. - **렌더링 중에 다른 컴포넌트의 set 함수를 호출하는 것은 에러**이다.
                  
                  <aside>
                  💡 이 패턴은 일반적으로 피하는 것이 가장 좋지만, <strong>Effect 에서 state 를 업데이트 하는 것보다는 낫다 !</strong>
                  </aside>
