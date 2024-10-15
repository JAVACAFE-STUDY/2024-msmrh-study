# 모듈 상태의 한계

모듈 상태는 싱글턴이기 때문에 컴포넌트 트리나 하위 트리마다 다른 상태를 가질 수 없다는 한계가 있다.

따라서 모듈 상태를 사용하면서 서로 다른 상태를 가지는 컴포넌트를 구현하려면, 다음과 같이 똑같은 코드를 가진 컴포넌트를 중복해서 만들어야 하며 이는 재사용성이 좋지 않다는 단점으로 이어진다.

```tsx
const store = createStore({count: 0});
const store2 = createStore({count: 0});

const Counter1 = () => {
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

const Counter2 = () => {
	const [state, setState] = useStore(store2);
//...이하 동일
```

이러한 모듈 상태의 한계를 극복하기 위해 컨텍스트를 도입할 수 있다.

# 컨텍스트와 구독 패턴 사용하기

컨텍스트는 공급자를 중첩해서 각 하위 트리마다 서로 다른 값을 제공할 수 있다. 하지만 불필요한 리렌더링이 발생한다는 문제가 있다.

구독을 이용한 모듈 상태는 리렌더링 문제가 없지만 전체 컴포넌트 트리에 대해 하나의 값만 제공한다는 문제가 있다.

따라서 두 가지를 함께 사용한다면 각자의 문제점을 극복할 수 있다.

둘을 함께 사용하는 패턴은 다음과 같다. 먼저 모듈 상태에서 사용했던 createStore를 통해 컨텍스트를 생성한다.

```tsx
type State = { count: number; text?: string };

const StoreContext = createContext<Store<State>>(
  createStore<State>({ count: 0, text: "hello" })
);
```

하위 트리에 스토어를 제공하기 위해 StoreContext.Provider를 감싼 StoreProvider를 구현한다.

```tsx
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
```

이 때 useRef를 사용해서 store를 저장한다. 이는 스토어 객체가 첫 번째 렌더링에서 한 번만 초기화되게 만들기 위함이다.

다음으로 스토어 객체를 사용하기 위해 useSelector 훅을 구현한다.

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

useSelector는 useContext와 함께 useSubscription을 사용하여 컨텍스트와 구독의 이점을 모두 누릴 수 있게 한다.

이후 useSetState를 통해 store에서 setState함수를 반환하는 훅을 제공한다.

```tsx
const useSetState = () => {
  const store = useContext(StoreContext);
  return store.setState;
};
```

이렇게 만든 컨텍스트는 다음과 같이 사용 가능하다.

```tsx
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
```

Component 컴포넌트는 모듈 상태를 사용할 때와 달리 특정 스토어와 결합되어 있지 않으며, 스토어는 Provider에서 관리되고 Component는 서로 다른 프로바이더 내에서 사용할 수 있다.

이렇듯 컨텍스트로 하위 트리에서 상태를 분리하고, 구독으로 리렌더링 문제를 피하는 패턴은 규모가 큰 애플리케이션에서 유용하다.
