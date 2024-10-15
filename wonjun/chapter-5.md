# 5장 : 리액트 컨텍스트와 구독을 통한 컴포넌트 상태 공유

## 모듈 상태의 한계

4장에서 정의한 모듈 형태의 store 는 모듈 형태를 통해 싱글톤으로 제공되기 때문에 만약 하위 컴포넌트에서 여러개의 store 값을 사용해야 할 경우 store 를 prop 등으로 내려주거나 다음과 같이 각 store를 참조하는 Counter 컴포넌트를 만들어야 한다.

```tsx
const count1Store = createStore({ count: 0 });
const count2Store = createStore({ count: 0 });

function Counter1() {
	const [count, setCount] = useStore(countStore)
	// ...
}

function Counter2() {
	const [count, setCount] = useStore(count2Store)
	// ...
}

function Help() {
	<>
		<h1>Store1</h1>
		<Counter />
		<Counter2 />
	</>
}
```

## Context API

이때 Context API를 사용하여 store 값을 공유할 수 있다.

```tsx
const StoreContext = createContext(createStore({ count: 0 }));

export function StoreProvider({ children, initialState }) {
  const storeRef = useRef();
  if (!storeRef.current) {
    storeRef.current = createStore(initialState);
  }

  return (
    <StoreContext.Provider value={storeRef.current}>
      {children}
    </StoreContext.Provider>
  );
}

export function useSelector(selector) {
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
}

export function useSetState() {
  const store = useContext(StoreContext);
  return store.setState;
}
```

4장에서 작성한 상태를 다음과 같이 Context API를 사용하도록 개조할 수 있다.

## 한번 해보자

```tsx
const count1Store = createStore({ count: 0 });
const count2Store = createStore({ count: 0 });

const selectCount = (state) => state.count;

function Counter() {
  const count = useSelector(selectCount);
  // ...
}

function Help() {
  <>
    <h1>Default Store</h1>
    <Counter />
    <Counter />

    <StoreProvider initialState={{ count: 10 }}>
      <h1>Store1</h1>
      <Counter />
      <Counter />

      <StoreProvider initialState={{ count: 20 }}>
        <h1>Store2</h1>
        <Counter />
        <Counter />
      </StoreProvider>
    </StoreProvider>
  </>;
}
```

이렇게 하면 Default Store 밑에는 카운터가 0부터 시작하고 Store1은 10부터, Store2는 20부터 시작한다.

굿이다.