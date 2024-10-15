## 모듈상태의 한계

모듈 상태는 리액트 컴포넌트 외부에 존재하는 전역으로 정의된 싱글턴이기 때문에 컴포넌트 트리나 하위 트리마다 다른 상태를 가질 수 없다는 한계가 있다.

P97 참조

## 컨텍스트 사용이 필요한 시점

기본값이 있다면 공급자를 사용할 이유가 없지만, 전체 컴포넌트 트리의 하위 트리에 대해 다른 값을 제공할 필요가 있다면 공급자를 사용한다.

## 컨텍스트와 구독 패턴

하나의 컨텍스트를 사용해 전역 상태 값을 전파하는 것은 불필요한 리렌더링이 발생한다는 문제

구독을 이용한 모듈 상태는 이런 문제가 없지만 전쳬 컴포넌트 트리에 대해 하나의 값만 제공한다는 문제

컨텍스트와 구독을 함께 사용한다면 두 가지 문제점을 극복할 수 있다

하위 트리에서 분리된 값을 제공하고 리렌더링을 피하는 두 가지 이점을 모두 제공

구독 패턴

```tsx
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
```

구독 + 컨텍스트

```tsx
const useSelector = <S extends unknown>(selector: (state: State) => S) => {
  const store = useContext(StoreContext);
  return useSubscription(
    useMemo(
      () => ({
        getCurrentValue: () => selector(store.getState()),
        subscribe: store.subscribe,
      }),
      [store, selector]
    )
  );
};
```

```tsx
import { ReactNode, createContext, useContext, useRef, useMemo } from "react";
import { useSubscription } from "use-subscription";

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

type State = { count: number; text?: string };

const StoreContext = createContext<Store<State>>(
  createStore<State>({ count: 0, text: "hello" })
);

const StoreProvider = ({
  initialState,
  children,
}: {
  initialState: State;
  children: ReactNode;
}) => {
  const storeRef = useRef<Store<State>>();
  if (!storeRef.current) {
    storeRef.current = createStore(initialState);
  }
  return (
    <StoreContext.Provider value={storeRef.current}>
      {children}
    </StoreContext.Provider>
  );
};

const useSelector = <S extends unknown>(selector: (state: State) => S) => {
  const store = useContext(StoreContext);
  return useSubscription(
    useMemo(
      () => ({
        getCurrentValue: () => selector(store.getState()),
        subscribe: store.subscribe,
      }),
      [store, selector]
    )
  );
};

const useSetState = () => {
  const store = useContext(StoreContext);
  return store.setState;
};

const selectCount = (state: State) => state.count;

const Component = () => {
  const count = useSelector(selectCount);
  const setState = useSetState();
  const inc = () => {
    setState((prev) => ({
      ...prev,
      count: prev.count + 1,
    }));
  };
  return (
    <div>
      count: {count} <button onClick={inc}>+1</button>
    </div>
  );
};

const App = () => (
  <>
    <h1>Using default store</h1>
    <Component />
    <Component />
    <StoreProvider initialState={{ count: 10 }}>
      <h1>Using store provider</h1>
      <Component />
      <Component />
      <StoreProvider initialState={{ count: 20 }}>
        <h1>Using inner store provider</h1>
        <Component />
        <Component />
      </StoreProvider>
    </StoreProvider>
  </>
);

export default App;
```

## Provider 없이 Context를 사용하는 경우?

```tsx
const MyContext = React.createContext({ value: "default" });

const MyComponent = () => {
  const value = useContext(MyContext);
  return <div>{value}</div>;
};
```

- 앱 전체에서 사용되는 고정된 설정값
- 테마 정보나 언어 설정 등의 기본값
