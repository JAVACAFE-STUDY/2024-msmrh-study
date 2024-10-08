- 컨텍스트는 싱글턴 패턴을 위해 설계된 것이 아닌 싱글턴 패턴을 피하고 각 하위 트리에 서로 다른 값을 제공하기 위한 기능
- 전역 상태를 싱글턴과 유사하게 만들고 싶다면 모듈 상태를 사용하는 것이 싱글턴 값으로 메모리에 할당되기 때문에 더 좋음

- 모듈 상태
    - ECMAScript 모듈 스코프에 정의된 상수 또는 변수

## 모듈 상태 살펴보기

- 모듈 상태(module state)는 모듈 수준에서 정의된 변수
    - 모듈은 ES 모듈 또는 파일을 의미
- 모듈 상태를 직접 정의하는 대신 상태와 상태에 접근할 수 있는 함수가 내부에 있는 컨테이너를 만들고, 컨테이너를 생성하는 함수를 만들기

```tsx
export const createContainer = (initialState) => {
	let state = initialState;
	
	const getState = () => state
	
	const setState = (nextState) => {
		state = typeof nextState === 'function' ? nextState(state) : nextState
	}

	return { getState, setState };	
}
```

### 리액트에서 전역 상태를 다루기 위한 모듈 상태 사용법

- 리렌더링 최적화를 직접 처리해야 함

- 리렌더링을 일으키는 훅은 useState와 useReducer 두 개만 있으므로 모듈 상태에 반응하는 컴포넌트를 만들려면 둘 줄 하나를 사용해야 함

```tsx
let count = 0;

const Component1 = () => {
	const [state, setState] = useState(count);
	
	const inc = () => {
		count += 1;
		setState(count);	
	}
	
	return (
		<div>{state} <button onClick={inc}>+1</button></div>
	);
};
```

```tsx
const Component2 = () => {
	const [state, setState] = useState(count);
	...
};
```

- Component1에서 버튼을 클릭해도 Component2가 리렌더링되지 않음, Component2에서 버튼을 클릭해야만 리렌더링되고 최신 모듈 상태가 표시됨

- 해결책
    - 컴포넌트 생명 주기를 고려하면서 컴포넌트 외부에 있는 모듈 수준에서 setState를 관리
    - useEffect 훅을 이용해 setState를 리액트 외부에 Set과 같은 별도의 자료구조에 추가해서 관리
    
    ```tsx
    let count = 0;
    const setStateFunctions = new Set<(count: number) =>void>();
    
    const Component1 = () => {
      const [state, setState] = useState(count);
      
      useEffect(() => {
        setStateFunctions.add(setState);
        return () => {
          setStateFunctions.delete(setState);
        }
      }, []);
      
      const inc = () => {
        count += 1;
        setStateFunctions.forEach((fn) => {
          fn(count);
        });
      };
      
      return (
        <div>
          {state} <button onClick={inc}>+1</button>
        </div>
      );
    };  
    ```
    
    - useEffect의 부수 효과를 정리하기 위해 useEffect에서 함수를 반환
    - inc 함수에서 setStateFunctions 집합을 통해 모든 setState 함수를 호출
    - Component1, Component2에 중복 코드가 있어서 좋은 해결책은 아님

## 기초적인 구독 추가하기

- 구독
    - 갱신에 대한 알림을 받는 방법
    
    ```tsx
    const unscribe = store.subscribe(() => {
    	console.log(`store is updated`);
    });
    ```
    
    - `store`에 `subscribe` 메서드가 있고 이 메서드는 `callback` 함수를 받고 `unscribe` 함수를 반환
    - `store`가 갱신될 때마다 콜백 함수가 호출되고 로그가 출력됨

- createStore 사용
      
    ```tsx
    const store = createStore(0);
    ```

- `getState`, `setState` 메서드 외에도 `state` 값과 `subscribe` 메서드를 보유한 객체를 `store`로 지칭, `createStore`는 초기 상태 값을 통해 `store`를 생성하는 함수
    
    ```tsx
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
    ```
    
    - 콜백을 호출하는 `subscribe` 메서드와 `setState` 메서드가 있음
    - `store` 변수는 `state`를 담고 있기에 `store` 변수 전체를 모듈 상태라고 볼 수 있음

- `store`의 상태 값과 갱신 함수를 튜플로 반환하는 새로운 사용자 정의 훅인 `useStore`를 정의
    
    ```tsx
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
    ```
    
    - `setState(store.getState());`
        - `useEffect`가 뒤늦게 실행돼서 store가 이미 새로운 상태를 가지고 있을 가능성이 있기 때문에
            - `useEffect`에서 `setState()` 함수를 한 번 호출 함
                            
    
    ```tsx
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
    ```
    
    - 모듈 상태는 결국 리액트에서 갱신되기 때문에 리액트의 상태와 동일하게 모듈 상태를 불변적으로 갱신해야 함

## 선택자와 useSubscription 사용하기

- `useStore`는 상태 객체 전체를 반환하기 떄문에 상태 객체에서 일부분만 변경되더라도 모든 `useStore` 훅에 전달되기 때문에 불필요한 리렌더링을 발생시킬 수 있음
- 컴포넌트가 필요로 하는 상태의 일부분만 반환하는 선택자(`selector`) 도입

- `store` 만들기
    
    ```tsx
    const store = createStore({ count1: 0, count2: 0 });
    ```
    

- `useStoreSelector`
    - 선택자 함수를 받고, 선택자 함수를 사용해 상태의 범위를 지정함
    
    ```tsx
    const useStoreSelector = <T, S>(store: Store<T>, selector: (state: T) => S) => {
      const [state, setState] = useState(() => selector(store.getState()));
    
      useEffect(() => {
        const unsubscribe = store.subscribe(() => {
          setState(selector(store.getState()));
        });
    
        setState(selector(store.getState()));
    
        return unsubscribe;
      }, [store, selector]);
    
      return state;
    };
    
    ```
    

- `useStoreSelector` 사용
    - `useCallback`을 사용해 선택자 함수를 정의
        
        ```tsx
        const Component1 = () => {
          const state = useStoreSelector(
            store,
            useCallback((state) => state.count1, [])
          );
          
          const inc = () => {
            store.setState((prev) => ({
              ...prev,
              count1: prev.count1 + 1,
            }));
          };
          
          return (
            <div>
              count1: {state} <button onClick={inc}>+1</button>
            </div>
          );
        };
        ```
        
        - `useEffect`의 두 번째 인수(의존성 배열)에 선택자 함수가 지정돼 있으므로 선택자 함수를 바꾸지 않더라도 컴포넌트를 렌더링할 때마다 `store`를 구독 해제하고 구독하는 것을 반복하게 되므로 안정적으로 선택자 함수를 넘기려면 `useCallback`을 사용
    - `useCallback` 사용 대신 컴포넌트 외부에 선택자 함수를 정의
        
        ```tsx
        const selectCount2 = (state: ReturnType<typeof store.getState>) => state.count2;
        
        const Component2 = () => {
          const state = useStoreSelector(store, selectCount2);
        
          const inc = () => {
            store.setState((prev) => ({
              ...prev,
              count2: prev.count2 + 1,
            }));
          };
        
          return (
            <div>
              count2: {state} <button onClick={inc}>+1</button>
            </div>
          );
        };
        ```
        

- `useEffect`는 조금 늦게 실행되기 때문에 재구독 될때까지는 갱신되기 이전 상태 값을 반환함
    - 리액트에서 `use-subscription`이라는 공식적인 훅을 제공
        
        ```tsx
        import { useSubscription } from "use-subscription";
        
        const useStoreSelector = <T, S>(store: Store<T>, selector: (state: T) => S) =>
          useSubscription(
            useMemo(
              () => ({
                getCurrentValue: () => selector(store.getState()),
                subscribe: store.subscribe,
              }),
              [store, selector]
            )
          );
        ```
        
    - 직접 useSubscription을 사용
        
        ```tsx
        const Component1 = () => {
          const state = useSubscription(useMemo(() => ({            
            getCurrentValue: () => store.getState().count1,        
            subscribe: store.subscribe,                            
          }), []));                                           
        
          const inc = () => {
            store.setState((prev) => ({
              ...prev,
              count1: prev.count1 + 1,
            }));
          };
          
          return (
            <div>
              count1: {state} <button onClick={inc}>+1</button>
            </div>
          );
        };
        ```
        
        - `useMemo`를 사용했기 때문에 `useCallback`은 불필요
    - 리액트 18버전에서는 [`useSyncExternalStore`](https://ko.react.dev/reference/react/useSyncExternalStore)