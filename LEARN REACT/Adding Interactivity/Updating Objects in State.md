# 5. 객체 state 업데이트
state에는 모든 값을 저장할 수 있다. (객체도 가능!)

하지만 React state에 있는 객체를 직접 변이하면 안 되고,
객체를 새로 만들어서 새로 만든 것을 state에 넣어주어야 한다.

왜?? 
Mutation을 막기위해서!

## 5-1. Mutation
number, string, boolean 등은 불변(immutable, 변이할 수 없음)이거나 읽기 전용(read-only)이다.
따라서 이런 타입으로 된 state가 변경되면 리렌더링이 발생하여 값이 변경된다.

불변 타입인 number, string, boolean과 달리
**객체**는 내용을 변경할 수 있으며, 이를 `mutation`이라고 한다.

하지만 number, string, boolean처럼 불변하는 것처럼 취급해야 한다.

<br/>

### React에서 state 변이를 권장하지 않는 이유
#### 1) 디버깅
디버깅 과정에서 `console.log`를 사용해 state 변화를 추적할 때,
상태를 변경하지 않으면 이전에 로깅된 내용들이 최근 상태 변경으로 인해 삭제되지 않는다.
따라서 렌더링 사이에서 상태가 어떻게 변화했는지 명확하게 볼 수 있다.

#### 2) 최적화
일반적인 React의 최적화 전략은 이전 프로퍼티나 state가 다음 프로퍼티나 State와 동일한 경우 작업을 건너뛰는 것이다.
상태를 직접 수정(mutate)하지 않는다면(즉, 상태가 불변하다면), 변화를 체크하는 것이 매우 빠르다

#### 3) 새로운 기능
새로운 React 기능의 기반은 state를 스냅샷처럼 보고 다루는 것이다.
각 상태(state)는 특정 시점의 애플리케이션 상태의 스냅샷이며, 이전 상태를 변경하지 않고, 새로운 상태를 생성하여 이전 상태와 구분한다.
상태를 직접 변경하면 이러한 새로운 기능들을 제대로 활용하지 못할 수 있다.


#### 4) 요구사항 변경
Undo/Redo 구현, 변경 내역 표시, 사용자가 이전 값으로 폼을 리셋하는 것과 같은 애플리케이션의 몇몇 기능들은, 아무 것도 변이되지 않았을 때 더 쉽게 할 수 있다.
이는 state의 과거 복사본들을 메모리에 보관하고, 필요할 때 그것들을 재사용할 수 있기 때문이다.
만약 상태를 직접 변경하는 방식으로 코드를 작성하면, 나중에 시스템에 변화를 추적하거나 되돌리는 기능을 추가하는 것이 복잡해질 수 있다.

#### 5) 더 간단한 구현
React는 변이에 의존하지 않게 설계되어 있기 때문에, (데이터의 불변성을 유지하므로) 객체를 특별한 방식으로 변경하거나 감쌀 필요가 없다.
이러한 방식은 React가 효율적이면서도 예측 가능한 방식으로 동작하게 하며, 개발자에게 더 단순하고 직관적인 개발 경험을 제공한다.

즉, state의 불변성을 유지하는 접근 방식을 염두에 두고 개발된 새로운 리액트 기능들과의 호환성을 보장하기 위한 것!


<br/>

## 5-2. Spread 구문을 사용해서 객체 복사하기
객체의 일부 속성만 변경하고 싶은 경우,
spread 구문을 사용하면 모든 속성을 복사할 수 있다.

```js
setPerson({
  ...person, // Copy the old fields
             // 이전 필드를 복사합니다.
  firstName: e.target.value // But override this one
                            // 단, first name만 덮어씌웁니다. 
});
```

🚨주의할 점!
spread 구문은 shallow copy로, 한 단계 깊이만 복사한다.
속도는 빠르지만 중첩된 속성을 업데이트하려면 여러 번 사용해야 한다.

<br/>

## 5-3. 중첩 객체 업데이트하기

아래와 같은 중첩 객체가 있을 때, 이를 업데이트하는 방법들을 알아보자!

```js
const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
});
```

### 1) 새 객체 생성하기
```js
const nextArtwork = { ...person.artwork, city: 'New Delhi' };
const nextPerson = { ...person, artwork: nextArtwork };
setPerson(nextPerson);
```

### 2) 한번에 써도 됨
```js
setPerson({
  ...person, // Copy other fields
  artwork: { // but replace the artwork
             // 대체할 artwork를 제외한 다른 필드를 복사합니다.
    ...person.artwork, // with the same one
    city: 'New Delhi' // but in New Delhi!
                      // New Delhi'는 덮어씌운 채로 나머지 artwork 필드를 복사합니다!
  }
});
```

<br/>

## 5-4. Immer
state가 깊게 중첩되어 있는 경우, Immer를 사용하면 더 간편하다.
Immer는 변이 구문으로 작성해도 알아서 사본을 생성해준다.

```js
updatePerson(draft => {
  draft.artwork.city = 'Lagos';
});
```

여기서 draft는 프록시라는 특수한 유형의 객체로, 사용자가 수행하는 작업을 기록하고 있어서
어떤 부분이 변경되었는지 파악해서 완전히 새로운 객체를 생성해준다.

