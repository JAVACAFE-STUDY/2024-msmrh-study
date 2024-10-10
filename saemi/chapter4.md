# 구독을 이용한 모듈 상태 공유

## 모듈상태

모듈 상태 - 모듈 수준에서 정의된 변수, 모듈은 ES 모듈 또는 파일을 의미

## 리액트에서 전역 상태를 다루기 위한 모듈 상태 사용법

- 전체 트리에서 전역 상태가 필요하다면 컨텍스트보다 모듈 상태가 더 적합할 수 있다.
- 리액트 컴포넌트에서 모듈 상태를 사용하려면 리렌더링 최적화를 직접 처리해야한다.

## 기초적인 구독 추가하기

### 구독의 일반적인 사용법

```tsx
const unsubscribe = store.subscribe(() => {
  console.log("store is updated");
});
```

### 구독으로 모듈 상태 구현하기

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

    // 각 지역 상태를 일괄 업데이트시켜 리렌더링을 유발하도록 하기 위함
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
const useStore = (store) => {
  const [state, setState] = useState(store.getState());

  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      // store의 state가 업데이트 될 때 store를 구독하고 있는 각 상태들을 일괄 업데이트함으로써 리렌더링을 유발하게함
      setState(store.getState());
    });

    // useEffect가 뒤늦게 실행되어 store가 이미 새로운 상태를 가지고 있는 엣지 케이스를 다루기 위한 코드
    setState(store.getState());
    return unsubscribe;
  }, [store]);

  return [state, store.setState];
};
```

- 모듈 상태는 리액트에서 갱신되기 때문에 리액트의 상태와 동일하게 모듈 상태를 불변적으로 갱신하는 것이 중요!

## 선택자와 useSubscription 사용하기

필요 없는 리렌더링을 피하기 위해 컴포넌트가 필요로하는 상태의 일부분만 반환하는 선택자(selector)를 도입할 수 있다.

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

const Component = () => {
  const state = useStoreSelector(
    store,
    useCallback((state) => state.count1)
  );

  ...
};
```

- 안정적으로 선택자 함수를 넘기려면 useCallback을 사용해야한다. 그렇지 않으면 Component를 렌더링할 때 마다 store를 구독 해제하고 구독하는 것을 반복하게 된다.
- useStoreSelector 훅은 잘 작동하고 사용할 수 있지만 useEffect는 조금 늦게 실행되기 때문에 store 또는 selector가 변경될 때 갱신되기 이전 상태 값을 반환한다.
  이 때 리액트 공식 훅 useSubscription 혹은 useSyncExternalStore를 사용해서 해결할 수 있다.

## useSyncExternalStore로 로컬스토리지 상태 갱신 시 리렌더링 유발하기

- 로컬스토리지에 컴포넌트 렌더링에 필요한 상태를 저장하여 관리할 때, 로컬스토리지의 상태 변화와 리렌더링은 관련이 없기 때문에 useState를 하나 더 두고 로컬스토리지 상태와 싱크를 맞춰줘야한다.
- 상태가 변경될 때 마다 렌더링을 위한 상태 업데이트 코드를 한 번 더 작성해야하는 것이 비효율적이라고 생각해 리렌더링이 되는 로컬스토리지 상태 관리 훅을 만들어보았다.
- 참고 - https://ko.react.dev/reference/react/useSyncExternalStore

```tsx
type Storage<T> = {
  getState: () => T;
  setState: (action: T | ((prev: T) => T)) => void;
  subscribe: (callback: () => void) => () => void;
};

export const createLocalStorage = <T,>(
  key: string,
  initialState: T
): Storage<T> => {
  let state = initialState;
  const listeners = new Set<(value: T) => void>();

  const setState = (newValue: T | ((prev: T) => T)) => {
    state =
      typeof newValue === "function"
        ? (newValue as (prev: T) => T)(state)
        : newValue;
    listeners.forEach((listener) => listener(state));
    localStorage.setItem(key, JSON.stringify(state));
  };

  const getState = () => state;

  const subscribe = (listener: (value: T) => void) => {
    listeners.add(listener);
    return () => {
      listeners.delete(listener);
    };
  };

  return { setState, getState, subscribe };
};
```

```tsx
const useLocalStorage = <T,>(store: Storage<T>) => {
  const state = useSyncExternalStore(store.subscribe, store.getState);

  return [state, store.setState] as const;
};
```

```tsx
const latestSearchKeywordsStore = createLocalStorage<string[]>(
  "LATEST_SEARCH_KEYWORDS",
  []
);
```

사용 예시

```tsx
const SearchList = () => {
  const navigate = useNavigate();
  const [searchKeyword, setSearchKeyword] = useState("");
  //   const [latestSearchList, setLatestSearchList] = useState(
  //     JSON.parse(
  //       getLocalStorageItem(LOCAL_STORAGE_KEYS.LATEST_SEARCH_KEYWORDS)
  //     ) || []
  //   );
  const [latestSearchList, setLatestSearchList] = useLocalStorage(
    latestSearchKeywordsStore
  );

  const submitSearchKeyword = (searchKeyword: string) => {
    const filteredLatestSearchList = latestSearchList.filter(
      (keyword) => keyword !== trimmedSearchKeyword
    );
    const updatedLatestSearchList = [
      trimmedSearchKeyword,
      ...filteredLatestSearchList,
    ];
    setLatestSearchList(updatedLatestSearchList);
    // setLocalStorageItem(
    //   LOCAL_STORAGE_KEYS.LATEST_SEARCH_KEYWORDS,
    //   JSON.stringify(updatedLatestSearchList)
    // );
    setSearchKeyword("");
  };

  const deleteLatestSearchKeyword = (searchKeyword: string) => {
    const filteredLatestSearchList = latestSearchList.filter(
      (keyword) => keyword !== searchKeyword
    );
    setLatestSearchList(filteredLatestSearchList);
    // setLocalStorageItem(
    //   LOCAL_STORAGE_KEYS.LATEST_SEARCH_KEYWORDS,
    //   JSON.stringify(filteredLatestSearchList)
    // );
  };

  return (
    <>
      <form
        onSubmit={(e) => {
          e.preventDefault();
          submitSearchKeyword(searchKeyword);
        }}
      >
        <input
          value={searchKeyword}
          onChange={(e) => {
            setSearchKeyword(e.target.value);
          }}
        />
      </form>
      <p>최근 검색어</p>
      <ul>
        {latestSearchList.map((searchKeyword) => (
          <ListButton
            key={searchKeyword}
            onClick={() => submitSearchKeyword(searchKeyword)}
            onDelete={() => deleteLatestSearchKeyword(searchKeyword)}
          >
            {searchKeyword}
          </ListButton>
        ))}
      </ul>
    </div>
  );
};
```

- 상태가 바뀔 때 마다 여러 컴포넌트에서 동일한 상태를 렌더링할 수 있게된다.
