- `**useSyncExternalStore`** 는 외부 스토어를 구독할 수 있는 React Hook
- Reference
    
    ```jsx
    const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)
    ```
    
    ```jsx
    import { useSyncExternalStore } from 'react';
    import { todosStore } from './todoStore.js';
    
    function TodosApp() {
      const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
      // ...
    }
    ```
    
- Parameters
    - **subscribe**
        - 하나의 `callback` 인수를 받아 스토어를 구독하는 함수
        - 스토어가 변경되면 제공된 `callback` 을 호출 **( 컴포넌트 리렌더링 촉발 )**
        - subscribe 함수는 구독을 해제하는 함수를 반환해야 함
    - **getSnapshot**
        - 컴포넌트에 필요한 **스토어 데이터의 스냅샷**을 반환하는 함수
        - 스토어가 변경되지 않은 상태에서 getSnapshot 을 반복적으로 호출하면 동일한 값 반환해야 함
        - 저장소가 변경되어 반환한 값이 달라지면, 컴포넌트 리렌더링
    - getServerSnapshot?
        - 스토어에 있는 데이터의 초기 스냅샷을 반환하는 함수
        - **서버 렌더링과 클라이언트 hydrate 하는 동안에만 사용됨.**
        ( 일반적으로는 서버에서 직렬화해 클라이언트에 전달 )
        - **이 함수가 제공되지 않으면 서버에서 컴포넌트를 렌더링할 때 오류 발생**
- Returns
    - 렌더링 로직에 사용할 수 있는 스토어의 **현재 스냅샷**
- Caveats
    - `getSnapshot` 이 **반환하는 스토어 스냅샷은 불변해야 함**.
    ( 기본 스토어에 변이 가능한 데이터가 있으면, 데이터가 변이되면 새로운 불변 스냅샷을 반환하도록, 변이 사항이 없으면 캐시된 마지막 스냅샷을 반환하도록 할 것 )
    - 리렌더링시에 다른 `subscribe` 함수가 전달되면 React 는 새로 전달된 `subscribe` 함수를 사용해 저장소를 다시 구독함. 
    ( 컴포넌트 외부에서 `subscribe` 를 선언하면 이를 방지할 수 있음 )
- Usage
    - **외부 스토어 구독하기**
        - 대부분의 React 컴포넌트는 `props`, `state`, `context`에서만 데이터를 읽음.
        하지만, React 외부의 저장소에서 데이터를 읽어야 하는 경우도 있음.
        ( **React 외부에 state 를 보관하는 서드파티 상태 관리 라이브러리** 또는 **변이 가능한 값을 노출하는 브라우저 API 와 그 변이 사항을 구독하는 이벤트** )
            
            ```jsx
            import { useSyncExternalStore } from 'react';
            import { todosStore } from './todoStore.js';
            
            function TodosApp() {
            // todos : 스토어에 있는 데이터의 snapshot 반환
            // subscribe 함수는 스토어를 구독해야하고, 구독 취소 함수를 반환해야 함.
            // getSnapshot 함수는 스토어에서 데이터의 스냅샷을 읽어야 함.
              const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
              // ...
            }
            
            ```
            
            - React 는 이 함수를 사용해 컴포넌트가 스토어를 구독한 상태로 유지하고 변경 사항이 있을 때 다시 렌더링 함.
            
            <aside>
            💡 `useSyncExternalStore` API 는 주로 **기존의 비 React 코드와 통합해야 할 때 유용**하다.
            ( 가능하면 React 빌트인 state 를 쓸 것 ! )
            
            </aside>
            
    - **브라우저 API 구독하기**
        - `useSyncExternalStore` 는 **브라우저상의 시간이 지남에 따라 변경되는 일부 값을 구독하는 경우**에 쓰임.
            
            ```jsx
            import { useSyncExternalStore } from 'react';
            // navigator.onLine : 컴포넌트에 네트워크 연결이 활성화되어 있는지 여부를 표시하는 브라우저 API
            export default function ChatIndicator() {
              const isOnline = useSyncExternalStore(subscribe, getSnapshot);
              return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
            }
            
            function getSnapshot() {
              return navigator.onLine;
            }
            
            function subscribe(callback) {
              window.addEventListener('online', callback);
              window.addEventListener('offline', callback);
              return () => {
                window.removeEventListener('online', callback);
                window.removeEventListener('offline', callback);
              };
            }
            ```
            
    - **커스텀 훅으로 로직 추출하기**
        - 일반적으로 컴포넌트에서 직접 `useSyncExternalStore` 를 작성하지 않음.
        대신 커스텀훅으로 만들면, 서로 다른 컴포넌트에서 동일한 외부 저장소를 사용할 수 있음.
            
            ```jsx
            // useOnlineStatus.js
            import { useSyncExternalStore } from 'react';
            
            export function useOnlineStatus() {
              const isOnline = useSyncExternalStore(subscribe, getSnapshot);
              return isOnline;
            }
            
            function getSnapshot() {
              return navigator.onLine;
            }
            
            function subscribe(callback) {
              window.addEventListener('online', callback);
              window.addEventListener('offline', callback);
              return () => {
                window.removeEventListener('online', callback);
                window.removeEventListener('offline', callback);
              };
            }
            
            // App.js
            import { useOnlineStatus } from './useOnlineStatus.js';
            
            function StatusBar() {
              const isOnline = useOnlineStatus();
              return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
            }
            
            function SaveButton() {
              const isOnline = useOnlineStatus();
            
              function handleSaveClick() {
                console.log('✅ Progress saved');
              }
            
              return (
                <button disabled={!isOnline} onClick={handleSaveClick}>
                  {isOnline ? 'Save progress' : 'Reconnecting...'}
                </button>
              );
            }
            
            export default function App() {
              return (
                <>
                  <SaveButton />
                  <StatusBar />
                </>
              );
            }
            ```
            
    - **서버 렌더링의 지원 추가하기**
        - React 앱이 서버 렌더링을 사용하는 경우,
        React 컴포넌트는 브라우저 환경 외부에서도 실행되어 초기 HTML 을 생성.
        → 외부 스토어에 연결할 때 문제가 발생함 ( 브라우저 전용 API : 서버에 해당 API 가 존재하지 않으므로 작동 X , 타사 데이터 저장소 : 서버와 클라이언트 간에 일치하는 데이터 필요 )
            
            ```jsx
            import { useSyncExternalStore } from 'react';
            
            export function useOnlineStatus() {
              const isOnline = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
              return isOnline;
            }
            
            function getSnapshot() {
              return navigator.onLine;
            }
            // 이로 인해 서버 렌더링에 의미 있는 초기값이 없다면 컴포넌트가 클라이언트에서만 렌더링되도록 강제 설정할 수 있음.
            **function getServerSnapshot() {
              return true; // Always show "Online" for server-generated HTML
            }**
            
            function subscribe(callback) {
              // ...
            }
            ```
            
- TroubleShooting
    - **I’m getting an error: “The result of `getSnapshot` should be cached”**
        - 스냅샷은 불변해야 함.