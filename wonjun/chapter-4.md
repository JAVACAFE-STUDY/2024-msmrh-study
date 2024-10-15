# 4장 : 구독을 이용한 모듈 상태 공유

**모듈 상태란?**

ECMAScript 모듈 스코프에 정의된 상수 / 변수

# **모듈 상태 살펴보기**

```
let a = 0
```

이 상태는 모듈 상태일까?

만약 이 코드가 모듈에 정의되어 있다면 O 라고 말할 수 있다.

```
export const createContainer = (initialState) => {
  let state = initialState;
  const getState = () => state;
  const setState = (nextState) => {
    state = typeof nextState === 'function' ? nextState(state) : nextState;
  }

  return { getState, setState }
}
```

그래서 이렇게 상태를 관리하는 모듈을 만들 수 있다.

# **React 에서의 전역 상태를 위한 모듈 상태**

```
let a = 0
```

위와 같은 코드이다. 이는 JS 환경에서는 동작하지만, React 에서 동작하지 않는다.

```tsx
let GlobalCount = 0;

function NotWork() {
  const [count, setCount] = useState(0);

  function inc() {
    GlobalCount += 1;
    setCount(GlobalCount);
  }

  return (
    <button
      onClick={() => {
        inc();
      }}
    >
      되겠냐
    </button>
  );
}

function NotWorkToo() {
  const [count, setCount] = useState(0);

  function inc() {
    GlobalCount += 1;
    setCount(GlobalCount);
  }

  return (
    <button
      onClick={() => {
        inc();
      }}
    >
      되겠냐고
    </button>
  );
}
```

**tsx**

이 코드도 동작하지 않는다.

Set 등을 사용해서 구현할 수 있지만, 이런 코드가 더 많아질 경우 머리가 아파지기 시작할 것이다.

# **구독 추가하기**

```
export const createStore = (initialState) => {
  let state = initialState;
  const callbacks = new Set();

  const getState = () => state;

  const setState = (nextState) => {
    state = typeof nextState === 'function' ? nextState(state) : nextState;
    callbacks.forEach((callback) => callback())
  }

  const subscribe = (callback) => {
    callbacks.add(callback);
    return () => {
      callbacks.delete(callback)
    }
  }

  return { getState, setState, subscribe }
}
```

이는 위에서 정의한 `createContainer` 와 동일하지만, `subscribe` 라는 값을 구독하는 함수가 있는 것이 특징이다.

JS 환경에서는 다음과 같이 사용할 수 있다.

```
const store = createStore({ count: 0 });
console.log(store.getState());
store.setState({ count: 1 })
store.subscribe(() => {
  console.log(store.getState())
})
```

React 환경에서는 `useStore` 라는 hook을 정의함으로써 더욱 쉽게 사용할 수 있다.

```
const useStore = (store) => {
  const [state, setState] useState(store.getState());

  useEffect(function handleSubscribe() {
    const unsubscribe = store.subscribe(() => {
      setState(store.getState())
    })

    setState(store.getState());

    return unsubscribe
  }, [store])

  return [state, store.setState]
}
```

방금 우리는 전역 상태 관리 라이브러리를 만들었다. 🎉

```tsx
const countStore = createStore({ count: 0 });

function A() {
  const [count, setCount] = useStore(countStore);

  function inc() {
    setCount((p) => p + 1);
  }

  return (
    <button
      onClick={() => {
        inc();
      }}
    >
      {count}
    </button>
  );
}

function B() {
  const [count, setCount] = useStore(countStore);

  function inc() {
    setCount((p) => p + 1);
  }

  return (
    <button
      onClick={() => {
        inc();
      }}
    >
      {count}
    </button>
  );
}
```

**tsx**

이렇게 사용할 수 있다.

# **선택자**

위에서 정의한 `useStore` 는 함정이 있는데, 객체를 반환하다보니 객체의 내용이 바뀌면 '모두' 리렌더링 된다는 점이다.

이를 해결하기 위해 특정 값만 구독하는 기능을 추가해보자.

```
const useStoreSelector = (store, selector) => {
  const [state, setState] = useState(() => selector(store.getState()))

  useEffect(function subscribeStoreValue() {
    const unsubscribe = store.subscribe(() => {
      setState(selector(store.getstate()));
    });

    setState(selector(store.getstate()));

    return unsubscribe;
  }, [store, selector])

  return state;
}
```

이제 이렇게 사용할 수 있다.

```tsx
const countStore = createStore({ count: 0 });

const selectCount = (store) => store.count;

function A() {
  const [count, setCount] = useStoreSelector(countStore, selectCount);

  function inc() {
    setCount((p) => p + 1);
  }

  return (
    <button
      onClick={() => {
        inc();
      }}
    >
      {count}
    </button>
  );
}
```

**tsx**

사소한 문제는 useEffect가 늦게 실행되기 때문에 재구독 되기 전에는 이전 값을 반환한다는 점이다.

# **useSyncExternalStore**

물론 React 팀에서는 이 문제를 해결하기 위한 `useSyncExternalStore` 이라는 hook을 이미 만들어두었다.

```tsx
const useStoreSelector = (store, selector) =>
  useSyncExternalStore(
    useMemo(
      () => ({
        getCurrentValue: () => selector(store.getState()),
        subscribe: store.subscribe,
      }),
      [store, selector]
    )
  );
```

**tsx**
