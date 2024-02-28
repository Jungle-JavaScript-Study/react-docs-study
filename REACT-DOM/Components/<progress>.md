# `<progress>`
- 진행률을 보여주는 브라우저 내장 컴포넌트
- 모든 일반 엘리먼트 props에 더해 `max`, `value` 지원
  - `max`: 최댓값을 지정하는 숫자, 지정하지 않으면 기본적으로 1로 간주
  - `value`: 얼만큼 완료되었는지를 나타내는 숫자 또는 `null`. `0`~`max` 사이의 값이나, 불확정 상태일 때는 `null`을 사용

## 진행률 표시기 제어
```jsx
export default function App() {
  return (
    <>
      <progress value={0} />
      <progress value={0.5} />
      <progress value={0.7} />
      <progress value={75} max={100} />
      <progress value={1} />
      <progress value={null} />
    </>
  );
}
```
<img width="193" alt="image" src="https://github.com/Jungle-JavaScript-Study/react-docs-study/assets/70076564/a393122e-e61f-4c55-a755-00c788d9c396">
