- 속성 감지를 기반으로 자동으로 렌더링 최적화를 수행하는 `상태 사용 추적 라이브러리`
- 다른 상태 관리 라이브러리와 함께 사용 가능

## React Tracked 이해하기

- 상태 관리 기능을 제공하지는 않음
- `상태 사용 추적`
    - 렌더링 최적화 기능을 제공

- `Context API`를 사용할 경우 구독하지 않는 값이 변경되더라도 리렌더링이 발생함

- `React Tracked`는 프락시로 이를 해결하고자 함

- `useContext` 대신 `useTracked` 훅을 정의할 수 있음
    - 상태를 프락시로 감싸고 사용을 추적함
        
        ```tsx
        const useFirstName = () => {
        	const [{ firstName }] = useContext(NameContext);
        	
        	return firstName
        };
        
        const useFirstName = () => {
        	const [{ firstName }] = useTracked();
        	
        	return firstName
        };
        ```
        
        - 내부에서 상태 사용을 추적하고 렌더링을 자동으로 최적화함

- `Valtio`와 `React Tracked`는 상태 사용 추적 기능을 위해 [`proxy-compare`](https://github.com/dai-shi/proxy-compare#readme)라는 라이브러리를 사용

## useState, useReducer와 함께 React Tracked 사용하기

- `React Tracked`는 주로 리액트 컨텍스트를 대체할 용도로 사용됨

### useState와 함께 React Tracked 사용하기

```tsx
import { useState } from "react";
import { createContainer } from "react-tracked";

const useValue = () => useState({ count: 0, text: "hello" });

const { Provider, useTracked } = createContainer(useValue);

const Counter = () => {
  const [state, setState] = useTracked();
  
  const inc = () => {
    setState((prev) => ({ ...prev, count: prev.count + 1 }));
  };
  
  return (
    <div>
      count: {state.count} <button onClick={inc}>+1</button>
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
```

- `useTracked`에 의해 반환된 `state` 객체는 추적됨
- 접근된 속성이 변경된 경우에만 리렌더링을 감지함
- `Context API`를 사용했을 때와 다르게 `Counter` 버튼을 클릭하면 `Counter` 컴포넌트만 리렌더링됨
- `createContainer`는 `[state, update]` 튜플을 반환하는 형태를 갖춘 어떤 것을 사용하든 상관 없음

### useReducer와 함께 React Tracked 사용하기

```tsx
const useValue = () => {
  type State = { count: number; text: string };

  type Action = { type: "INC" } | { type: "SET_TEXT"; text: string };

  const [state, dispatch] = useReducer(
    (state: State, action: Action) => {
      if (action.type === "INC") {
        return { ...state, count: state.count + 1 };
      }
      if (action.type === "SET_TEXT") {
        return { ...state, text: action.text };
      }
      throw new Error("unknown action type");
    },
    { count: 0, text: "hello" }
  );

  useEffect(() => {
    console.log("latest state", state);
  }, [state]);

  return [state, dispatch] as const;
};
```

- useEffect
    - 단순히 로그 출력에만 사용하는 것이 아니라, 원격 리소스와 상호 작용하는 용도로도 사용될 수 있음
- useValue 훅은 state와 dispatch 튜플을 반환함

```tsx
const { Provider, useTracked } = createContainer(useValue);

const Counter = () => {
  const [state, dispatch] = useTracked();

  const inc = () => dispatch({ type: "INC" });

  return (
    <div>
      count: {state.count} <button onClick={inc}>+1</button>
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

```

- `use-context-selector`
    - `React Tracked` 는 `use-context-selector` 라이브러리를 사용해 리렌더링 최적화를 함
    - 해당 라이브러리의 `selector` 함수를 사용해 컨텍스트 값을 구독할 수 있고, 이 구독을 통해 리액트 컨텍스트의 제약을 우회함

## React Redux와 함께 React Tracked 사용하기

- 리액트 컨텍스트를 사용하지 않는 경우를 위해 `createTrackedSelector` 라는 저수준 함수를 제공함
- `useSelector`  훅을 받아 `useTrackedState` 라는 훅을 반환함
    
    ```tsx
    const useTrackedState = createTrackedSelector(useSelector);
    ```
    
- `useSelector`
    - 선택자 함수를 받아 선택자 함수의 결과를 반환하는 훅
    - 결과가 변경된 경우 리렌더링을 발생시킴
- `useTrackedState`
    - 상태 사용을 추적하기 위해 전체 상태를 프락시로 감싸서 반환하는 훅

<aside>
💡

React Redux
- 내부적으로 리액트 컨텍스트를 사용하지만, 상태 값을 전파하는 데는 컨텍스를 사용하지 않음
- 의존성 주입에는 리액트 컨텍스트를 사용하지만 상태 전파는 구독을 통해 이뤄짐
- React Redux의 useSelector는 선택자의 결과가 변경된 경우에만 리렌더링하도록 최적화돼 있음

</aside>

```tsx
import { createStore } from "redux";
import { Provider, useDispatch, useSelector } from "react-redux";
import { createTrackedSelector } from "react-tracked";

type State = { count: number; text: string };
type Action = { type: "INC" } | { type: "SET_TEXT"; text: string };

const initialState: State = { count: 0, text: "hello" };

const reducer = (state = initialState, action: Action) => {
  if (action.type === "INC") {
    return { ...state, count: state.count + 1 };
  }

  if (action.type === "SET_TEXT") {
    return { ...state, text: action.text };
  }

  return state
};

const store = createStore(reducer);

const useTrackedState = createTrackedSelector<State>(useSelector);

const Counter = () => {
  const dispatch = useDispatch();

  const { count } = useTrackedState();

  const inc = () => dispatch({ type: "INC" });

  return (
    <div>
      count: {count} <button onClick={inc}>+1</button>
    </div>
  );
};

const TextBox = () => {
  const dispatch = useDispatch();

  const state = useTrackedState();

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
  <Provider store={store}>
    <div>
      <Counter />
      <Counter />
      <TextBox />
      <TextBox />
    </div>
  </Provider>
);
```

- `const useTrackedState = createTrackedSelector<State>(useSelector);`
- `const state = useTrackedState();`
    - 위 두 줄을 제외하고는 일반적인 React Redux 사용 패턴과 비슷함
- `useSelector` 를 사용하면 개발자가 리렌더링을 최적화하기 위해 세세하게 제어할 수 있고 책임도 많아지게 되나, `useTrackedState` 를 사용하면 훅이 자동으로 리렌더링을 제어함

## 향후 전망

- 첫 번째 방법
    - 리액트 컨텍스트에서 `createContainer` 를 사용
- 두 번째 방법
    - `React Redux` 에서 `createTrackedSelector` 를 사용
- `createTrackedSelector`
    - `proxy-compare` 로 구현됨
- `createContainer` 함수
    - `createTrackedSelector` 와 `use-context-selector` 라이브러리로 구현된 한 단계 더 추상화된 함수
    
- `useContextSelect` 를 리액트에서 정식으로 지원할 수도 있다고 함…?