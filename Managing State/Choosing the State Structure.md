## state 구조화 원칙 
1. 관련된 state는 그룹화
    1. 여러개의 state가 항상 함께 변경되는 경우
    2. 필요한 state 수를 모를 경우(ex. 사용자가 필드를 추가할 수 있을 때)
2. 불필요한 state는 삭제
    1. 여러 state가 모순될 떄(`isSending`과 `isSent`는 동시에 `true`일 수 없음)<br>->편의를 위한 변수는 state가 아니라 상수로 선언하면 동기화 문제를 걱정하지 않아도 됨
    2. props나 기존 state로부터 계산할 수 있는 경우
    3. 이미 존재하는 state의 일부를 그대로 저장하는 것<br>->`selectedItem` 대신 `selectedId`로 대체
6. 깊게 중첩된 형태의 state 피하기(객체가 깊게 중첩되면 deep copy를 깊게 들어가야 함)<br>->깊이 중첩된 state는 flat하게 바꾸기<br>
   ex.
   ```javascript
   export const initialTravelPlan = {
     id: 0,
     title: '(Root)',
     childPlaces: [{
       id: 1,
       title: 'Earth',
       childPlaces: [{
         id: 2,
         title: 'Africa',
         childPlaces: [{
           id: 3,
           title: 'Botswana',
           childPlaces: []
         }]
       }]
     }]
   };

   ⬇︎

   export const initialTravelPlan = {
      0: {
        id: 0,
        title: '(Root)',
        childIds: [1, 43, 47],
      },
      1: {
        id: 1,
        title: 'Earth',
        childIds: [2, 10, 19, 27, 35]
      },
      2: {
        id: 2,
        title: 'Africa',
        childIds: [3, 4, 5, 6 , 7, 8, 9]
      }, 
      3: {
        id: 3,
        title: 'Botswana',
        childIds: []
      }
    };
   ```
