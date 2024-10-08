모듈 상태를 전역 상태로 사용하려면 어떻게 해야할까?

모듈 상태란?
모듈 상태의 엄격한 정의는 ECMAScript(ES) 모듈 스코프에 정의된 상수 또는 변수다.

### 기초적인 구독 추가하기

```jsx
import { useEffect, useState } from "react";

type Store<T> = {
  getState: () => T;
  setState: (action: T | ((prev: T) => T)) => void;
  subscribe: (callback: () => void) => () => void;
};

const createStore = <T extends unknown>(initialState: T): Store<T> => {
  let state = initialState;
  const callbacks = new Set<() => void>();
  const getState = () => state;
  const setState = (nextState: T | ((prev: T) => T)) => {
    state =
      typeof nextState === "function"
        ? (nextState as (prev: T) => T)(state)
        : nextState;
    callbacks.forEach((callback) => callback());
  };
  const subscribe = (callback: () => void) => {
    callbacks.add(callback);
    return () => {
      callbacks.delete(callback);
    };
  };
  return { getState, setState, subscribe };
};

const store = createStore({ count: 0 });

const useStore = <T extends unknown>(store: Store<T>) => {
  const [state, setState] = useState(store.getState());
  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      setState(store.getState());
    });
    setState(store.getState());
    return unsubscribe;
  }, [store]);
  return [state, store.setState] as const;
};

const Component1 = () => {
  const [state, setState] = useStore(store);
  const inc = () => {
    setState((prev) => ({
      ...prev,
      count: prev.count + 1,
    }));
  };
  return (
    <div>
      {state.count} <button onClick={inc}>+1</button>
    </div>
  );
};

const Component2 = () => {
  const [state, setState] = useStore(store);
  const inc2 = () => {
    setState((prev) => ({
      ...prev,
      count: prev.count + 2,
    }));
  };
  return (
    <div>
      {state.count} <button onClick={inc2}>+2</button>
    </div>
  );
};

const App = () => (
  <>
    <Component1 />
    <Component2 />
  </>
);

export default App;
```

## 선택자와 useSubscription 사용하기

use-subscription

https://www.npmjs.com/package/use-subscription

https://github.com/facebook/react/blob/main/packages/use-subscription/src/useSubscription.js

useSubscription과 useSyncExternalStore
