# `useLayoutEffect`
: 브라우저 렌더링 과정 중 리페인트 전에 실행되는 `useEffect`

- 컴포넌트가 DOM에 추가되기 전에 셋업함수 실행, DOM에서 제거되기 전에 클린업 함수 실행
- `useLayoutEffect`와 이 과정에서 예약된 모든 state 업데이트는 브라우저의 리페인트를 차단함 -> **속도 저하를 유발하므로 가급적이면 `useEffect 사용`**
- 이외에는 `useEffect`의 특징과 동일

## 리페인트 전에 레이아웃 측정하기
일반적인 상황에서는 렌더링 전에 화면에서의 위치나 크기를 알 필요가 없음
  
### Ex.) 툴팁
- 요소 옆에 툴팁을 표시할 때는 공간이 충분한 지 알아야 함
- 이 때의 렌더링 과정은 <툴팁 렌더링(일단 원하는 곳에 두기)-공간 측정 및 위치 결정-올바른 위치에 리렌더링> <br>
⇒ 사용자가 툴팁이 이동하는 것을 보지 못해야 하므로 `useLayoutEffect`를 사용해 리페인트 전에 레이아웃을 측정

```javascript
function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0); // 1. 아직 높이를 모르므로 0으로 초기화

  useLayoutEffect(() => { // 2. 높이를 측정하고 리렌더링을 촉발
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height); 
  }, []);

  // 3. 측정된 높이를 사용해 리렌더링
}
// 4. 리액트가 DOM을 업데이트하면 툴팁이 올바른 위치에 표시됨
```
※ [`getBoundingClientRect`](https://developer.mozilla.org/ko/docs/Web/API/Element/getBoundingClientRect): `padding`과 `border`를 포함한, 요소의 가장 바깥쪽 사각형인 `DOMRect` 객체를 리턴, 위치와 크기 등의 정보 제공

## Error: "`useLayoutEffect` does nothing on the server"
: 서버사이드 렌더링에서 발생하는 문제
- `useLayoutEffect`의 목적은 렌더링을 위해 레이아웃 정보를 사용할 수 있도록 하는 것<br>
  ⇒ 초기 콘텐츠 렌더링 → 리페인트 전 레이아웃 측정 → 측정한 레이아웃 정보를 사용해 최종 콘텐츠 렌더링
- 그런데 서버사이드 렌더링을 하는 경우에는 Javascript 코드가 로드되기 전에 서버에서 가져온 HTML로 초기 화면을 렌더링하는데, 서버에는 레이아웃 정보가 존재하지 않음
- 따라서 서버사이드 렌더링에서는 `useLayoutEffect`를 사용해도 초기 렌더링 후 위치가 이동함

### 해결
- `useLayoutEffect`를 `useEffect`로 변경: 페인트를 막지 않고 초기 렌더링 결과를 표시해도 괜찮다고 리액트에게 알려주기
- 컴포넌트를 클라이언트 전용으로 표시: 리액트는 서버 렌더링 중에 가장 가까운 `<Suspense>` 경계까지의 모든 콘텐츠를 로딩 폴백(스피너 등)으로 대체
- hydration 이후에만 `useLayoutEffect`를 사용해 컴포넌트를 렌더링: `isMounted`를 `false`로 초기화한 뒤 `useEffect` 내에서 `true`로 변경 → `return isMounted ? <RealContent /> : <FallbackContent />`로 렌더링하면 서버에서 hydration하는 동안에는 `useLayoutEffect`를 호출하지 않는 `FallbackContent`를 보여주고, 추후에 리액트가 클라이언트 전용으로 실행되고 `useLayoutEffect`를 포함하는 `RealContent`로 대체함
  <br>※ hydration: SSR할때, 동적 이벤트가 전혀 없는 초기 HTML에 JS 코드를 매칭시켜 동적인 페이지를 렌더링하는 기술(새로운 DOM을 생성하지 않고 기존 DOM Tree에서 해당되는 DOM 요소를 찾아 정해진 자바스크립트 속성(이벤트 리스너 등)들을 부착) 
- 레이아웃 측정 외에 외부와의 동기화를 위해 `useLayoutEffect`를 사용한 경우, 서버렌더링을 지원하는 `useSyncExternalStore` 고려
