- `useDebugValue` 는 **React 개발자 도구**에서 커스텀 훅에 레이블을 추가해주는 React Hook
- Reference
    
    ```jsx
    // 읽을 수 있는 디버그 값 표시하려면, 커스텀 훅의 최상위 레벨에서 useDebugValue 호출
    useDebugValue(value, format?)
    ```
    
    ```jsx
    import { useDebugValue } from 'react';
    
    function useOnlineStatus() {
      // ...
      useDebugValue(isOnline ? 'Online' : 'Offline');
      // ...
    }
    ```
    
- Parameters
    - value : React 개발자도구에서 표시하려는 값 ( 모든 타입 가능 )
    - format? : 포매팅 함수. 컴포넌트가 검사할 때, React 개발자 도구는 인수로 포매팅 함수를 호출한 다음 반환된 포매팅된 값을 표시한다.
- Returns : void
- Usage
    - **커스텀 훅에 레이블 추가**
        
        ```jsx
        import { useDebugValue } from 'react';
        
        function useOnlineStatus() {
          // ...
          useDebugValue(isOnline ? 'Online' : 'Offline');
          // ...
        }
        ```
        
        ![`useDebugValue` 호출이 없으면 기본 데이터(예시에서는 `true`)만 표시됩니다.](https://prod-files-secure.s3.us-west-2.amazonaws.com/20991e95-6e3a-45e4-bccc-2672d974adb9/53b01695-ca2d-48f0-93e1-27039da48673/Untitled.png)
        
        `useDebugValue` 호출이 없으면 기본 데이터(예시에서는 `true`)만 표시됩니다.
        
        <aside>
        💡 모든 커스텀훅에 디버그 값을 추가하지 마시오 !
        공유 라이브러리에서 **검사하기 어려운 복잡한 내부 데이터를 가진 커스텀 훅**에 가장 **유용**합니다.
        
        </aside>
        
    - **디버그 값의 형식 지정 연기**
        
        ```jsx
        useDebugValue(date, date => date.toDateString());
        ```
        
        - 이렇게하면 컴포넌트를 실제로 조사하지 않는 한 비용이 많이 들 수 있는 포매팅 로직을 실행하지 않아도 됩니다.
        ex> `date` 가 날짜 값인 경우, 컴포넌트를 렌더링 할 때마다 toDateString() 을 호출하지 않습니다.