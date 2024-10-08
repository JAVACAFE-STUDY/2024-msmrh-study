# 모듈 상태 살펴보기

**모듈 상태** : 모듈 수준에서 정의된 변수. 이 책에서는 **함수 외부에서 정의된 변수**를 모듈 상태라고 가정한다.

리액트에서 상태를 다루는 일반적인 로직을 모듈 상태로 구현하면 다음과 같다.

```jsx
export const createContainer = (initialState) => {
  let state = initialState;
  const getState = () => state;
  const setState = (nextState) => {
    state = typeof nextState === "function" ? nextState(state) : nextState;
  };
  return { getState, setState };
};

//아래와 같이 사용
import { createContainer } from "...";

const { getState, setState } = createContainer({
  count: 0,
});
```

보통 리액트에서는 상태를 객체로 정의하고, 상태를 읽는 getState 함수와 상태를 쓰는 setState 함수로 상태에 접근한다. 상태 갱신은 함수와 값 두 가지 형태로 가능하다.

# 리액트에서 전역 상태를 다루기 위한 모듈 상태 사용법

전체 트리에서 전역 상태가 필요하다면 모듈 상태가 더 적합할 수 있다.

하지만 리액트 컴포넌트에서 모듈 상태를 사용하려면 **리렌더링 최적화를 직접 처리**해야 한다.

예를 들어 다음과 같은 코드가 있을 때,

```jsx
let count = 0;

const Component1 = () => {
  const inc = () => {
    count += 1;
  };

  return (
    <div>
      {count} <button onClick={inc}>+1</button>
    </div>
  );
};
```

버튼을 클릭하면 count 값은 증가하지만 컴포넌트가 리렌더링되지 않는다.

리액트에서 리렌더링을 일으키는 훅은 useState와 useReducer 두 개만 있으므로, 모듈 상태에 반응하는 컴포넌트를 만들려면 둘 중 하나를 사용해야 한다.

useState를 사용하여 위 예제를 다음과 같이 수정하면 count 값이 증가할 때 컴포넌트가 리렌더링된다.

```jsx
let count = 0;

const Component1 = () => {
  const [state, setState] = useState(count);
  const inc = () => {
    count += 1;
    setState(count);
  };

  return (
    <div>
      {state} <button onClick={inc}>+1</button>
    </div>
  );
};
```

그러나 위와 같은 수정은 여러 개의 컴포넌트가 동일한 상태를 사용 및 수정할 때 적합하지 않다.

만약 위 컴포넌트와 똑같이 count를 사용하는 다른 컴포넌트 Component2가 있을 경우, 각 컴포넌트에서 버튼을 클릭하더라도 다른 컴포넌트에 변경된 count가 반영되지 않는다.

다음과 같이 Component1와 Component2에서 동시에 setState 함수를 호출하게 하면 이를 해결할 수는 있다.

```jsx
import { useEffect, useState } from "react";

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

const Component2 = () => {
  const [state, setState] = useState(count);
  useEffect(() => {
    setStateFunctions.add(setState);
    return () => {
      setStateFunctions.delete(setState);
    }
  }, []);
  const inc2 = () => {
    count += 2;
    setStateFunctions.forEach((fn) => {
      fn(count);
    });
  };
  return (
    <div>
      {state} <button onClick={inc2}>+2</button>
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

위 예제에서는 두 컴포넌트의 setState를 리액트 외부 Set 데이터로 저장하고, 버튼을 누를 때마다 Set에 저장되어 있는 모든 setState들을 호출한다.

하지만 위와 같은 방식은 좋은 해결책이 아니다. 게다가 Component1와 Component2에는 중복 코드가 있으며 컴포넌트가 추가될수록 늘어날 것이다.

그 외에 위 해결이 바람직하지 않은 이유를 클로드에게 물어봤더니 다음과 같이 대답해 주었다.

<aside>
💡

이 해결책이 바람직하지 않은 이유는 다음과 같습니다:

1. 복잡성 증가:
   리액트의 상태 관리 철학에 반하는 방식으로, 코드가 불필요하게 복잡해집니다. 리액트는 선언적이고 예측 가능한 상태 관리를 지향하는데, 이 방식은 명령적이고 예측하기 어려운 패턴을 만듭니다.
2. 유지보수의 어려움:
   여러 컴포넌트가 공유하는 외부 상태와 Set을 사용하는 방식은 디버깅과 유지보수를 어렵게 만듭니다. 상태 변화의 추적이 어려워집니다.
3. 성능 문제:
   모든 컴포넌트의 setState를 호출하는 방식은 불필요한 리렌더링을 유발할 수 있어 성능 저하로 이어질 수 있습니다.
4. 확장성 부족:
   새로운 컴포넌트를 추가하거나 기존 컴포넌트를 수정할 때마다 setStateFunctions Set을 수동으로 관리해야 하므로 확장성이 떨어집니다.
5. 부작용 관리의 어려움:
   useEffect를 사용해 컴포넌트 마운트/언마운트 시 Set을 수동으로 관리해야 하는데, 이는 실수하기 쉽고 부작용을 초래할 수 있습니다.
6. 리액트의 단방향 데이터 흐름 위반:
   이 패턴은 리액트의 핵심 원칙인 단방향 데이터 흐름을 위반합니다. 상태 변화의 예측과 추적이 어려워집니다.
7. 테스트의 어려움:
전역 변수와 Set을 사용하는 이 방식은 단위 테스트와 컴포넌트 테스트를 복잡하게 만듭니다.
</aside>

# 기초적인 구독 추가하기

구독 방식으로 모듈 상태를 리액트의 상태에 연결할 수 있다.

구독은 갱신에 대한 알림을 받기 위한 방법으로, 일반적으로 이렇게 사용한다.

```jsx
const unsubscribe = store.subscribe(() => {
  console.log("store is updated");
});
```

위와 같이 사용하면 store가 갱신될 때마다 콜백 함수가 호출되고 로그가 출력되는 동작이 예상된다.

이 아이디어를 바탕으로 구독 기능을 제공하는 store를 구현해 보면 다음과 같다.

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
```

이전 예제에서 리액트 바깥에 정의하던 state를 이젠 store 안에서 정의하고, setState와 getState로 관리한다.

createStore를 통해 생성한 store 변수는 state를 담고 있으므로 변수 전체를 모듈 상태라고 할 수 있다.

리액트에서 이 store 변수를 활용하고자 한다면, store의 상태값과 갱신 함수를 튜플로 반환하는 사용자 정의 훅(useStore)을 만들어서 사용할 수 있다.

```jsx
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

여기서 useEffect 내에 setState()를 한 번 호출하는 부분은 에지 케이스를 다루기 위한 코드로, useEffect가 호출되기 전에 다른 곳에서 setState가 호출되어 store가 이미 새로운 객체를 가지고 있는 상황을 방지한다.

useStore를 컴포넌트에서 활용하면 다음과 같다.

```jsx
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

모듈 상태는 결국 리액트에서 갱신되기 때문에 리액트의 상태와 동일하게 모듈 상태를 불변적으로 갱신하는 것(현재 state 객체를 직접 수정하지 않고, 새 객체를 만들어 변경하는 것)이 중요하다.

# 선택자와 useSubscription 사용하기

이전 절에서 만든 useStore는 상태 객체 전체를 반환하므로, 상태 객체의 일부분만 변경되더라도 모든 useStore 훅에 전달되어 불필요한 리렌더링을 유발할 수 있다.

이를 피하기 위해 컴포넌트가 필요로 하는 상태의 일부분만 반환하는 **선택자(selector)**를 도입할 수 있다.

useStoreSelector는 다음과 같이 정의할 수 있다.

```jsx
const store = createStore({ count1: 0, count2: 0 });

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

useStoreSelector는 useStore와 비슷하지만 추가로 선택자 함수를 받는다. 또 useState에 적용하는 값으로 상태의 전체 내용(store)이 아닌 selector 함수의 반환 값을 가진다.

useStoreSelector를 통해 count1을 사용하는 컴포넌트를 만들어 보면 다음과 같다.

```jsx
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

useStoreSelector 내부에서 useEffect의 두 번째 인수로 선택자 함수가 지정되어 있으므로, 선택자 함수를 안정적으로 넘기기 위해 useCallback을 사용해야 한다. 그렇지 않으면 Component1을 렌더링할 때마다 store을 구독 해제하고 구독하는 일을 반복하게 된다.

혹은 다음과 같이 컴포넌트 외부에서 선택자 함수를 정의할 수도 있다.

```jsx
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

이렇게 구현하면 컴포넌트가 store가 변경될 때마다 리렌더링되지 않고, selector로 구독하는 값이 변경될 때에만 리렌더링된다.

useStoreSelector 훅은 잘 작동하고 실제 운영 환경에서 사용할 수 있지만, store 또는 selector가 변경되었을 때 재구독될 때까지 갱신되기 이전 상태값을 반환할 수 있다는 단점이 있다. 이는 useEffect가 조금 늦게 실행되기 때문이다.

리액트 팀은 이를 보완할 수 있는 [use-subscription](https://www.npmjs.com/package/use-subscription)이라는 공식적인 훅을 제공한다. useSubscription을 사용해 useStoreSelector를 다시 구현하면 다음과 같다.

```jsx
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

useStoreSelector 훅을 정의하지 않고 컴포넌트 내에서 직접 useSubscription을 사용할 수도 있다.

이 useSubscription 훅은 리액트 18 버전에서 [useSyncExternalStore](https://react.dev/reference/react/useSyncExternalStore)라는 이름으로 정식 릴리즈되었다.
