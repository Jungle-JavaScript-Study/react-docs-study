# useInsertionEffect
: DOM 변이 전에 실행되는 `useEffect`

- CSS-in-JS 라이브러리리를 위한 훅이기 때문에 CSS-in-JS 라이브러리리를 사용하지 않는다면 `useEffect`나 `useLayoutEffect` 권장
- `useInsertionEffect`를 사용하면 DOM 변이 전에 스타일을 주입할 수 있음
  
  ```javascript
  import { useInsertionEffect } from 'react';

  // CSS-in-JS 라이브러리 내에서
  function useCSS(rule) {
    useInsertionEffect(() => {
      // <style> 태그를 여기에 주입
      if (!isInserted.has(rule)) {
      isInserted.add(rule);
      document.head.appendChild(getStyleForRule(rule));
      }
    });
    return rule;
  }
  ```
- `useInsertionEffect` 내부에서는 state를 업데이트할 수 없음
- `useInsertionEffect`가 실행되는 시점에는 ref가 아직 부착되지 않고 DOM이 업데이트되지 않음
- 이외에는 `useEffect`의 특징과 동일

## CSS-in-JS 라이브러리에서 동적 스타일 삽입하기 
CSS 파일을 따로 작성하지 않고 JavaScript 코드에서 바로 스타일을 작성하는 CSS-in-JS에는 세가지 접근 방식이 있고 일반적으로는 첫번째와 두번째를 조합하여 사용하길 권장
1. 컴파일러로 CSS 파일 정적 추출: 정적 스타일일 때 사용
2. 인라인 스타일링: 동적 스타일일 때 사용
3. 런타임에 `<style>` 태그 삽입: 브라우저가 스타일을 훨씬 자주 계산하게 하고, React 라이프사이클 중 잘못된 시점에 주입되면 속도가 매우 느려짐

- 세번째 방식의 두번째 문제를 `useInsertionEffect`로 해결 가능(`<style>` 태그의 런타입 주입은 권장하지 않으나, 꼭 해야겠다면 `useInsertionEffect`에서 하는 것이 중요)
- `useInsertionEffect`는 서버에서 실행되지 않음 → 서버에서 어떤 CSS 규칙이 사용되었는지 수집해야 한다면 렌더링 중에 가능
  ```javascript
  let collectedRulesSet = new Set();
  
  function useCSS(rule) {
    if (typeof window === 'undefined') {
      collectedRulesSet.add(rule);
    }
    useInsertionEffect(() => {
      // ...
    });
    return rule;
  }
  ```
- `useInsertionEffect`가 렌더링 중에 스타일을 주입하거나 `useLayoutEffect`를 사용하는 것보다 나은 이유:
  1. 렌더링 중에 스타일을 주입하면 브라우저가 컴포넌트 트리를 렌더링하는 동안 매 프레임마다 스타일을 다시 계산하므로 심하게 느려짐
  2. `useInsertionEffect`는 다른 Effect가 컴포넌트 내에서 실행될 때, `<style>` 태그가 이미 주입되어 있는 것을 보장함. `useEffect`나 `useLayoutEffect`를 사용하는 경우에는 오래된 스타일로 인해 Effect에서의 레이아웃 계산이 잘못될 수 있음
