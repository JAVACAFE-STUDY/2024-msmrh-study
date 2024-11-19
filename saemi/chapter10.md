# 사용 사례 시나리오 4: React Tracked

- 속성 감지 기반 자동 렌더링 최적화

## React Tracked 이해하기

- React Tracked는 상태 관리 기능은 제공하지 않지만 렌더링 최적화 기능을 제공한다. 이를 상태 사용 추적이라고 한다.

- useTracked는 상태를 프락시로 감싸고 사용을 추적한다.

```JS
const useFirstName = () => {
    const [{firstName}] = useTracked();
    return firstName
}
```

- useContext(NameContext)와 사용법이 같지만 useTracked는 내부에서 상태 사용을 추적하고 렌더링을 자동으로 최적화한다.

- React Tracked와 Valtio는 proxy-compare라는 내부 라이브러리를 사용해 상태 사용을 추적한다.

## useState, useReducer와 함께 React Tracked 사용하기

- React Tracked는 주로 리액트 컨텍스트를 대체할 용도로 사용된다.

```tsx
import { useState } from "react";
import { createContainer } from "react-tracked";
import React from "react";

const useValue = () =>
  useState({
    count: 0,
    text: "Hello",
  });

const { Provider, useTracked } = createContainer(useValue);

const Counter = () => {
  const [state, setState] = useTracked();
  const inc = () => {
    setState((prev) => ({ ...prev, count: prev.count + 1 }));
  };

  return (
    <div>
      count:{state.count}
      <button onClick={inc}>+1</button>
    </div>
  );
};

const TextBox = () => {
  const [state, setState] = useTracked();
  const setText = (text: string) => {
    setState((prev) => ({ ...prev, text }));
  };

  return (
    <div>
      <input value={state.text} onChange={(e) => setText(e.target.value)} />
    </div>
  );
};

const App = () => (
  <Provider>
    <div>
      <Counter />
      <Counter />
      <TextBox />
      <TextBox />
    </div>
  </Provider>
);

export default App;
```

## useReducer와 함께 React Tracked 사용하기

```tsx
import React, { useEffect, useReducer } from "react";
import { createContainer } from "react-tracked";

const useValue = () => {
  type State = { count: number; text: string };
  type Action = { type: "INC" } | { type: "SET_TEXT"; text: string };

  const [state, dispatch] = useReducer(
    (state: State, action: Action): State => {
      switch (action.type) {
        case "INC":
          return { ...state, count: state.count + 1 };
        case "SET_TEXT":
          return { ...state, text: action.text };
      }
    },
    { count: 0, text: "Hello" }
  );

  useEffect(() => {
    console.log("latest state", state);
  }, [state]);

  return [state, dispatch] as const;
};

const { Provider, useTracked } = createContainer(useValue);

const Counter = () => {
  const [state, dispatch] = useTracked();
  const inc = () => {
    dispatch({ type: "INC" });
  };

  return (
    <div>
      count: {state.count}
      <button onClick={inc}>+1</button>
    </div>
  );
};

const TextBox = () => {
  const [state, dispatch] = useTracked();
  const setText = (text: string) => {
    dispatch({ type: "SET_TEXT", text });
  };

  return (
    <div>
      <input value={state.text} onChange={(e) => setText(e.target.value)} />
    </div>
  );
};

const App = () => (
  <Provider>
    <div>
      <Counter />
      <Counter />
      <TextBox />
      <TextBox />
    </div>
  </Provider>
);

export default App;
```

## React Redux와 함께 React Tracked 사용하기

- React Tracked는 리액트 컨텍스트를 사용하지 않는 경우를 위해 createTrackedSelector라는 저수준 함수를 제공한다.

```JS
const useTrackedState = createTrackedSelector(useSelector);
```

\*React Redux

React Redux는 내부적으로 리액트 컨텍스트를 사용하지만 상태 값을 전파하는 데는 컨텍스트를 사용하지 않는다. 의존성 주입에는 리액트 컨텍스트를 사용하지만 상태 전파는 구독을 통해 이뤄진다.
