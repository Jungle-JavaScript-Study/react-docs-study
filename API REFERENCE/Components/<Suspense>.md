`<Suspense>`를 사용하면 자식 컴포넌트의 로딩이 완료될 때까지 fallback을 표시할 수 있다.

```jsx
<Suspense fallback={<Loading />}>
  <SomeComponent />
</Suspense>
```

# 1. Reference
## 1-1. Props
### 1) children
- 렌더링하려는 실제 UI
- 렌더링하는 동안 자식 컴포넌트인 children이 중단되면 Suspense 경계가 대체로 지정된 컨텐츠인 fallback을 렌더링한다.
- Suspense 경계(Suspense boundary): Suspense 컴포넌트를 사용하여 정의하는 영역

### 2) fallback
- 대체 UI
- children이 일시 중단되면 자동으로 fallback으로 전환되고, 데이터가 준비되면 다시 children으로 전환된다.
- 렌더링 중에 fallback이 일시 중단되면 가장 가까운 상위 Suspense 경계가 활성화된다.

## 1-2. 주의사항
### 1) 일시 중단된 렌더링의 state
- 처음 마운트하기 전에 일시 중단된 렌더링의 state는 보존되지 않는다.
- 데이터가 준비되면 일시 중단된 트리의 렌더링을 처음부터 다시 그린다.

### 2) 다시 일시 중단된 경우
- Suspense가 트리에 대한 컨텐츠를 표시하고 있다가 다시 일시 중단된 경우, 그 원인이 된 업데이트가 startTransition이나 useDeferredValue로 인한 것이 아니라면 fallback이 다시 표시된다.

### 3) useLayoutEffect
- 다시 일시 중단되어 이미 표시된 컨텐츠를 숨겨야 하는 경우, 컨텐츠 트리에서 layout Effect를 정리(클린업)한다. 
- 컨텐츠가 다시 표시될 준비가 되면 layout Effect를 다시 실행한다. 
- 컨텐츠가 숨겨져 있는 동안에는 DOM 레이아웃을 측정하는 Effect가 작동하지 않게 하기 위한 것!

### 4) 내부 최적화
- 리액트는 '스트리밍 서버 렌더링'과 '선택적 하이드레이션' 같은 내부 최적화 기능을 Suspense와 통합하여 제공하고 있다.

# 2. Usage
## 2-1. 로딩하는 동안 Fallback 표시하기

```jsx
<Suspense fallback={<Loading />}>
  <Albums />
</Suspense>
```

- 애플리케이션의 어떤 부분이든 Suspense로 감싸면 끝!
- 감싸진 자식에게 필요한 모든 코드와 데이터가 로드될 때까지 fallback이 표시된다.

## 2-2. Suspense가 활성화되는 데이터 소스
- 아무때나 다 되는 건 아님
- Suspense를 도입한 데이터 소스에서만 쓸 수 있다.

👇 가능한 것들
- Relay 및 Next.js와 같은 Suspense를 지원하는 프레임워크를 사용한 데이터 페칭
- 코드 스플리팅을 위해 React.lazy를 사용하여 컴포넌트 코드를 지연 로딩하는 경우

❌ 얘넨 안됨
- Effect나 이벤트 핸들러 내에서 이루어지는 데이터 페칭은 감지 못함

프레임워크를 사용하지 않고 데이터 페칭에 Suspense를 도입하는 방법은 아직은 없다.
Suspense와 데이터 소스를 통합하기 위한 React 공식 API를 출시할 예정이라고 한다. 👍

## 2-3. 컨텐츠를 한 번에 드러내기
- 기본적으로 Suspense 내부의 전체 트리는 단일 단위로 취급된다.
- 아래 컴포넌트 중 하나만 중단되어도 모든 컴포넌트가 함께 fallback으로 대체된다.

```jsx
<Suspense fallback={<Loading />}>
  <Biography />
  <Panel>
    <Albums />
  </Panel>
</Suspense>
```

## 2-4. 중첩된 컨텐츠가 로드될 때 표시하기
- 컴포넌트가 일시 중단되면 가장 가까운 상위 Suspense 컴포넌트가 폴백을 표시하므로
- 여러 Suspense를 중첩해서 로딩 시퀀스를 만들 수 있다.

```jsx
<Suspense fallback={<BigSpinner />}>
  <Biography />
  <Suspense fallback={<AlbumsGlimmer />}>
    <Panel>
      <Albums />
    </Panel>
  </Suspense>
</Suspense>
```

렌더링 순서: BigSpinner -> AlbumsGlimmer -> 전체 렌더링

## 2-5. 새 컨텐츠가 로드되는 동안 이전 컨텐츠 보여주기

- useDeferredValue 훅을 사용해서 쿼리의 지연된 버전을 전달한다.

```jsx
export default function App() {
  const deferredQuery = useDeferredValue(query);
  return (
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={deferredQuery} />
      </Suspense>
  );
}
```

## 2-6. 이미 표시된 컨텐츠가 숨겨지지 않도록 방지하기

- 컴포넌트가 일시 중단될 때 상위 Suspense 경계가 폴백으로 전환되는데, 이미 컨텐츠가 표시되고 있는 경우에는 그냥 두는 것이 자연스럽겠지!
- 이를 방지하려면 startTransition을 사용해서 state 업데이트를 트랜지션으로 표시한다.
- 참고로, Suspense가 도입된 라우터는 기본적으로 탐색 업데이트를 트랜지션으로 감싸고 있다.

## 2-7. 트랜지션이 발생하고 있음을 나타내기
- 트랜지션중인지 알고싶으면 isPending과 startTransition을 반환하는 useTransition을 쓰면 됨

```
const [isPending, startTransition] = useTransition();
```

## 2-8. 재탐색시 Suspense 경계 재설정하기

- 트랜지션 중에는 이미 표시된 컨텐츠를 숨기지 않는다.
- 하지만 다른 데이터로 로딩하는 상황이라면 이전 컨텐츠는 가리고 fallback이 보여져야겠지!
- 이럴 때 key를 지정하면 서로 다른 컴포넌트로 취급해서 Suspense 경계가 재설정된다.

```jsx
<ProfilePage key={queryParams.id} />
```

## 2-9. 서버 오류 및 서버 전용 컨텐츠에 대한 fallback 제공
- 스트리밍 서버 렌더링 API를 사용하는 경우, Suspense를 활용해서 서버에서 발생하는 오류를 처리할 수 있다.
- 서버에서 에러를 발생시켜도 서버 렌더링이 중단되지 않고, 가장 가까운 Suspense 컴포넌트를 찾아서 생성된 서버 HTML에 그 fallback을 포함시킨다.
- 클라이언트에서는 동일한 컴포넌트를 다시 렌더링한다.
- 이를 이용해서 일부 컴포넌트를 서버에서 렌더링하지 않도록 선택할 수 있다.
- 서버 환경에서 오류를 발생시키고 해당 컴포넌트를 Suspense 경계로 감싸서 해당 HTML을 fallback으로 대체하면 된다.

일부 컴포넌트를 서버에서 렌더링하지 않도록 하는 예시 👇

```jsx
<Suspense fallback={<Loading />}>
  <Chat />
</Suspense>

function Chat() {
  if (typeof window === 'undefined') {
    throw Error('Chat should only render on the client.');
  }
  // ...
}
```

Chat 컴포넌트가 서버에서 렌더링되려고 하면 오류가 발생해서 fallback된다.
따라서,
Chat 컴포넌트는 클라이언트 사이드에서만 렌더링 되고
서버 사이드에서는 Loading이 렌더된다.

이런 방식으로
서버 사이드와 클라이언트 사이드에서 각각 적절한 UI를 렌더링하면서, 클라이언트 전용 컴포넌트가 서버 사이드에서 렌더링되는 것을 방지할 수 있다.
