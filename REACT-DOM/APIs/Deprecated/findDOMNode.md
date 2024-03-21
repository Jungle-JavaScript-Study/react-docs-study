## `findDOMNode`
- **`findDOMNode`** 는 **React 의 클래스 컴포넌트** 에서 브라우저의 DOM 노드를 찾는 방법
- **Reference**
    
    ```jsx
    import { findDOMNode } from 'react-dom';
    
    const domNode = findDOMNode(componentInstance);
    ```
    
- **Parameters**
    - **`componetInstance`** : 컴포넌트 하위 클래스의 인스턴스. 클래스 컴포넌트의 경우 **`this`** 가 포함되어 있음
- **Returns**
    - 주어진 `componentInstance` 에서 **가장 처음 등장하는 브라우저 DOM 노드를 반환**
    - 컴포넌트가 null 이나 false 를 렌더링할 경우 `findDOMNode` 는 `null` 을 반환
- **Caveats**
    - 컴포넌트가 배열이나 다수의 자식을 가진 Fragment 를 반환할 수 있음 - 이런경우 DOM 노드 중 비어있지 않은 첫 번째 자식을 반환함.
    - `findDOMNode` 는 컴포넌트가 마운트 되었을 때(컴포넌트가 DOM에 배치되었을 때)만 동작함. - 컴포넌트가 아직 마운트되지 않은 상태에서 호출했을 때 예외 발생
    - `findDOMNode` 는 호출할 당시의 결과만 반환. 그 후 자식 컴포넌트가 다른 노드를 렌더링해도 그 변화에 대해 알 수 없음.
    - `findDOMNode` 는 클래스 컴포넌트에서만 사용 가능. 함수형 컴포넌트는 사용할 수 없음.
- **Usage**
    - **클래스 컴포넌트의 root DOM 노드 찾기**
        
        ```jsx
        class AutoselectingInput extends Component {
          componentDidMount() {
            const input = findDOMNode(this);
            input.select()
          }
        
          render() {
            return <input defaultValue="Hello" />
          }
        }
        ```
        
        - 이 때, input 변수는 <input> DOM 요소를 값으로 갖게 됨.
        - **`console`** 에 `findDOMNode` 는 `StrictMode` 에서 `deprecated` 되었다고 경고
- **Alternative**
    - findDOMNode 를 사용하는 코드는 JSX 노드와 이에 해당하는 코드 사이의 연결이 명시적이지 않아 취약하다.
    - 클래스 컴포넌트에서는 **`createRef`** 를 이용해 <div> 태그로 감싸줘 관리할 수 있다.
    - 최신 **React** 에서는 클래스형 컴포넌트도 사용하지 않기 때문에 **`useRef`** 를 대신 호출하여 동일한 기능을 구현한다.

    
    💡 컴포넌트는 종종 자식의 위치와 크기를 알아야할 때가 있어서, `findDOMNode(this)` 를 사용하여 자식들을 찾고 측정을 위해 `getBoundingClientRect` 와 같은 DOM 메서드를 사용했지만 이러한 사용사례에 직접적으로 대응되는 방법이 없기 때문에 `findDOMNode` 는 더 이상 사용되지 않음에도 React 에서 완전히 사라지지는 않았다. 그동안 이를 해결하기 위해 콘텐츠 주위를 `<div>` 로 감싸주고 그 노드의 `ref` 를 가져왔다. - ( 하지만 콘텐츠를 감싸는 요소를 추가하는 방법은 스타일을 손상할 수 있다. )
