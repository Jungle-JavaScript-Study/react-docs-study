# `experimental_taintObjectReference(message, object)`
> 안정적인 버전에서는 아직 사용할 수 없음. 리액트 서버 컴포넌트 내에서만 사용 가능.

: 특정 인스턴스가 클라이언트 컴포넌트로 전달되는걸 막을 수 있다
- `object`를 통해 클라이언트에 전달되면 안되는 것들을 리액트에 등록
- 키, 해쉬, 토큰이 전달되는걸 막으려면 `taintUniqueValue` 참조
- `message`: object가 클라이언트 컴포넌트에 전달되었을 때 던져지는 에러의 부분으로 보여줄 메세지 
- `object`: 오염되는 객체. 함수나 클래스 인스턴스가 `object`가 될 수 있다. 함수와 클래스는 이미 클라이언트 컴포넌트에 전달되는게 금지되어 있긴 하지만 리액트의 기본 에러 메세지가 `message`로 대체된다. 만약 형식화 배열의 특정 인스턴스가 `object`로 전달되면 그 형식화 배열의 다른 복사본은 오염되지 않는다.
-  `undefined` 리턴
-  오염된 객체를 재생성하거나 복제하는 것은 민감 정보를 포함하는 새로운 비오염 객체를 만들어낸다. `taintObjectReferece`는 해당 객체가 바뀌지 않은 채로 클라이언트 컴포넌트에 전달될 때만 보호할 수 있다.
-  보안을 tainting에만 의존해서는 안된다! 객체를 오염시키는 것은 모든 파생될 수 있는 값들이 새어나가는 것을 막아주지 않는다. 예를 들어, 오염된 객체의 데이터를 사용하면 오염되지 않은 새로운 값이나 객체를 만들 수 있다(ex. `{secret: taintedObj.secret}`). tainting은 한 겹의 보호막이다. 안전한 앱은 여러겹의 보호막, 잘 디자인된 API, 격리 패턴을 가져야 한다.

## 유저 데이터가 의도치 않게 클라이언트에 도달하는 것을 방지하기
- 클라이언트 컴포넌트는 결코 민감 정보가 들은 객체를 받아서는 안된다.
- 이상적으로는 데이터 페칭 함수는 현재의 유저가 접근하면 안되는 데이터를 노출하면 안된다. 그러나 가끔 실수할 수도 있기 때문에 이에 대비하기 위해 데이터 API의 유저 객체를 "오염"시킬 수 있다.
  ```javascript
  import {experimental_taintObjectReference} from 'react';

  export async function getUser(id) {
    const user = await db`SELECT * FROM users WHERE id = ${id}`;
    experimental_taintObjectReference(
      'Do not pass the entire user object to the client. ' +
        'Instead, pick off the specific properties you need for this use case.',
      user,
    );
    return user;
  }
  ```
- 이제 누구든 이 객체를 클라이언트 컴포넌트에 전달하려고 하면 대신 전달된 에러 메세지와 함께 에러가 던져질 것이다.

### 데이터 페칭에서의 누수 보호하기
- 민감 정보에 대한 접근 권한을 가지고 있는 서버 컴포넌트를 사용중이라면 객체를 바로 전달하지 않도록 주의해야 한다.
  ```javascript
  export async function Profile(props) {
    const user = await getUser(props.userId);
    // 이거 금지!!
    return <InfoCard user={user} />;
  }
  ```
- 이상적으로는 `getUser`는 현재 사용자가 접근하면 안되는 정보를 유출해서는 안됨. 이럴 때 유저 객체를 "오염"시켜서 클라이언트 컴포넌트에 전달되는 것을 방지할 수 있다.
  ```javascript
  // api.js
  import {experimental_taintObjectReference} from 'react';
  
  export async function getUser(id) {
    const user = await db`SELECT * FROM users WHERE id = ${id}`;
    experimental_taintObjectReference(
      'Do not pass the entire user object to the client. ' +
        'Instead, pick off the specific properties you need for this use case.',
      user,
    );
    return user;
  }
  ```
