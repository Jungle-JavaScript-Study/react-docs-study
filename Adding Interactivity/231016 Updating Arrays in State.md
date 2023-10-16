
### JS에서 배열은 mutable(변경 가능)이지만 state에 저장할 때는 immutable(불변)로 취급해야 함.
### state에 저장된 배열을 업데이트하려면, 새로운 배열(또는 기존 배열 복사본)을 만들고, 새 배열을 사용하도록 state 설정해야 함.
<br />

## 변이 없이 배열 업데이트하기

### JS에서 배열은 객체의 한 종류일 뿐이다. 객체와 마찬가지로 React state에 있는 배열은 읽기 전용으로 취급해야 함.

```Javascript
arr[0] = 'bird' // -> X. 배열 내부 항목 재할당 금지
push(), pop() // -> X. 배열 변이 메서드 사용 금지
```

### 배열 업데이트할 때마다 state 설정 함수에 새 배열을 전달하고 싶어요!
#### -> filter(), map() 같은 비변이 메서드 호출해서 새 배열을 만들고, 만들어진 새 배열을 state로 설정해.
<br />

### 배열 연산 참조 표.
#### 왼쪽 열 메서드는 피하고 오른쪽 열에 있는 걸 쓰자.



|      |비추천 (배열 직접 변이)| 추천 (새 배열 반환)|
|------|---|---|
|추가|push, unshift|<span style="background-color: black; color:lightgreen">concat, [...arr] spread syntax</span>|
|삭제|pop, shift, splice|<span style="background-color: black; color:lightgreen">filter, slice</span>|
|교체|splice, arr[i] = ... assignment|<span style="background-color: black; color:lightgreen">map</span>|
|정렬|reverse, sort|<span style="background-color: black; color:lightgreen">배열 먼저 복사하고 처리</span>|

### immer를 쓰면 위 메서드 모두 사용할 수 있다고 함.
<br/>

### slice와 splice는 매우 다르므로 주의!<br/>
### -> 변이를 하지 않는 slice를 훨씬 더 많이 사용할 것
<br />


## 배열에 추가하기

### push()는 배열 변이함 (not good)
### 대신에 전개 구문을 쓰자 (여러 방법이 있지만 이게 쉬움)

```Javascript
setArtists( // 상태 변경
  [
    ...artists, // 이전 항목을 모두 갖고 있고
    { id: nextId++, name: name } // 새 항목을 끝에 가지는 새 배열로
  ]
);
```
#### 이렇게 하면 배열 변이 안 하고 push와 똑같은 기능을 할 수 있음~
#### ...artists를 뒤에 놓으면 배열 앞에 새로운 항목이 추가됨 (unshift()의 기능)
<br />

## 배열에서 제거하기 

### filter 메서드 쓰면 쉽다

```Javascript
setArtists(
  artists.filter(a => a.id !== artist.id)
);

// "artist.id와 다른 ID를 가진 artists로 구성된 배열을 생성한다"는 의미 (원본 배열 수정 X)
```
<br />

## 배열 변경하기

### map() 을 쓰자! (새로운 배열 생성해서 수행함)
<br/>

## 배열에서 항목 교체하기 

### 하나 이상의 항목 바꾸고 싶다!
### -> 이것도 map 쓰자!
<br/>

## 배열에 삽입하기
### 특정 위치에 요소를 삽입하고 싶다!
### -> ... 전개 구문과 slice() 메서드를 함께 사용하자

```Javascript
    const insertAt = 1; // 어떤 인덱스든 될 수 있겠죠? 이 예시에서는 1번째 인덱스에 들어가겠군요.
    const nextArtists = [
      // 삽입 지점 이전의 요소들:
      ...artists.slice(0, insertAt),
      // 새롭게 삽입될 항목:
      { id: nextId++, name: name },
      // 삽입 지점 이후 요소들:
      ...artists.slice(insertAt)
    ];
    setArtists(nextArtists);
```

<br />

## 배열에 다른 변경 사항 적용하기 

### 배열을 reverse 하거나 sort 하고 싶어요! (근데 reverse(), sort()는 원래 배열을 변이해요)
### -> 배열을 먼저 복사하고 변이하자
<br/>

### 전개 구문으로 복사본을 만들고 복사본에 변이 메서드를 사용하거나, 
### newList[0] = "무언가" 이렇게 할당할 수 있어요.
<br/>

### 하지만, 배열을 복사하더라도 배열 내부 기존 항목을 직접 변이할 수 없어요.
### 얕은 복사가 이뤄져 새 배열에 원래 배열과 동일한 항목이 포함되기 때문이죠.
### 복사된 배열 내부 객체를 수정하면 기존 state를 변이하게 됩니다.

```Javascript
const nextList = […list];
nextList[0].seen = true; // 문제 발생: list[0]을 변이함
setList(nextList);
```

###  *list와 nextList는 서로 다른 배열이지만, nextList[0]과 list[0]은 같은 객체를 가리킵니다!!
#### 해결 방법은 아래에 나옵니다.
<br />

## 배열 내부의 객체 업데이트하기

### 객체는 배열 "내부"에 위치하지 않음
### 배열의 각 객체는 배열이 "가리키는" 별도의 값
### 다른 배열의 동일한 요소를 가리킬 수 있음에 주의.


### 중첩된 state 업데이트 할 때는 업데이트하려는 지점부터 최상위까지 복사본 만들어야 함.

```Javascript
const myNextList = [...myList];
const artwork = myNextList.find(a => a.id === artworkId);
artwork.seen = nextSeen; // 문제: 존재하는 요소를 변이함.
setMyList(myNextList);
```

```Javascript
setMyList(myList.map(artwork => {
  if (artwork.id === artworkId) {
    // 변화들과 함께 새로운 객체를 생성
    return { ...artwork, seen: nextSeen };
  } else {
    // 변화 X
    return artwork;
  }
}));

// 이렇게 하면 변이 X
```

### 새롭게 만들어 낸 객체만 변이하자
<br />

## Immer로 간결한 업데이트 로직 작성하기 

### 일반적으로 state를 몇 레벨 이상 깊이 업데이트할 필요는 없음(깊다면 평평하게 만들자)
### state 구조 변경하고 싶지 않다면, Immer를 사용해보자.(변이 구문 써도 자동으로 사본 생성해줘서 편함!)
<br />

```Javascript
updateMyTodos(draft => {
  const artwork = draft.find(a => a.id === artworkId);
  artwork.seen = nextSeen;
});

// Immer를 쓰면, artwork.seen = nextSeen과 같은 변이도 오케이다.
```


## 요약

### state에 배열 넣을 수 있지만 변경할 수는 없다.
### 배열 변이하지 말고 새로운 사본을 만들어서 state 업데이트 하자.
### 배열 전개 구문을 써서 새 항목과 함께 배열 만들 수 있다. [...arr, newItem]
### filter() 및 map()을 사용하여 새 배열 만들 수 있다.
### 코드를 간결하게 하기 위해 Immer를 쓸 수 있다. 