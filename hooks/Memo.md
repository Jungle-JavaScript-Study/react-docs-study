# memo
memo를 사용하면 props가 변경되지 않은 경우에 컴포넌트의 재렌더링을 건너뛸 수 있다.

# 1. Reference

```js
import { memo } from 'react';

const SomeComponent = memo(function SomeComponent(props) {
  // ...
});
```

컴포넌트를 memo로 감싸서 해당 컴포넌트의 **memoized**된 버전을 얻을 수 있다.
memoized 컴포넌트 버전은 부모가 다시 렌더링되더라도 props가 변경되지 않으면 리렌더링되지 않는다.
🚨 이것은 보장된 것은 아니다! React가 리렌더링할 수도 있다!

## 1-1. Parameters

### Component
- 메모화 하려는 컴포넌트
- memo는 이 컴포넌트를 수정하는 것은 아니고, 새롭게 메모화된 컴포넌트를 반환한다.
- 함수와 forwardRef 컴포넌트를 포함한 모든 유효한 React 컴포넌트가 허용된다.

### arePropsEqual (optional)
- 이 함수는 컴포넌트의 이전 Props와 새 Props를 인자로 받는다.
- 이전과 새 props가 동일한지 여부를 boolean으로 반환한다.
- 보통은 이 함수를 지정하지 않고, React가 Object.is를 사용해서 Props를 비교한다.

## 1-2. Returns
- 새로운 React 컴포넌트를 반환한다.
- 인자로 받은 원본 컴포넌트와 동일하게 동작하지만, 부모가 다시 렌더링 되더라도 props가 변경되지 않으면 항상 리렌더링되지는 않는다.

# 2. 사용법
## 2-1. props가 변경되지 않았을 때 리렌더링 건너뛰기
- 위에서 계속 했던 얘기..
- memo를 사용해서 부모가 리렌더되어도 리렌더링되지 않는 컴포넌트를 memoized되었다고 한다.
- React 컴포넌트는 항상 순수한 렌더링 로직을 가져야 한다.
즉, props, state, context가 변경되지 않았다면 동일한 출력을 반환해야 한다.
memo를 사용함으로써 컴포넌트가 이 요구사항을 준수한다고 React에 알려 props가 변경되지 않는 한 React는 리렌더링하지 않는다.
- 당연히.. memo를 사용해도 컴포넌트의 자체 state나 사용중인 context가 변경되면 컴포넌트는 리렌더된다.

🚨 memo는 성능 최적화를 위해서만 사용해야 한다.
memo 없이 코드가 작동하지 않으면 근본적인 문제를 찾아서 해결하자.!!!

## 모든 곳에 memo를 추가할까요?
그럴리가

### 어떤 상호작용을 하나요?
대부분의 인터랙션이 투박한 경우(페이지 or 전체 섹션 교체 등)에는 일반적으로 memoization이 불필요하다.
반면, 그림판처럼 대부분의 상호작용이 미세하다면 (도형 이동 등) memoization이 매우 유용하다.

### 언제 쓰는 게 좋은가요?
memo를 이용한 최적화가 가치 있는 경우는
컴포넌트가 정확히 동일한 props로 자주 리렌더링되고 그 리렌더링 로직이 비용이 많이 드는 경우이다.

컴포넌트가 리렌더링될 때 눈에 띄는 지연이 없다면 memo가 필요하지 않다.

#### useMemo, useCallback과 함께 사용해야 하는 경우
또한 컴포넌트가 전달 받는 props가 매번 다르다면 (ex 렌더링 중에 정의된 객체나 일반 함수가 전달되는 경우), memo는 완전히 무용지물이다..
따라서 종종 useMemo와 useCallback을 memo와 함께 사용해야 한다.

### 이점이 없는 경우
위에서 언급하지 않은 경우에는 memo의 이점이 없다.
종종 개별 사례에 대해 생각하지 않고 가능한 한 많이 메모하는 방식을 사용하기도 하는데, 그렇게 하면 코드의 가독성이 떨어진다.
또한 모든 memoization이 효과적이지는 않다.
항상 새로운 값이 단 하나만 있어도 컴포넌트 전체의 메모이제이션을 깨뜨릴 수 있다.

### memoization이 불필요해지는 원칙
1. 다른 컴포넌트를 시각적으로 감싸는 컴포넌트는 [JSX를 자식으로 받게](https://react-ko.dev/learn/passing-props-to-a-component#passing-jsx-as-children) 한다.
이렇게 하면 wrapper 컴포넌트가 자체 상태를 업데이트해도 자식은 리렌더되지 않는다.

2. 로컬 state를 선호하고, 필요 이상으로 state를 끌어올리지 않는다.

3. 렌더링 로직을 순수하게 유지한다. 컴포넌트를 리렌더링하면 문제가 발생하거나 시각적 artifact(부정확하거나 원치 않는 결과)가 생성되는 것은 버그임.. memoization이 아니라 버그를 수정하자.

4. state를 업데이트하는 불필요한 Effect를 피한다.
React 앱의 대부분의 성능 문제는 컴포넌트를 반복해서 렌더링하게 만드는 Effect에서 비롯된 업데이트 체인에 의해 발생한다.

5. Effect에서 불필요한 의존성을 제거한다.
memoization 대신 일부 객체나 함수를 효과 내부나 컴포넌트 외부로 이동하는 것이 memoization보다 더 간단할 때가 많다.

이 원칙들을 따라도 특정 인터랙션이 여전히 느리게 느껴진다면 [React Developer Tools의 profiler](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html)를 사용해서 어떤 컴포넌트가 memoization으로 가장 많은 이득을 볼 수 있는지 확인해서 추가한다.
이 원칙들은 컴포넌트를 디버깅하고 이해하기 쉽게 만들어주므로, 어떤 경우에도 따르는 것이 좋다.

장기적으로 React는 이 문제를 일단락하기 위해 자동으로 세밀한 memoization을 하는 방법을 연구하고 있다고 한다.

#### [📺 React without memo](https://www.youtube.com/watch?v=lGEMwh32soc)
리액트에서 연구중인, 컴파일러가 auto-memoizing을 해주는 React Forget 프로젝트에 대해서 알아보자!

이렇게 memoization을 위해 코드를 추가해야 하고 읽기도 어려워진다.
또한 데이터의 의존성 등을 고려하는 것도 복잡하다.

![](https://velog.velcdn.com/images/e_juhee/post/30728f58-9dc6-4798-94ec-ceff57f398e5/image.png)

memoization 관련 코드를 제거하면 이렇게나 간결하다.! 👇

![](https://velog.velcdn.com/images/e_juhee/post/dfe740ea-539e-41d4-a0f3-b869cbcc6398/image.png)

넘나 복잡하니 memoization을 자동으로 하는 방법을 생각해보자.

이 컴포넌트는 단순히 함수일 뿐이다.
의존성을 시각화한 의존성 그래프는 아래 그림과 같다.

![](https://velog.velcdn.com/images/e_juhee/post/53af9e2b-a3e9-48e0-8f85-210c96fc16cd/image.png)

값이 변경되었는 지 확인할 수 있는 변수가 있다고 상상하고,
이 변수를 통해서 값이 변경되었으면 새로운 값을, 변경되지 않았으면 이전에 캐시된 값을 사용하는 것이다.

![](https://velog.velcdn.com/images/e_juhee/post/35affd02-99bb-4f94-90f2-312387576063/image.png)

위 아이디어를 코드 전체에 적용하면 이렇게 된다.

![](https://velog.velcdn.com/images/e_juhee/post/d07f8928-9276-4cb0-bb8b-46fb8d46bb49/image.png)

이런 일들을 리액트에 컴파일러가 해준다면! 짱 편하겠지
현재 데모는 아래와 같은 모습이다.

![](https://velog.velcdn.com/images/e_juhee/post/9db732c2-352c-421f-807a-c76a81f6275d/image.png)

이런식으로 Memoization을 자동화해주는 것을 연구하고 있다. 멋쪄🩵

## 2-2. memoized된 컴포넌트를 state로 업데이트하기

컴포넌트가 memoized된 경우에도 state가 변경되면 리렌더된다.
memoization은 부모로부터 전달받는 props와만 관련이 있다.

이때 현재 값과 동일한 값으로 state를 설정하면 React는 memo 없이도 컴포넌트의 리렌더링을 건너뛴다. (당연한 소리...)
컴포넌트 함수가 추가로 호출될 수는 있지만, 결과는 무시된다.

## 2-3. context를 사용하여 memoized된 컴포넌트 업데이트하기
사용 중인 컨텍스트가 변경되면 리렌더된다.

컨텍스트의 일부가 변경될 때만 컴포넌트가 재렌더링되도록 하려면, 컴포넌트를 두 부분으로 나눠서
외부 컴포넌트에서 필요한 컨텍스트들을 읽고, 그것을 Props로 memoized된 자식 컴포넌트에 전달하자!

컨텍스트의 변화가 자주 발생하고, 그 변화가 컴포넌트의 특정 부분에만 영향을 미칠 때 유용한 방법이다.

## 2-4. props 변경 최소화하기

### 객체 props (useMemo 활용)
memo를 사용하면 이전 prop과 현재 prop을 얕은 비교로 비교해서 다를 경우에 리렌더한다.
즉, 모든 prop을 이전 값과 `Object.is`로 비교한다는 것이다.

```js
Object.is(3, 3) // true
Object.is({}, {}) // false
```

memo의 효과를 극대화하려면 Props가 변경되는 횟수를 최소화해야 한다.
prop이 객체라면 매번 새로 생성하지 않도록 부모 컴포넌트에서 useMemo를 사용하는 것이 좋다.

### 객체 대신 값 전달
props 변경을 최소화하는 더 나은 방법은 컴포넌트가 필요한 최소한의 정보만 Props로 받도록 하는 것이다.
예를 들어, 전체 객체 대신 개별 값들을 전달한다.

심지어 개별 값들도 덜 자주 변경되는 값들로 변환할 수 있다.
예를 들어, 값 대신 값의 존재 여부를 전달할 수 있다.

### 함수 props (useCallback 활용)
memoized된 컴포넌트에 함수를 전달해야 할 때는, 해당 함수를 컴포넌트 외부에서 선언하여 변경되지 않도록 하거나, useCallback을 사용해 정의를 캐시할 수 있다.


## 2-5. 사용자 정의 비교 함수 지정하기

드문 경우, memoized된 컴포넌트의 props 변경을 최소화하는 것이 불가능할 수 있다.
이런 경우에는 커스텀 비교 함수를 memo의 두 번째 인자로 제공할 수 있다.

이 함수는 새로운 Props가 이전 Props와 동일한 출력을 생성할 경우에만 true를 반환하게 해야 한다.

이 방법을 사용할 때는 브라우저 개발자 도구의 성능 패널을 사용해서 비교 함수가 컴포넌트를 재렌더링하는 것보다 실제로 더 빠른지 확인하자. (예상치 못한 결과를 볼 수도 있다.)
+) 성능 측정을 할 때는 React가 프로덕션 모드에서 실행되고 있는지 확인하자.

비교 로직을 효과적으로 구현하는 ㄱ서이 중요하다.

### Pitfall

#### 모든 Props를 비교할 것!

커스텀 비교 함수를 쓸 때는, 함수를 포함하여 모든 Props를 비교해야 한다.
함수는 종종 부모 컴포넌트의 Props와 state를 클로저로 갖는다..!

`oldProps.onClick !== newProps.onClick` 일 때 true를 반환하면
컴포넌트는 이전 렌더링에서의 props와 state를 onClick 핸들러 내에서 계속 인식하여 매우 혼란스러운 버그를 야기할 수 있다.

#### 깊은 비교 ㄴㄴ!

비교 함수 내에서 깊은 비교를 수행하는 것은 피해야 한다.
작업 중인 데이터 구조가 알려진 제한된 깊이를 가지고 있다는 것을 100% 확신하는 경우가 아니라면 깊은 비교는 매우 느려질 수 있고, 누군가 나중에 데이터 구조를 변경하면 앱이 몇 초 동안 멈출 수도 있다.

# 3. Troubleshooting

## 3-1. prop이 객체, 배열, 또는 함수인 props로 인해 리렌더되는 경우

React는 **얕은 비교**로 props를 비교한다.
즉 각 새 prop이 이전 prop과 참조가 동일한지(reference-equal) 고려한다.

부모 컴포넌트가 재렌더링될 때마다 새로운 객체나 배열을 생성하면, 개별 요소가 동일하더라도 React는 변경되었다고 간주한다.

마찬가지로, 부모 컴포넌트를 렌더링할 때 새로운 함수를 생성하면
함수의 정의가 동일해도 변경되었다고 간주한다.

이를 방지하기 위해 props를 단순화하거나 부모 컴포넌트에서 Props를 memoize하자!
