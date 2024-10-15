- 전역 상태를 위한 두 가지 방법의 장점
    - 리액트 컨텍스트 사용
        - 하위 트리에 전역 상태를 제공할 수 있음
        - 컨텍스트 공급자를 중첩하는 것이 가능
        - 리액트 생명 주기 내에서 `useState`와 같은 훅으로 전역 상태 제어 가능
    - 구독을 사용
        - 불필요한 리렌더링을 막을 수 있음

## 모듈 상태의 한계

- 컴포넌트 외부에 존재하는 전역으로 정의된 싱글턴이기 때문에 컴포넌트 트리나 하위 트리마다 다른 상태를 가질 수 없다는 한계

→ 모듈 상태는 싱글턴이기 때문에 모듈 상태를 전역 상태로 사용하는 컴포넌트는 재사용이 어려운 한계가 있음

→ 새로운 스토어를 구독하는 컴포넌트를 매번 재정의 해야하는 번거로움이 있음

```tsx
const store = createStore({ count: 0 });
const Component1 = () => {
	const [state, setState] = useStore(store);
	
	...
};

const store2 = createStore({ count: 0 });
const Component2 = () => {
	const [state, setState] = useStore(store2);
	
	...

};
```

- Provider 를 다르게 적용(리액트 컨텍스트 활용)하여 하나의 컴포넌트를 재사용하면 좋을 듯

```tsx
const Component1 = () = {
	<Provider1>
		<Counter />
	</Provider1>
};

const Component2 = () = {
	<Provider2>
		<Counter />
	</Provider2>
};
```

## 컨텍스트 사용이 필요한 시점

- 컨텍스트 작동 방식
    - 공급자는 중첩 가능하며, 가장 안쪽에 위치한 공급자의 값을 사용
    - 공급자가 없는 경우 기본값을 사용

P.103

- 컨텍스트를 언제 사용해야 하는지?
    - 컨텍스트에 적절한 기본값을 설정하는 것은 중요
        - 컨텍스트 공급자는 기본 컨텍스트 값 또는 부모 공급자가 제공하는 값이 있는 경우 이 값을 재정의(override)하는 메서드라고 볼 수 있음
    - 적절한 기본값이 있다면 공급자를 사용할 이유가 없지만, 전체 컴포넌트 트리의 하위 트리에 대해 다른 값을 제공할 필요가 있다면 공급자를 사용해야 함
        
        → 이전 챕터에서도 궁금했는데, 일반적으로 ‘Provider의 사용 === 해당 범위 내에서 컨텍스트 값을 사용하겠다.’ 고 명시하는거라고 생각해서 일부러 Context에 기본값을 사용하지 않는 편인데, 컨텍스트를 Provider없이 사용하는 것이 맞는건가? 하는 궁금증이 생겼음
        
    - 컨텍스트를 사용한 전역 상태의 경우 트리 최상위에서 하나의 공급자만 사용할 수 있음 
        - 구독을 통한 모듈 상태를 사용하는 편이 나을 수 있음
        - 전역 상태를 위한 컨텍스트는 서로 다른 하위 트리에 서로 다른 값을 제공해야하는 경우에만 필요
            - **서로 다른 하위 트리에서 다른 값이 필요할 경우**:
                
                ```tsx
                const ThemeContext = React.createContext("light");
                
                function App() {
                  return (
                    <div>
                      <ThemeContext.Provider value="dark">
                        <Toolbar />
                      </ThemeContext.Provider>
                      <ThemeContext.Provider value="light">
                        <Sidebar />
                      </ThemeContext.Provider>
                    </div>
                  );
                }
                
                ```
                
        
    - **컨텍스트(Context)**: 서로 다른 하위 트리에 서로 다른 값을 제공해야 하는 경우 유용. 트리 구조와 강하게 연관되어 있는 경우 적합
    - **모듈 상태(Module State)**: 트리 구조와 독립적으로 상태를 관리해야 하는 경우 유용. 상태를 쉽게 구독하고, 상태 변경을 알리는 방식으로 유연하게 관리 가능

## 컨텍스트와 구독 패턴 사용하기

- 컨텍스트, 구독을 이용한 모듈 상태를 하나만 사용할 때의 문제
    - 하나의 컨텍스트를 사용해 전역 상태 값을 전파하는 것은 불필요한 리렌더링이 발생하는 문제
    - 구독을 이용한 모듈 상태는 리렌더링 문제는 없지만, 전체 컴포넌트 트리에 하나의 값만 제공한다는 문제

- 컨텍스트 생성 시 구독을 위한 모듈 상태의 `createStore`를 사용
    
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
    
    ```tsx
    type State = { count: number; text?: string };
    
    const StoreContext = createContext<Store<State>>(
      createStore<State>({ count: 0, text: "hello" })
    );
    ```
    
- 하위 트리에 서로 다른 스토어를 제공하기 위해 `StoreContext.Provider`를 간단하게 감싼 `StoreProvider`를 구현
    
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
    
    - `useRef`는 스토어 객체가 첫 번째 렌더링에서 한 번만 초기화되게 만드는 데 사용됨

- 스토어 객체를 사용하기 위해 useSelector 훅을 구현
    
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
    
    - 4장과 다르게 인수로 스토어 객체를 받지 않고, `StoreContext`에서 `store` 객체를 가져옴
    - `useContext`와 함께 `useSubscription`을 사용하는 것이 이 패턴의 핵심

- 모듈 상태와 다르게 컨텍스트를 사용해서 상태를 갱신하는 방법을 제공해야 함
    
    ```tsx
    const useSetState = () => {
      const store = useContext(StoreContext);
      
      return store.setState;
    };
    ```
    
    - `store`에서 `setState`함수를 반환하는 간단한 훅

- 사용
    
    ```tsx
    // useCallback으로 감싸지 않기 위해 컴포넌트 외부에 selectCount를 정의
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
    
    - Component 컴포넌트는 특정 스토어 객체에 연결돼 있지 않음
        - Component 컴포넌트는 다른 스토어에서 사용할 수 있음
        - 다양한 위치에서 Component를 가질 수 있음
            - 공급자 외부
            - 첫 번째 공급자 내부
            - 두 번째 공급자 내부
        - StoreProvider 외부, 첫 번째 StoreProvider 컴포넌트 내부, 두 번째로 중첩된 StoreProvider 컴포넌트 내부
            
            ```tsx
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
            ```
            
            - 각 Component는 각 store에 해당하는 count 값을 표시함