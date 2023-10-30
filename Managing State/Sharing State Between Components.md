- 형제 컴포넌트 간 state 공유하려면 state 끌어올리기([리액트로 사고하기](https://github.com/Jungle-JavaScript-Study/react-docs-study/blob/main/Quick%20Start/Thinking%20in%20React.md#%EB%A6%AC%EC%95%A1%ED%8A%B8%EB%A1%9C-%EC%82%AC%EA%B3%A0%ED%95%98%EA%B8%B0) 참고)
- &nbsp;비제어 컴포넌트: 로컬 state를 가져서 부모가 온전히 컨트롤할 수 없는 경우 → 추상화는 잘 되어있지만 유연성은 떨어짐<br>
    ↔︎ 제어 컴포넌트: props로 제어할 수 있어서 부모가 온전히 컨트롤 할 수 있음 → 유연성은 좋지만 부모가 props를 전달할 때 신경써야 함<br>
    ⇒ 어떤 정보를 제어(props)하고 어떤 정보를 비제어(state)할지 고려해서 코드를 구성
- 각 state는 단일 진실 공급원을 가짐: 해당 state를 소유하는 유일한 컴포넌트를 선정하고 다른 컴포넌트에서는 해당 정보를 복사하는 대신 state 끌어올리기로 획득해야 함
