# `<Profiler>`
- 컴포넌트 트리를 <Profiler>로 감싸 React 트리의 렌더링 성능을 측정
  ```javascript
  <Profiler id="App" onRender={onRender}>
    <App />
  </Profiler>
  ```
- `id`: 측정 중인 UI가 어느 부분인지 식별하는 문자열
- `onRender`: 프로파일된 트리 내의 컴포넌트가 업데이트 될 때마다 호출되는 콜백, 렌더링된 내용과 소요 시간에 대한 정보를 받음
- 프로파일링은 오버헤드가 약간 추가되므로 production 빌드에서는 기본적으로 비활성화되어 있음. production 환경을 프로파일링하고 싶으면 프로파일링이 활성화된 특수 production 빌드를 사용해야 함(CRA 최신 버전에서는 `npm run build -- --profile`)

### `onRender` 콜백
```javascript
function onRender(id, phase, actualDuration, baseDuration, startTime, commitTime) { }
```
- 리액트는 무엇이 렌더링되었는지에 대한 정보와 함께 `onRender` 콜백을 호출함
- `id`: 방금 커밋된 `<Profiler>` 트리의 `id` prop. 프로파일러를 여러개 사용할 경우 트리의 어느 부분이 커밋됐는지 식별할 수 있음
- `phase`: "mount", "update", "nested-update" 중 하나. 트리가 처음 마운트됐는지, props/state/훅의 변경으로 리렌더링됐는지 알 수 있음
- `actualDuration`: 현재 업데이트에 대해 `<Profiler>`와 하위 컴포넌트들을 렌더링하는데 걸린 밀리초. 하위 트리가 메모화(`memo`, `useMemo` 등)을 얼마나 잘 사용하는지를 나타냄. 이상적으로는 초기 마운트 이후에는 크게 감소해야 함(자손은 특정 props가 변경되는 경우에만 리렌더링이 필요하므로)
- `baseDuration`: 최적화 없이 전체 `<Profiler>` 하위 트리를 리렌더링하는 데 걸리는 밀리초의 추정값. 트리의 각 컴포넌트들의 가장 최근 렌더링 시간의 합계로 계산됨(=초기 마운트나 메모화가 없는 트리 등 최악의 렌더링 비용을 추정함) <br>→ `actualDuration`과 비교해서 메모화가 잘 작동하는지 확인 가능
- `startTime`: 리액트가 현재 업데이트에 대해 렌더링을 시작한 시점에 대한 타임스탬프(숫자)
- `commitTime`: 리액트가 현재 업데이트를 커밋한 시점에 대한 타임스탬프. 한 커밋의 모든 프로파일러들 사이에 공유되므로 프로파일러를 그룹화할 수 있음

## 프로그램적으로 렌더링 성능 측정하기
`<Profiler>`는 프로그램적으로 렌더링 성능을 측정하는 방법이고, 인터랙티브 프로파일러를 원한다면 리액트 개발자 도구의 프로파일러 탭을 사용

## 어플리케이션의 여러 부분 측정하기
- `<Profiler>` 컴포넌트를 여러개 사용하면 여러 부분을 측정할 수 있음
- 중첩도 가능
- `<Profiler>`가 가벼운 컴포넌트이긴 하지만, 추가될 때마다 약간의 CPU와 메모리 오버헤드가 발생하므로 필요할 때만 사용해야 함
