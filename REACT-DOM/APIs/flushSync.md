# flushSync

💡 flushSync는 자주 사용되지 않으며, 앱 성능을 저하시킬 수 있다.

- flushSync는 제공된 콜백 내의 모든 업데이트를 동기적으로 즉시 처리하도록 React에 강제로 지시한다.
- React에 대기 중인 모든 작업을 처리하고 DOM을 동기적으로 업데이트하도록 강제한다.
- 대부분의 경우 flushSync는 피해야 한다. 최후의 수단으로 사용하자

```js
import { flushSync } from "react-dom";

flushSync(() => {
  setSomething(123);
});
```

## Parameters

### callback

- React는 즉시 이 콜백을 호출하고, 안에 포함된 모든 업데이트를 동기적으로 처리한다.
- 대기 중인 업데이트, Effects, 또는 Effects 내부의 업데이트를 처리할 수도 있다.
- flushSync 호출로 인해 업데이트가 중단되면, fallback이 다시 표시될 수 있다.

## Returns

- undefined

<br/>

# 주의사항

- 성능을 크게 저하시킬 수 있다.
- 대기 중인 Suspense 경계를 강제로 fallback으로 표시할 수 있다.
- 대기 중인 Effect들을 실행하고 반환하기 전에 그 안에 포함된 모든 업데이트를 동기적으로 적용할 수 있다.
- 필요한 경우 콜백 외부의 업데이트를 처리하여 콜백 내부의 업데이트를 처리할 수 있다.
  - 예를 들어, 클릭으로 인한 대기 중인 업데이트가 있는 경우, 콜백 내의 업데이트를 처리하기 전에 이러한 업데이트를 먼저 처리할 수 있다.

<br/>

# 사용법

## third-party 통합을 위한 업데이트 flush

- 브라우저 API나 UI 라이브러리와 같은 third-party 코드와 통합할 때, React가 업데이트를 강제로 처리하도록 할 필요가 있을 수 있다.

```js
flushSync(() => {
  setSomething(123);
});
// 이 줄이 실행될 때는 DOM이 업데이트 된 상태임
```

- 일부 브라우저 API는 콜백 함수 내부에서 발생하는 변경사항이 콜백 함수가 종료되기 전에 동기적으로 DOM에 반영되기를 요구한다.
  - 변경된 DOM을 기반으로 추가 작업을 수행할 수 있게 하기 위함!
- 대부분의 경우 React는 이를 자동으로 처리해서 브라우저 API와의 호환성을 유지하지만, 안 해줄 때도 있음..

예시를 보자 (`onbeforeprint` )

- 브라우저의 `onbeforeprint` API는 인쇄 대화 상자가 열리기 바로 전에 페이지를 변경한다.
  - 인쇄에 더 적합하게 문서를 표시하기 위한 사용자 정의 인쇄 스타일을 적용하는 데 유용함
- 아래 예시는 인쇄 대화 상자가 열리기 전에 isPrinting이 yes를 표시하도록 onbeforeprint 콜백 내에서 flushSync를 사용하여 즉시 React 상태를 DOM에 flush한다.
- 여기서 flushSync를 사용하지 않으면 yes가 아닌 no가 표시된다.
- React는 업데이트를 비동기적으로 일괄 처리하는데, state가 업데이트되기 전에 인쇄 대화 상자가 표시되기 때문!

```jsx
import { useState, useEffect } from "react";
import { flushSync } from "react-dom";

export default function PrintApp() {
  const [isPrinting, setIsPrinting] = useState(false);

  useEffect(() => {
    function handleBeforePrint() {
      flushSync(() => {
        setIsPrinting(true);
      });
    }

    function handleAfterPrint() {
      setIsPrinting(false);
    }

    window.addEventListener("beforeprint", handleBeforePrint);
    window.addEventListener("afterprint", handleAfterPrint);
    return () => {
      window.removeEventListener("beforeprint", handleBeforePrint);
      window.removeEventListener("afterprint", handleAfterPrint);
    };
  }, []);

  return (
    <>
      <h1>isPrinting: {isPrinting ? "yes" : "no"}</h1>
      <button onClick={() => window.print()}>Print</button>
    </>
  );
}
```
