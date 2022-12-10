# Preserving and Resetting State

## 개요

- React에 의해 관리되는 state가 어떤 컴포넌트에 속하는지는(연결, 연관되는지는), UI 트리에서 컴포넌트의 위치에 따라 결정한다.
- 이 성질을 이용해 동일한 위치에 동일한 종류의 컴포넌트가 올 때, state를 유지할지, 리셋할지 결정할 수 있다.

## UI 트리

- 브라우저는 UI를 표현하기 위해서 트리 구조를 사용한다.
  - DOM은 HTML elements를 표현.
  - CSSOM은 CSS를 표현.
  - 심지어 Accessibility tree라는 것도 있다.
- React 또한 UI를 관리하고 표현하기 위해 트리 구조를 사용한다.
- React는 JSX로부터 UI 트리를 생성한다. 그리고, ReactDom이 UI 트리를 반영하기 위해 브라우저 DOM을 수정한다. (모바일이면, ReactDom 대신 ReactNative가 UI 트리를 적절하게 사용하게 될 것이다.)

## "State"와 "UI 트리에서 컴포넌트의 위치" 간의 관계

- state를 이용한 코드를 컴포넌트에 작성할 때, 컴포넌트 내부에 state가 존재하는 것처럼 생각할 수 있다.
- 실제로 state는 컴포넌트 외부에서 React가 관리한다.
- React는 관리하던 state가 어떤 컴포넌트와 연관(연결)되는지 컴포넌트의 위치를 이용해 판단한다.
- 예시

    ```jsx
    export default function App() {
     return (
      <div>
       <Counter />
       <Counter />
      </div>
     );
    }
    
    function Counter() {
     const [count, setCount] = useState(0);
     return ( ... );
    }
    ```

  - App 컴포넌트에서 div의 자식으로 두 개의 Counter 컴포넌트를 렌더했다.
  - React는 두 개의 `count` state를 관리한다.
    - 첫 번째 `count` state는 div에서 첫 번째 Counter 컴포넌트와 관계가 있다.
    - 두 번째 `count` state는 div에서 두 번째 Counter 컴포넌트와 관계가 있다.
    - 따라서 두 개의 Counter 컴포넌트들은 서로 state에 상관없이 독립적으로 동작한다.
      - 두 번째 Counter의 값을 2 증가시키면, 두 번째 Counter 컴포넌트와 연관된 `count` state 값이 변경되지, 첫 번째 Counter 컴포넌트와 연관된 `count` state는 변경되지 않는다.
- **React는 UI 트리에서, 동일한 위치에 동일한 컴포넌트(동일한 component function의 JSX)를 렌더하는 경우, 렌더링 간에 state를 유지한다.**
- React는 state와 연관 있는 컴포넌트가 원래 위치에서 제거되거나, 원래 위치에 다른 컴포넌트가 위치하면, 해당 state를 제거한다. 그래서 이후의 렌더링에서 이전 state 값을 그대로 사용할 수 없다.

    ```jsx
    export default function App() {
      const [showSecond, setShowSecond] = useState(true);
    
      return (
        <div>
          <Counter />
          {showSecond ? <Counter /> : <p>second counter disappeared</p>}
          <label>
            <input 
              type="checkbox"
              checked={showSecond} 
              onChange={(e) => setShowSecond(e.target.checked)} 
              />
            Render second counter
          </label>
        </div>
      );
    }
    
    function Counter() {
      const [count, setCount] = useState(0);
    
      return (
        <div>
          <div>{count}</div>
          <button onClick={() => setCount(c => c + 1)}>plus</button>
        </div>
      );
    }
    ```

  - 초기에는 두 개의 Counter 컴포넌트가 App 컴포넌트의 div의 자식으로 렌더된다.
  - 그래서 plus 버튼을 클릭해 첫 번째 Counter의 `count` state를 3, 두 번째 Counter의 `count` state를 7로 만들었다고 하자.
    - 이때 체크박스의 체크를 해제하면, 두 번째 Counter 대신 p 태그가 렌더링된다.
    - React가 두 번째 Counter를 제거할 때, 연관된 state도 제거한다.
    - 다시 체크박스에 체크를 하면, `count` state가 0으로 초기화된 두 번째 Counter가 나타난다.

### “렌더 간에 동일한 컴포넌트가 UI 트리에서 동일한 위치에 나타나면, 상태가 보존된다” 의 예시

```jsx
import { useState } from "react";

export default function App() {
  const [isRedText, setIsRedText] = useState(false);
  return (
    <div>
      {isRedText ? (
        <Counter isRedText={true} />
      ) : (
        <Counter isRedText={false} />
      )}
      <label>
        <input type="checkbox" checked={isRedText} onChange={(e) => setIsRedText(e.target.checked)} />
        make text red
      </label>
    </div>
  );
}

function Counter({ isRedText }) {
  const [count, setCount] = useState(0);

  return (
    <div>
      <div style={{
        color: isRedText ? "red" : ""
      }}>{count}</div>
      <button onClick={() => setCount(c => c + 1)}>plus</button>
    </div>
  );
}
```

- `{isRedText ? (<Counter isRedText={true} />) : (<Counter isRedText={false} />)}` 코드 대신에, `<Counter isRedText={isRedText} />` 가 더 깔끔한 코드이지만 예시를 위해서 위처럼 작성했다.
- 두 개의 다른 `<Counter />` 태그가 있고, `isRedText` state가 변경될 때마다 서로 다른 `<Counter />` 태그가 사용되어서, 마치 `isRedText` state가 변경될 때마다, Counter 내부의 `count` state도 초기화될 것 같다고 생각할 수 있다.
- 하지만, `isRedText` state가 변경되어 리렌더링 되어도, 여전히 동일한 종류의 Counter 컴포넌트가 동일한 위치(App 컴포넌트 div 태그의 첫 번째 자식위치)에 렌더링 되므로 이전의 `count` state가 그대로 유지된다.
- 다른 두 개의 `<Counter />` 태그이더라도, 동일한 종류의 Counter 컴포넌트가 UI 트리 상에서 동일한 위치에 나타나면 React 입장에서는 동일한 Counter로 인식한다.
- React는 컴포넌트의 UI 트리 상의 위치를 key(address, 주소)로 생각해 해당 컴포넌트와 연관된 state를 관리한다.

### 중첩 함수를 이용해 컴포넌트를 정의했을 때, 생길 수 있는 버그

```jsx
import { useState } from "react";

export default function App() {
  const [count, setCount] = useState(0);

  function TextInput() {
    const [text, setText] = useState("");

    return (
      <input value={text} onChange={(e) => setText(e.target.value)} />
    );
  }

  return (
    <>
      <TextInput />
      <button type="button" onClick={() => setCount(c => c+1)}>
        clicked {count} times
      </button>
    </>
  );
}
```

- TextInput 컴포넌트의 state가 버튼을 클릭할 때마다 유지되지 않는다.
  - `<TextInput />` 라고 적어서 동일한 컴포넌트가 동일한 위치에 오는 것 같지만, 사실 버튼을 클릭해 `count` state가 변경되어 리렌더가 발생할 때마다 새로운 TextInput 함수가 생성되기 때문에 렌더 간에 서로 다른 컴포넌트가 하나의(동일한) 위치에 오게 된다.
  - 따라서, 매번 새로운 TextInput 컴포넌트가 생성되고, 기존 TextInput 컴포넌트가 사라질 때마다 연관된 state가 제거되어서 `text` state가 렌더 간에 유지되지 않는다.

## 렌더 간에 UI 트리에서 동일한 위치에 동일한 컴포넌트가 올 때, state가 유지되는 기본동작을 포기하는 방법

```jsx
import { useState } from "react";

export default function App() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  
  return (
    <div>
      {isPlayerA ? (
        <Score person="playerA" />
      ): (
        <Score person="playerB" />
      )}
      <button onClick={() => setIsPlayerA(prev => !prev)}>
        next player
      </button>
    </div>
  );
}

function Score({ person }) {
  const [score, setScore] = useState(0);

  return (
    <div>
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(s => s+1)}>+</button>
    </div>
  );
}
```

- 현재 player가 playerA이든 playerB이든 상관없이 하나의 Score 컴포넌트가 렌더 간에 동일한 위치에 나타난다.
- 따라서 player가 변경되어 App 컴포넌트가 리렌더 되어도, 동일한 Score 컴포넌트가 UI 트리 상에서 동일한 위치에 나타나기 때문에 `score` state는 유지된다. 반면에 `person` prop은 변경이 된다.
- player마다 독립된 `score` state를 가지게 하려면…
  - 첫 번째 방법: 두 개의 Score 컴포넌트를 서로 다른 위치에 렌더한다.

      ```jsx
      import { useState } from "react";
      
      export default function App() {
        const [isPlayerA, setIsPlayerA] = useState(true);
        
        return (
          <div>
            {isPlayerA ? (<Score person="playerA" />) : null}
            {!isPlayerA ? (<Score person="playerB" />) : null}
            <button onClick={() => setIsPlayerA(prev => !prev)}>
              next player
            </button>
          </div>
        );
      }
      
      function Score({ person }) {
        const [score, setScore] = useState(0);
      
        return (
          <div>
            <h1>{person}'s score: {score}</h1>
            <button onClick={() => setScore(s => s+1)}>+</button>
          </div>
        );
      }
      ```

    - playerA를 위한 첫 번째 Score, playerB를 위한 두 번째 Score는 다른 위치에 렌더 된다.
      - 첫 번째 Score 컴포넌트는 div의 첫 번째 자식에 위치
      - 두 번째 Score 컴포넌트는 div의 두 번째 자식에 위치
  - 두번째 방법: 동일한 위치에 동일한 컴포넌트가 위치하더라도, 위치를 식별할 때 UI 트리에서의 위치로 식별하지 않고, 주어진 key로 위치를 식별하도록 유니크한 key를 설정한다.
    - 디폴트로는, React가 parent 내에서 순서를 이용해 컴포넌트를 식별하고 state를 연관시켰다.
    - key를 지정하면, parent 컴포넌트 내에서 순서가 아니라, key 값을 이용해 컴포넌트를 식별하고 state를 연결한다.
      - 예시: 이 Score 컴포넌트는 “playerB”를 위한 컴포넌트이구나!

    ```jsx
    import { useState } from "react";
    
    export default function App() {
      const [isPlayerA, setIsPlayerA] = useState(true);
      
      return (
        <div>
          {isPlayerA ? (
            <Score key="playerA" person="playerA" />
          ) : (
            <Score key="playerB" person="playerB" />
          )}
          <button onClick={() => setIsPlayerA(prev => !prev)}>
            next player
          </button>
        </div>
      );
    }
    
    function Score({ person }) {
      const [score, setScore] = useState(0);
    
      return (
        <div>
          <h1>{person}'s score: {score}</h1>
          <button onClick={() => setScore(s => s+1)}>+</button>
        </div>
      );
    }
    ```

    - 동일한 위치에 동일한 Score컴포넌트가 위치한다.
      - 기존에는 `isPlayerA`가 `true`든 `false`든 Score 컴포넌트가 div(parent)의 첫 번째 자식이어서(=parent 내에서 위치), 렌더 간에 동일한 컴포넌트라고 판단했다.
      - 하지만 이제는 `isPlayerA` state가 변경되어 리렌더가 될 때마다 렌더 간에 Score 컴포넌트의 `key`값이 변경되기 때문에, 비록 위치와 종류가 동일하더라도 다른 컴포넌트라고 판단한다.
        - 따라서 매번 Score 컴포넌트가 unmount되고 이때 연관된 state가 React에서 제거된다.
        - 그래서 playerA와 playerB가 별도의 state를 가진 것처럼 동작한다.
