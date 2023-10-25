# 6. Context로 데이터 깊숙이 전달하기

보통 컴포넌트 간 정보를 전달할 때는 부모 컴포넌트에서 자식 컴포넌트로 props를 통해 전달한다.
- 중간에 많은 컴포넌트를 거쳐야 하거나
- 여러 컴포넌트가 동일한 정보를 필요로 하는 경우
props를 전달하는 것은 번거롭고 불편할 수 있다.

Context는 부모 컴포넌트가 그 아래 트리의 어떤 컴포넌트에든지 얼마나 깊든지 정보를 사용할 수 있게 한다.

## 6-1. props 전달의 문제

- props를 전달하는 것은 UI 트리를 통해 데이터를 사용하는 컴포넌트에 명시적으로 전달하는 좋은 방법이다.
- 하지만, 데이터가 필요한 컴포넌트로부터 가장 가까운 공통의 조상이 멀리 떨어져 있을 경우, 높은 위치로 상태를 끌어올리는 Prop drilling을 초래할 수 있다.

![](https://velog.velcdn.com/images/e_juhee/post/70c4ed39-9013-4e5a-9e1b-8c3defc52812/image.png)

## 6-2. Context: props 전달의 대안 

- Context를 사용하면 아래 전체 트리에 데이터를 제공할 수 있다.

![](https://velog.velcdn.com/images/e_juhee/post/07fee4e8-b045-4527-bf9a-7ebf90b802c7/image.png)

### 1) Context 만들기
- createContext에 기본값을 인자로 전달한다.

```js
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

### 2) Context 사용하기

- 아직 context를 제공하지 않았으므로 1에서 지정한 기본값 1이 반환된다.

```js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  // ...
}
```

### 3) Context 제공하기

- LevelContext로 감싼 컴포넌트가 LevelContext를 요청하면 level을 제공하게 된다.


```js
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

- 컴포넌트는 UI 트리에서 위에 있는 것들 중 가장 가까운 `<LevelContext.Provicer>`의 값을 사용한다.

```js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

## 6-3. Context는 중간 컴포넌트들을 통과한다.

- Context를 제공하는 컴포넌트와 사용하는 컴포넌트 사이에 컴포넌트들을 삽입해도 된다.
- 알아서 가장 가까운 Context를 가져온다. (마치 CSS 상속..)
- 이를 활용하면 어떤 컨텍스트에서 렌더링되는지에 따라 다르게 표시되는 컴포넌트를 만들 수 있다.
- `createContext()`로 만드는 각 컨텍스트는 다른 것들과 완전히 분리되어 있으며, 특정 컨텍스트를 사용하고 제공하는 컴포넌트들을 연결한다.
- 하나의 컴포넌트는 여러 다른 컨텍스트들을 사용하거나 제공할 수 있다.

## 6-4. 사용하기 전에!
- 남용 노노
- props를 깊이 전달해야 한다고 해서 context를 써야 하는 것은 아님

### Context 사용 전 고려할 대안들
#### prop로 일단 전달
- props로 전달하면 어떤 컴포넌트가 어떤 데이터를 사용하는지 매우 명확해진다.
- 유지보수하는 사람이 봤을 때 데이터 흐름이 명시적이어 좋을 것.

#### 컴포넌트를 추출하고 JSX를 children으로 전달
- 데이터를 사용하지 않는 중간 컴포넌트의 많은 레이어를 통해 데이터를 전달하고 있다면, 중간에 일부 컴포넌트를 추출하는 것을 잊었다는 것을 의미한다!

```jsx
<Layout posts={posts} />
대신
<Layout><Posts posts={posts} /></Layout>
이렇게 하고 Layout이 children을 prop으로 쓰도록 하자!
```

- 데이터를 지정하는 컴포넌트와 필요한 컴포넌트 사이의 레이어 수를 줄일 수 있다.

이 두 가지를 했는데도 적합하지 않을 경우에 context를 고려하자.

## 6-5. Context 사용 사례

### 1) 테마 변경
- 다크모드처럼 사용자가 앱의 외관을 변경할 수 있게 해주는 경우!
- 앱 상단에 컨텍스트 제공자를 두고 아래에서 사용 ㄱ

### 2) 현재 계정
- 현재 로그인한 사용자를 많은 컴포넌트에서 알아야겠지
- 여러 계정을 동시에 운영할 수 있게 해주는 앱일 경우, 다른 현재 계정 값으로 UI의 일부를 감싸면 편할 것

### 3) 라우팅
- 대부분의 라우팅 솔루션은 내부적으로 현재 라우트를 보유하기 위해 컨텍스트를 사용한다.

### 4) State 관리
- 앱이 성장함에 따라 앱 상단에 많은 상태를 가지게 될 수 있다.
- 복잡한 상태를 관리하고 먼 컴포넌트에 번거로움 없이 전달하기 위해 컨텍스트와 함께 리듀서를 사용하는 것이 일반적이다.

Context는 정적인 값에만 제한되지 않는다.
렌더링에서 다른 값을 전달하면 React는 아래에서 이를 읽는 모든 컴포넌트를 업데이트한다.
이것이 context가 state와 자주 사용되는 이유!

일반적으로, 트리의 다른 부분에 있는 먼 컴포넌트에 있는 정보가 필요하다면, 컨텍스트가 아주 도움이 될 것이다.
