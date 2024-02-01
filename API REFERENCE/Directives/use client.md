#### Canary
- `'use client'`는 React 서버 컴포넌트를 사용할 때만 필요하다.

#### 서버 컴포넌트
- 서버 컴포넌트는 새로운 애플리케이션 아키텍처이다.
- 미리 실행되고 JavaScript 번들에서 제외되는 새로운 종류의 컴포넌트이다.
- 빌드 중에 실행할 수 있으며 파일 시스템에서 읽거나 정적 콘텐츠를 가져올 수 있다.
- Next.js 앱 라우터에 적용되었다.
- 서버 컴포넌트는 React 프레임워크에서 작동하는 컴포넌트의 명세서로 고려되고 있다.

#### React 서버 컴포넌트
- 클라이언트 사이드에서만 렌더링되는 일반 React 컴포넌트와 달리 서버 사이드에서 렌더링되어 클라이언트로 전송되는 컴포넌트

#### 클라이언트에서 실행
- 브라우저에서 실행하는 것

# 'use client'
- React 서버 컴포넌트 환경에서 사용되는 지시어로, 특정 코드 블록이 클라이언트 사이드에서 실행되어야 함을 명시한다.
- 파일 상단에 'use client'를 추가하면 해당 파일과 그 파일이 의존하는 다른 파일들까지 클라이언트에서 실행되어야 하는 코드로 표시된다.
- 서버 컴포넌트에서 'use client'로 표시된 파일을 import 하면, 호환되는 번들러는 모듈 가져오기를 서버 실행 코드와 클라이언트 실행 코드 사이의 경계로 처리한다.

```js
'use client';

import { useState } from 'react';
import { formatDate } from './formatters';
import Button from './button';

export default function RichTextEditor({ timestamp, text }) {
  const date = formatDate(timestamp);
  // ...
  const editButton = <Button />;
  // ...
}
```

- 위 컴포넌트 `RichTextEditor`에 종속된 `formatDate`와 `Button`에 'use client'가 없어도 클라이언트에서 실행된다.
- 하나의 모듈은 서버에서 import되면 서버에서만 평가되고, 클라이언트에 import 되면 클라이언트에서만 평가된다.

## 주의사항
- 'use client'는 파일의 맨 처음에 작성해야 한다. (주석은 상관 없음)
- 백틱은 쓰면 안 되고 `''` `""`는 가능
- 클라이언트에서 렌더링되는 모듈에서 'use client'가 있는 모듈을 가져오면 아무런 효과가 없다.
- 'use client'가 포함된 모듈에서 정의되거나 의존된 컴포넌트는 클라이언트 컴포넌트로 간주되고, 이외에는 서버 컴포넌트로 간주된다.
- 클라이언트에서 평가하기 위해 표시된 코드는 컴포넌트에만 국한되는 것이 아니고, 클라이언트 모듈의 일부로 분류되는 모든 코드가 클라이언트에서 실행된다.
- 서버에서 'use client' 모듈의 값을 가져올 때는, 가져오려는 값이 React 컴포넌트이거나 클라이언트 컴포넌트에 전달될 수 있는 직렬화 가능한 prop 값이어야 한다.

## 작동 방식
- 컴포넌트들은 종종 분리된 파일이나 모듈로 나뉜다.
- 서버 컴포넌트를 사용하는 앱들은 기본적으로 서버에서 렌더링된다.
- 'use client'는 모듈 의존성 트리에 서버와 클라이언트 사이의 경계선을 그어주는 역할을 한다. 이 경계선을 사용함으로써 서버에서 처리되는 모듈들과 별도로 클라이언트에서 실행될 모듈들의 하위 트리를 만들어내는 셈이다.

```jsx
import FancyText from './FancyText';
import InspirationGenerator from './InspirationGenerator';
import Copyright from './Copyright';

export default function App() {
  return (
    <>
      <FancyText title text="Get Inspired App" />
      <InspirationGenerator> 👈 'use client'가 포함된 컴포넌트
        <Copyright year={2004} />
      </InspirationGenerator>
    </>
  );
}
```

```jsx
'use client';

import { useState } from 'react';
import inspirations from './inspirations';
import FancyText from './FancyText';

export default function InspirationGenerator({children}) {
  const [index, setIndex] = useState(0);
  const quote = inspirations[index];
  const next = () => setIndex((index + 1) % inspirations.length);

  return (
    <>
      <p>Your inspirational quote is:</p>
      <FancyText text={quote} />
      <button onClick={next}>Inspire me again</button>
      {children}
    </>
  );
}
```

### 의존성 트리

<img width="573" alt="image" src="https://github.com/Jungle-JavaScript-Study/react-docs-study/assets/82787570/ed510508-5f62-420b-b4e9-7a463600be48">

- 의존성 트리를 보면 'use client'가 포함된 컴포넌트 `InspirationGenerator`와 그 하위 트리인 `FancyText`, `inspirations`가 클라이언트 모듈로 표시된다.

### 렌더링 과정

<img width="523" alt="image" src="https://github.com/Jungle-JavaScript-Study/react-docs-study/assets/82787570/e84f7149-b060-42b2-9733-9f7b9e16d7bd">

1. 서버에서 루트 컴포넌트 렌더링
2. 클라이언트로 표시된 코드는 무시
3. 서버에서 렌더링된 부분을 클라이언트로 전송
4. 이미 가지고 있는 다운로드된 클라이언트 코드를 통해 트리의 나머지 부분을 렌더링

정리해보자면,,
- 서버 컴포넌트: APP, Fancy Text, Copyright
- 클라이언트 컴포넌트: InspirationGenerator, FancyText

#### FancyText
위에서 `FancyText`는 서버 컴포넌트와 클라이언트 컴포넌트 두 곳에 속해있다..!!!!
어찌 그러한지 알아보자!

우선, '컴포넌트'라는 용어가 명확하지 않다.
'컴포넌트'는 아래 두 가지를 의미할 수 있다.
- 컴포넌트의 정의
- 컴포넌트 사용

일반적으로 개념을 설명할 때 이러한 불명확성은 중요하지 않을 수 있지만, 지금은 중요하다..

서버 컴포넌트 또는 클라이언트 컴포넌트에 대해 얘기할 때는 컴포넌트의 사용을 의미한다.

- 컴포넌트가 'use client'가 있는 모듈에서 정의되었거나, 클라이언트 컴포넌트에서 가져와서 사용된 경우, 이 컴포넌트의 사용은 클라이언트 컴포넌트로 간주된다.
- 그렇지 않은 경우 컴포넌트의 사용은 서버 컴포넌트로 간주된다.

`FancyText` 문제로 돌아가보자
- `FancyText` 컴포넌트의 **정의**에는 'use client' 지시어가 없다.
- `FancyText`이 `App`의 자식으로 사용될 때, 이 사용은 서버 컴포넌트로 표시된다. 이 컨텍스트에서는 서버에서 렌더링된다.
- `FancyText`을 `InspirationGenerator`에서 가져와서 **사용**할 때 이 사용은 클라이언트 컴포넌트로 간주된다. (`InspirationGenerator`에 'use client'가 있어서) 클라이언트에서 렌더링되기 위해 클라이언트로 다운로드된다.
- 따라서 `FancyText` 컴포넌트는 서버 및 클라이언트에서 모두 평가된다. 서버 컴포넌트로 사용될 때는 서버에서 실행되고, 클라이언트 컴포넌트로 사용될 때는 클라이언트에서 렌더링된다.


#### Copyright
그럼 `Copyright`는 `InspirationGenerator`의 자식으로 렌더되는데 왜 서버 컴포넌트일까?

- 'use client'는 렌더 트리가 아닌 **모듈 의존성 트리**에서 서버 코드와 클라이언트 코드 간의 경계를 정의한다.

<img width="573" alt="image" src="https://github.com/Jungle-JavaScript-Study/react-docs-study/assets/82787570/ed510508-5f62-420b-b4e9-7a463600be48">

- 클라이언트 컴포넌트는 JSX를 props로 전달할 수 있기 때문에 클라이언트 컴포넌트가 서버 컴포넌트를 렌더링할 수 있다. 
- 이 경우 `InspirationGenerator`는 `Copyright`를 자식으로 받지만 직접 import하거나 호출하지 않는다. 따라서 실제로 `Copyright` 컴포넌트는 `InspirationGenerator`가 렌더링되기 전에 완전히 실행된다.

즉, 부모-자식 렌더 관계가 컴포넌트 간 동일한 렌더 환경을 보장하지 않는다. 부모와 자식 컴포넌트 간의 렌더링 환경이 서로 다를 수 있다.

## 언제 쓰지?
어떤 것을 클라이언트 컴포넌트로 쓸 지 판단하기 위해 서버 컴포넌트의 장점과 제한 사항을 살펴보자.

### 서버 컴포넌트의 장점
- 클라이언트로 전송되고 실행되는 코드 양을 줄일 수 있다. 클라이언트 모듈만이 클라이언트에서 번들링되고 평가된다.
- 서버에서 실행되기 때문에 로컬 파일 시스템에 액세스할 수 있고, 데이터 가져오기 및 네트워크 요청에 대한 대기 시간(latency)가 낮을 수 있다.

### 서버 컴포넌트의 제한 사항
- onClick 이벤트 핸들러 같은 상호작용을 지원할 수 없다.
- 대부분의 hook을 사용할 수 없다.
- 서버 컴포넌트 렌더링의 결과는 본질적으로 클라이언트가 렌더링할 컴포넌트 목록이다. 서버 컴포넌트는 렌더링 후 메모리에 지속되지 않으며 자체 상태를 가질 수 없다.

## 서버 컴포넌트가 반환하는 직렬화 가능한 유형
- 서버 컴포넌트와 클라이언트 컴포넌트는 다른 환경에서 렌더링되므로 서버 컴포넌트에서 클라이언트 컴포넌트로 데이터를 전달할 때는 추가적인 고려가 필요하다.
- 서버 컴포넌트에서 클라이언트 컴포넌트로 전달되는 prop 값을 직렬화가 가능해야 한다.

### 가능
- 기본 데이터 유형 (Primitives)
string, number, bigint, boolean, undefined, null, symbol
- 직렬화 가능한 값들을 포함한 Iterables 객체
String, Array, Map, Set, TypedArray, ArrayBuffer
- Date
- 직렬화 가능한 속성을 가진 일반 객체
- 서버 액션으로 정의된 함수
- 클라이언트 또는 서버 컴포넌트 요소 (JSX)
- Promises

### 불가능
- 클라이언트에서 export되지 않거나 'use server'로 표시되지 않은 함수
- Class
- 클래스의 인스턴스나 null 프로토타입을 가진 객체
- 글로벌로 등록되지 않은 심볼

# 사용법

## 상호작용과 state를 포함한 빌드

- 아래의 `Counter`는 useState 훅과 이벤트 핸들러가 모두 필요하므로 클라이언트 컴포넌트여야 한다.

```jsx
'use client';

import { useState } from 'react';

export default function Counter({initialValue = 0}) {
  const [countValue, setCountValue] = useState(initialValue);
  const increment = () => setCountValue(countValue + 1);
  const decrement = () => setCountValue(countValue - 1);
  return (
    <>
      <h2>Count Value: {countValue}</h2>
      <button onClick={increment}>+1</button>
      <button onClick={decrement}>-1</button>
    </>
  );
}
```

- `CounterContainer`는 훅과 이벤트 핸들러를 사용하지 않으므로 'use client'가 필요하지 않다.
- 게다가 `CounterContainer`는 서버의 로컬 파일 시스템에서 읽기 때문에 서버 컴포넌트여야만 한다.

```jsx
import { readFile } from 'node:fs/promises';
import Counter from './Counter';

export default async function CounterContainer() {
  const initialValue = await readFile('/path/to/counter_value');
  return <Counter initialValue={initialValue} />
}
```

### HTML 출력이 큰 경우
서버 또는 클라이언트 전용 기능을 사용하지 않고, 렌더링 위치와 관계없이 무관한 범용적인 컴포넌트로 만들 수 있다.
위에서 본 `FancyText` 컴포넌트처럼 사용에 따라 서버 컴포넌트 혹은 클라이언트 컴포넌트로 결정된다.

- 컴포넌트의 HTML 출력이 소스 코드(의존성 포함)에 비해 큰 경우, 항상 클라이언트 컴포넌트로 강제하는 것이 더 효율적일 수 있다.
- ex) 긴 SVG 경로 문자열을 반환하는 컴포넌트
- 서버 측에서 불필요한 출력 데이터 양을 줄이고 브라우저에서 렌더링할 수 있다.

## 클라이언트 API 사용
- 클라이언트 전용 API를 사용할 때
- ex) Web Storage, 오디오/비디오 처리, 디바이스 하드웨어와 같은 브라우저 API

캔버스 요소를 조작하기 위해 DOM API를 사용하는 예시

```jsx
'use client';

import {useRef, useEffect} from 'react';

export default function Circle() {
  const ref = useRef(null);
  useLayoutEffect(() => {
    const canvas = ref.current;
    const context = canvas.getContext('2d');
    context.reset();
    context.beginPath();
    context.arc(100, 75, 50, 0, 2 * Math.PI);
    context.stroke();
  });
  return <canvas ref={ref} />;
}
```

## Third-party 라이브러리
아래의 React API 중 하나라도 사용하는 서드 파티 컴포넌트는 클라이언트에서 실행되어야 한다.

- createContext
- use와 useId를 제외한 react 및 react-dom Hooks
- forwardRef
- memo
- startTransition
- 클라이언트 API를 사용하는 경우(ex. DOM 삽입 또는 기본 플랫폼 뷰 사용)

이런 라이브러리가 서버 컴포넌트와 호환되도록 업데이트된 경우, 이미 'use client'가 포함되어 있어서 서버 컴포넌트에서 직접 사용할 수 있다.
그러나 라이브러리가 업데이트되지 않았거나 이벤트 핸들러와 같은 클라이언트에서만 지정할 수 있는 Props가 필요한 경우, 해당 서드파티 클라이언트 컴포넌트와 서버 컴포넌트 사이에 자체적으로 클라이언트 컴포넌트 파일을 추가해야 할 수 있다.
