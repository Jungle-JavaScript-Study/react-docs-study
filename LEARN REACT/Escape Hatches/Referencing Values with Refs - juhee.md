### Escape Hatch란?
프로그래밍 라이브러리나 프레임워크는 특정 작업을 쉽게 만들어주는 추상화를 제공해주지만
때로는 이 추상화의 한계에 도달할 수 있다.

이럴 때 그 한계를 넘어서 원하는 작업을 할 수 있게 해주는 기능이나 방법을 **Escape Hatch**라고 한다.
이번 섹션에서는 React의 일반적인 작동 방식에서 약간 벗어나, 특별한 상황 or 요구사항을 다루기 위한 기능들을 알아보자!

_이 아래는 공식 문서의 설명 👇_

컴포넌트 중 일부는 React 외부의 시스템을 제어하고 동기화해야 할 수 있다.
- 예를 들면, 브라우저 API를 사용해 input에 포커스를 주거나
- React를 사용하지 않고 구현된 비디오 플레이어를 재생 or 일시정지시키거나
- 원격 서버에 연결해 메시지를 수신할 수 있다.

이 장에서는 React 밖으로 나가서 외부 시스템에 연결할 수 있게 해주는 escape hatches에 대해 배운다.
But, 앱의 대부분의 로직과 데이터 흐름은 이런 기능에 의존해서는 안된다!

<br/>

# 1. ref로 값 참조하기
리렌더링 없이 정보를 "기억하는" 방법

컴포넌트가 특정 정보를 기억하도록 하고 싶지만
그 정보가 새로운 렌더링을 촉발하지 않게 하려면 `ref`를 사용하면 된다!

## 1-1. ref 추가하기
React에서 useRef 훅을 가져와서 호출하고 참조할 초기값을 인자로 전달한다.

```js
import { useRef } from 'react';

const ref = useRef(0);
```

useRef가 반환하는 객체

```js
{ 
  current: 0 // The value you passed to useRef
}
```

- `ref.current` 속성을 통해 ref의 현재 값에 접근할 수 있다.
- 이것은 React가 추적하지 않는 컴포넌트의 비밀 주머니 ㅋㅋㅋ 와 같다..
그래서 ref가 React의 단방향 데이터 흐름에서 escape hatch가 되는 것이당~ (이에 대한 설명은 이어서 자세히 나온다.)

## 1-2. State vs Ref
- ref의 current 속성은 state와 달리 읽고 수정할 수 있는 일반 JS 객체이다.
- ref가 변경되어도 리렌더링 되지 않는다!
- 리렌더링되어도 값이 유지되는 것은 둘 다 동일하다.

|| refs | state |
|-|-|-|
| **사용법** | `useRef()`는 `{ current: initialValue }`을 반환 | `useState()`는 state 변수의 현재값과 state 설정자함수([value, setValue])를 반환 |
| **리렌더링** | 변경 시 리렌더링을 촉발하지 않음                          | 변경 시 리렌더링을 촉발함                                        |
| **변경 가능성** | Mutable — 렌더링 프로세스 외부에서 current 값을 수정하고 업데이트할 수 있음 | Immutable — setState를 사용해서 state 변수를 수정해 리렌더링을 대기열에 추가해야함 |
| **값 접근** | 렌더링 중에는 current 값을 읽거나 쓰지 않아야 함            | 언제든지 state를 읽을 수 있음. 각 렌더링에는 변경되지 않는 자체 state snapshot이 있음         |


👉 렌더링에 보여줘야 하는 값은 state로,
렌더링에 사용되지 않는 값은(값이 바뀌어도 리렌더링될 필요 없는 경우) ref로 보관하는 것이 더 효율적일 수 있다.

## 1-3. ref의 작동 방식

useRef는 useState로 구현할 수 있다.

```js
// Inside of React
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

useRef는 항상 동일한 객체를 반환하기 때문에 state 설정자는 사용되지 않는다.
즉, ref는 setter가 없는 일반 state 변수로 생각할 수 있다.
하지만 useRef를 기본적으로 제공해주므로, 이것을 쓰는 것이 바람직 ^ㅁ^

<br/>

## 1-4. ref는 언제 쓰나요

일반적으로 React의 범위를 벗어나서 외부 API와 통신할 때 사용한다.
주로 컴포넌트의 모양에 영향을 주지 않는 브라우저 API 등과 통신할 때 사용한다.

- 타임아웃 ID 저장
- DOM 요소 저장 or 조작
- JSX 계산에 필요하지 않은 다른 객체들 저장할 때 쓴다.

컴포넌트에 일부 값을 저장해야 하지만 렌더링 로직에는 영향을 미치지 않는 경우 ref를 쓰자.
이러한 상황에서 React의 일반적인 데이터 흐름 내에서 정보를 추적하기보다는
ref를 사용해 정보를 직접적으로 관리하는 것이 더 효율적일 수 있다.

## 1-5. ref 모범 사례

아래의 원칙을 따라 컴포넌트의 예측 가능성을 높일 수 있다.

### 1) ref를 escape hatch로 취급하자!
앱의 주요 로직과 데이터 흐름이 ref에 크게 의존한다면, 접근 방식을 재고해볼 필요가 있다.

### 2) 렌더링 중에는 읽거나 쓰지 말자
React는 ref.current가 언제 변경되는지 알 수 없기 때문에, 렌더링 중에 ref.current를 읽는 것은 컴포넌트의 동작을 예측하기 어렵게 만든다.

예외) 아래처럼 첫 렌더링 중에만 ref를 설정하는 것은 ㄱㅊ

`if (!ref.current) ref.current = new Thing()`

state는 동기적으로 업데이트되지 않지만, ref는 값을 변이하면 즉시 변경된다.

```js
ref.current = 5;
console.log(ref.current); // 5
