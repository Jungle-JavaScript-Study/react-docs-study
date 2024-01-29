# `experimental_taintUniqueValue(errMessage, lifetime, value)`
> 아직 안정적인 버전에서는 사용할 수 없음 + 리액트 서버 컴포넌트 내에서만 사용 가능

: 비밀번호, 키, 토큰, 해쉬 같은 고유값이 클라이언트 컴포넌트에 전달되는 것을 방지
- 민감 정보를 포함한 객체가 전달되는걸 방지하려면 `taintObjectRefernce` 참조
- `message`: `value`가 클라이언트 컴포넌트에 전달되었을 때 던져지는 에러의 일부분으로 보여줄 메세지
- `lifetime`: `value`가 얼마나 오래 오염되어야 하는지 지시하는 아무 객체. 이 객체가 존재하는 동안에는 `value`는 클라이언트 컴포넌트에 전달되지 않는다(ex. `globalThis`를 전달하면 앱의 일생동안 값의 전달을 막을 수 있음). `lifetime`은 일반적으로 프로퍼티에 `value`가 포함된 객체임
- `value`: 문자열, bigint, TypedArray 형태. 반드시 암호화 토큰, 개인 키, 해시, 긴 비밀번호 같이 엔트로피가 높은 문자 또는 바이트의 고유한 시퀀스여야 한다. `value`는 클라이언트 컴포넌트에 전송되는 것이 금지된다.
- `undefined` 리턴
- 오염값에서 새로운 값을 추출하는 것은 보호를 손상시킬 수 있다. 오염된 문자열 value를 더 큰 문자열로 연결하거나, 대문자화하거나, base64로 변환하거나, 부분 문자열로 만들거나 하는 변환 방식들은 그렇게 생성된 새로운 값에 대해 `tainUniqueValue`를 명시적으로 호출하지 않는 한 오염되지 않는다.
- PIN 코드나 전화번호 같이 엔트로피가 낮은 값을 보호하려고 `taintUniqueValue`를 사용하지 말아라! 요청 내의 어떤 값이 공격자에 의해 통제된다면, 가능한 모든 비밀 값을 열거해서 어떤 값이 오염되었는지 추론할 수 있다.

## 토큰이 클라이언트 컴포넌트에 전달되는 것 방지
- 비밀번호, 세션 토큰이나 다른 고유값이무심코 클라이언트 컴포넌트에 전달되지 않는 것을 보장하기 위해 `taintUniqueValue` 함수가 한 겹의 보호를 제공한다.
- 어떤 값이 오염되면, 그 값을 클라이언트 컴포넌트에 전달하려는 시도는 결과적으로 에러를 일으킨다.
- 무기한으로 오염을 유지해야 하는 값은 `globalThis`나 `process` 같은 객체가 `lifetime` 매개변수로써 제공될 수 있다.
- 만약에 오염값의 수명이 어떤 객체에 연결되어 있다면, `lifetime`은 값을 캡슐화하는 그 객체여야 한다(오염값이 캡슐화하는 객체의 수명동안 보호되는 것을 보장함)
  ```javascript
  import {experimental_taintUniqueValue} from 'react';

  export async function getUser(id) {
    const user = await db`SELECT * FROM users WHERE id = ${id}`;
    experimental_taintUniqueValue(
      'Do not pass a user session token to the client.',
      user,
      user.session.token
    );
    return user;
  }
  ```
  이 예제에서는 `user` 객체가 `lifetime` 매개변수로 전달된다. 만약 이 객체가 전역 캐시에 저장되거나 다른 요청에 의해 접근 가능하면 세션 토큰은 오염상태가 유지된다.
- 보안을 tainting에만 의존하지 말아라!

### `server-only`와 `taintUniqueValue`로 비밀 유출 방지하기
- 데이터베이스 암호같은 개인 키나 비밀번호에 접근 권한을 가진 서버 컴포넌트 환경변수를 사용중이라면 그걸 클라이언트 컴포넌트에 전달하지 않도록 조심해야 한다.
  ```javascript
  export async function Dashboard(props) {
    // 이런거 금지! 비밀 API 토큰이 클라이언트에 유출될 수 있고, 이 API 토큰이 어떤 유저가 접근해서는 안되는 데이터에 접근하는데 사용된다면 정보 유출로 이어질 수 있다.
    return <Overview password={process.env.API_PASSWORD} />;
  }
  ```
- 이상적으로는 이런 비밀은 오직 서버의 신뢰할 수 있는 데이터 유틸리티로만 import할 수 있는 단일 헬퍼 파일에 추출되어야 한다. 헬퍼는 이 파일이 클라이언트에서는 import되지 않음을 보장하기 위해 `server-only`로 태그될 수 있다.
  ```javascript
  import "server-only";
  
  export function fetchAPI(url) {
    const headers = { Authorization: process.env.API_PASSWORD };
    return fetch(url, { headers });
  }
  ```
- 그럼에도 일어날 수 있는 실수로부터 보호하기 위해 실제 비밀번호를 "오염"시킬 수도 있다.
  ```javascript
  import "server-only";
  import {experimental_taintUniqueValue} from 'react';
  
  experimental_taintUniqueValue(
    'Do not pass the API token password to the client. ' +
      'Instead do all fetches on the server.'
    process,
    process.env.API_PASSWORD
  );
  ```
- 이제 누가 이 암호를 클라이언트 컴포넌트에 전달하려고 하거나 서버 액턴으로 전송하면, `taintUniqueValue`를 호출할 때 정의한 메세지와 함께 에러가 던져질 것이다.  
