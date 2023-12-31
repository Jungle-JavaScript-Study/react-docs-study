## state로 입력에 반응하기

### React는 선언형임.
### UI 개별적 직접 조작 X, 다양한 상태 기술, 사용자 입력에 반응해 상태를 전환함.
<br />

## 선언형 vs 명령형 UI 차이
### 사용자가 답변 제출하는 form을 생각해보자.
<br />

#### - form에 뭔가 입력하면 Submit 버튼 활성화될 거임.
#### - 제출 누르면 폼과 버튼 비활성화 되고 스피너 나타날 것.
#### - 네트워크 요청 성공 ? 
#### form 숨겨지고 "Thank you" 메세지 띄움.
#### : 오류 메세지 표시 및 form 다시 활성화.
<br/>

#### 명령형 프로그래밍에서 상호작용 구현하는 데 위 내용이 직접적으로 해당함. 정확한 지침 작성해야하는 것.
<br />

#### 컴퓨터에게 이것저것 내용 지시하는 것이 명령형.
#### -> 단순한 UI 조작에는 충분하지만, 복잡한 시스템에서 관리하기 매우 어려워짐. 
<br />

### 그래서 React가 탄생했다!

#### 직접 조작 x, 표시할 내용을 "선언"하면 UI를 업데이트할 방법을 React가 알아냄.
#### 마치 택시 기사에게 목적지만 알려주는 것과 같음.
<br />

## UI를 선언적인 방식으로 생각하기

### UI를 React로 구현하는 과정

#### 1. 컴포넌트의 다양한 시각적 상태를 식별.
#### 2. 상태 변화를 촉발하는 요소를 파악.
#### 3. useState를 사용하여 메모리의 상태를 표현.
#### 4. 필수적이지 않은 state 변수 제거.
#### 5. 이벤트 핸들러 연결하여 state를 설정(set).
<br />

## Step 1: 컴포넌트의 다양한 시각적 상태 식별
#### React는 디자인과 컴퓨터 과학의 교차점이다..
<br />

### 먼저, 사용자에게 표시될 수 있는 UI의 다양한 "상태"를 모두 시각화해야 함.
#### Empty, Typing, Submitting, Success, Error 등등.
<br />

### 마치 디자이너처럼 다양한 상태에 대한 "목업"을 만들어 놔 보자.

### 컴포넌트에 시각적 상태가 많으면 한 페이지에 다 표시해놓는 게 편리할 수 있음.
### -> 이러한 페이지를 보통 "living styleguides" 또는 "storybooks" 라고 함.
<br/>

## Step 2: 상태 변경을 촉발하는 요인 파악
#### 사람의 입력(버튼 클릭, 필드 입력, 링크 이동 등)과 컴퓨터의 입력(네트워크 응답 도착, 시간 초과, 이미지 로딩 등)에 대한 응답으로 상태 변경 촉발할 수 있음.
<br />

### state 변수 설정해야 UI 업데이트 가능.
#### text 입력하면 Empty -> Typing
#### 제출 클릭하면 Submitting state로
#### 등등등...
<br />

#### Note: 사람의 입력에는 종종 이벤트 핸들러가 필요함
<br />

#### 각 상태가 적힌 원을 그리고 화살표로 흐름을 그려보면 좋다고 합니다.
<br />

## Step 3: 메모리의 상태를 useState로 표현
### 단순할수록 좋다. 반드시 필요한 state부터 시작하자!

### 좋은 방법을 즉시 떠올리기 어렵다면 모든 상태를 확실하게 다룰 수 있을 만큼 충분한 state를 먼저 추가해보자.
#### (state 리팩토링도 과정의 일부라고 합니다.)
<br/>

## Step 4: 비필수적인 state 변수 제거

### 당신의 state 변수에 물어볼 수 있는 몇가지 질문들
<br/>

#### - state가 모순을 야기하는가? 예를 들어 동시에 true일 수 없는 것들 (isTyping과 isSubmitting)
#### 이 경우, 불가능한 상태를 없애기 위해 3가지 값(typing, submitting, success)을 가진 status로 결합할 수 있음. 
<br />

#### - 다른 state 변수에 이미 같은 정보가 있는가?
#### isEmpty, isTyping 동시에 true 될 수 없음. isEmpty 제거하고 answer.length === 0으로 확인 가능.
<br />

#### - 다른 state 변수를 뒤집으면 동일한 정보를 얻을 수 있는가? 
#### isError는 error !== null로 확인할 수 있기 때문에 필요없음.
<br/>

#### 이런 식으로 하면, 7개의 state에서 3개의 필수적인 state가 남습니다.
```Javascript
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing'); // 'typing', 'submitting', or 'success'
```

### deep dive 
#### null이 아닌 error는 status가 success일 때 의미가 없음. reducer를 쓰면 되는데, 이건 다음 시간에..
<br />

## Step 5: 이벤트 핸들러 연결하여 state 설정
### 마지막으로, state 변수 설정하는 이벤트 핸들러를 만들고 연결합니다~!
<br />

## 요약

### - 선언형 프로그래밍은 UI를 세밀하게 관리(명령형)하지 않고 각 시각적 상태에 대해 UI를 기술하는 것을 의미함.
### - 컴포넌트 개발할 때
#### 1. 모든 시각적 상태를 식별하고,
#### 2. 사람 및 컴퓨터가 상태 변화를 촉발하는 요인을 결정하고,
#### 3. useState로 상태를 모델링,
#### 4. 버그를 막기 위해 필수적인 state만 남기고,
#### 5. 이벤트 핸들러를 연결해서 state를 설정하세요~!
