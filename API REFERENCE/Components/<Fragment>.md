# `<Fragment>` (<>...</>)
- 단일 엘리먼트가 필요할 때 `<Fragment>`로 묶어 그룹화
- 실제 DOM에는 아무런 영향 없음(그룹화하지 않은 것과 동일)
- `<></>` = `<Fragment></Fragment>`
- `<Fragment>`로 명시적으로 선언한 Fragment에는 선택적으로 key를 추가할 수 있음(`<></>`는 불가)
- 한 단계 깊이의 Child 컴포넌트를 렌더링할 때는 state가 초기화되지 않음
  - `<><Child /></>`를 렌더링하는 컴포넌트는 실제 렌더링 시 `<Child />`로 전환되어 렌더링되는데, 이렇게 한 단계 깊이를 렌더링할때는 state가 재설정되지 않음

## 복합 엘리먼트 리턴하기
- 단일 엘리먼트가 들어가는 위치에 복합 엘리먼트를 넣고자 할 때 Fragment 혹은 `<>...</>`로 그룹화할 수 있음<br>
→ 컴포넌트는 하나의 엘리먼트만 리턴할 수 있지만, Fragment로 묶으면 여러 엘리먼트를 그룹화해 단일 그룹 리턴 가능
- DOM 엘리먼트 같은 다른 컨테이너로 묶는 것과 달리 레이아웃이나 스타일에 영향을 주지 않음(DOM에 추가되는 순간 Fragment는 사라짐!)
- Fragment에 key를 전달해야 하는 경우가 아니라면 일반적으로 `<></>` 선호(<Fragment/>는 React에서 import 해야 함)

```javascript
// <h1>과 <article>이 wrapper 노드 없이 형제로 표시됨
export default function Blog() {
  return (
    <>
      <Post title="An update" body="It's been a while since I posted..." />
      <Post title="My new blog" body="I am starting a new blog!" />
    </>
  )
}

function Post({ title, body }) {
  return (
    <>
      <PostTitle title={title} />
      <PostBody body={body} />
    </>
  );
}

function PostTitle({ title }) {
  return <h1>{title}</h1>
}

function PostBody({ body }) {
  return (
    <article>
      <p>{body}</p>
    </article>
  );
}
```
![image](https://github.com/Jungle-JavaScript-Study/react-docs-study/assets/70076564/4d776afc-df56-4202-9a99-c8a37fed7291)


## 변수에 복합 엘리먼트를 할당하기
Fragment도 변수에 할당하거나 props로 전달하는 등 일반 엘리먼트와 똑같이 사용할 수 있음
```javascript
  const buttons = (
    <>
      <OKButton />
      <CancelButton />
    </>
  );
```

## 텍스트와 엘리먼트를 그룹화하기
텍스트를 컴포넌트와 그룹화할 때 사용
```javascript
function DateRangePicker({ start, end }) {
  return (
    <>
      From
      <DatePicker date={start} />
      to
      <DatePicker date={end} />
    </>
  );
}
```

## Fragment 목록 렌더링하기
반복문에서 복합 엘리먼트를 렌더링하는 경우에는 각 엘리먼트에 key를 할당해야 하므로 `<></>` 말고 명시적인 `<Fragment>`를 사용해야 함
```javascript
function Blog() {
  return posts.map(post =>
    <Fragment key={post.id}>
      <PostTitle title={post.title} />
      <PostBody body={post.body} />
    </Fragment>
  );
}
```
