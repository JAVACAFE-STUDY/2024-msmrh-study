# React 컨텍스트 심화 학습 가이드

## 1. 컨텍스트 소개

React 컨텍스트는 컴포넌트 간에 데이터를 효과적으로 전달할 수 있는 강력한 기능입니다. 이는 props를 대신하여 컴포넌트 간 데이터 전달을 가능하게 합니다.

### 1.1 컨텍스트의 장점

- props 드릴링 방지
- 컴포넌트 간 효율적인 데이터 전달
- 컴포넌트 상태와 결합하여 전역 상태 제공 가능

> 💡 컨텍스트의 단점 : 종속성이 강해진다.
>
> - 컴포넌트 재사용성 감소: 컨텍스트를 사용하는 컴포넌트는 해당 컨텍스트에 의존하게 되어, 다른 상황에서 재사용하기 어려워질 수 있습니다.

- 과도한 리렌더링: 컨텍스트 값이 변경될 때마다 해당 컨텍스트를 사용하는 모든 컴포넌트가 리렌더링됩니다. 이는 성능 문제를 야기할 수 있습니다.
- 복잡성 증가: 많은 컨텍스트를 사용하면 코드의 복잡성이 증가하고, 데이터 흐름을 추적하기 어려워질 수 있습니다.
- 테스트의 어려움: 컨텍스트를 사용하는 컴포넌트를 테스트할 때, 해당 컨텍스트 프로바이더로 감싸야 하므로 테스트 설정이 복잡해질 수 있습니다.
- 깊은 중첩: 여러 컨텍스트를 사용할 경우, 프로바이더들의 중첩이 깊어져 코드의 가독성이 떨어질 수 있습니다.
- 최적화의 어려움: 컨텍스트를 사용하면 React의 메모이제이션 최적화(React.memo, useMemo 등)가 효과적으로 작동하지 않을 수 있습니다.
- 부주의한 사용으로 인한 성능 저하: 잘못 설계된 컨텍스트는 불필요한 리렌더링을 유발하여 애플리케이션의 성능을 저하시킬 수 있습니다.
- 디버깅의 어려움: 컨텍스트를 통한 상태 변화는 명시적이지 않아, 디버깅이 어려울 수 있습니다.

### 1.2 prop drilling

prop drilling은 중간 컴포넌트가 해당 props가 필요하지 않음에도 오직 하위 컴포넌트에 전달하기 위한 목적으로 props를 받아 전달하는 경우를 말합니다. 컨텍스트를 사용하면 이러한 문제를 해결할 수 있습니다.

## 2. useContext 훅

`useContext` 훅은 함수 컴포넌트에서 컨텍스트를 쉽게 사용할 수 있게 해줍니다.

### 2.1 useContext 사용 예제

```jsx
const CountContext = createContext < CountContextType > [0, () => {}];

function Counter() {
  const [count, setCount] = useContext(CountContext);
  return (
    <div>
      Count: {count}
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

이 예제에서 `Counter` 컴포넌트는 `useContext`를 사용하여 `CountContext`의 값을 직접 접근합니다. 소비자 컴포넌트(useContext가 있는 컴포넌트)는 컴포넌트 트리 중에서 가장 가까운 공급자를 선택해 컨텍스트 값을 가져옵니다.

## 3. 컨텍스트 전파의 작동 방식

자식 컴포넌트가 리렌더링되는 이유는 두 가지입니다:

1. 부모 컴포넌트 때문
2. 컨텍스트 때문

컨텍스트 공급자가 새로운 컨텍스트 값을 갖게 되면 모든 컨텍스트 소비자는 새로운 값을 받고 리렌더링됩니다.

P.55

- 처음에 모든 컴포넌트가 렌더링된다.
- 텍스트 입력 필드에서 값을 변경하면 useState 때문에 App 컴포넌트가 리렌더링된다.
- ColorContext.Provider는 새로운 값을 받고 동시에 Parent 컴포넌트가 렌더링된다.
- DummyComponent는 리렌더링되지만 MemoedDummyComponent는 리렌더링되지 않는다.
- ColorComponent는 두 가지 이유로 렌더링된다. 첫 번째로 부모가 리렌더링됐고, 두 번째로 컨텍스트가 변경됐다.
- MemoedColorComponent는 컨텍스트가 변경됐기 때문에 리렌더링된다.

## 4. 컨텍스트에 객체를 사용할 때의 한계점

```jsx
const CountContext = createContext({ count1: 0, count2: 0 });

function Counter1() {
  const { count1 } = useContext(CountContext);
  return <div>Count 1: {count1}</div>;
}

function Counter2() {
  const { count2 } = useContext(CountContext);
  return <div>Count 2: {count2}</div>;
}
```

이 예제에서 `count1`과 `count2` 카운트는 완전히 분리되어 있습니다. 그러나 만약 `count1`이 변경되지 않았는데도 `Counter1`이 리렌더링된다면, 이는 불필요한 리렌더링을 의미합니다. 이 예제에서는 `count2`만 변경돼도 `Counter1`이 리렌더링됩니다.

## 5. 전역 상태를 위한 컨텍스트 만들기

### 5.1 작은 상태 조각 만들기

전역 상태를 여러 조각으로 나누는 것이 효과적일 수 있습니다.

```jsx
const Count1Context = createContext < CountContextType > [0, () => {}];
const Count2Context = createContext < CountContextType > [0, () => {}];

function Count1Provider({ children }) {
  return (
    <Count1Context.Provider value={useState(0)}>
      {children}
    </Count1Context.Provider>
  );
}

function Count2Provider({ children }) {
  return (
    <Count2Context.Provider value={useState(0)}>
      {children}
    </Count2Context.Provider>
  );
}
```

이 예제에서는 `Count1Provider`와 `Count2Provider` 컴포넌트가 유사하며 값을 제공하는 컨텍스트만 다릅니다. 이렇게 하면 `Counter1`과 `Counter2` 컴포넌트는 각각 `count1`과 `count2`가 변경될 때만 리렌더링됩니다.

count1, count2의 상태는 분리되어있지만 컨텍스트는 하나로 구성되어있기 때문에, 상태는 count1만 변함에도 count2의 컴포넌트에도 불필요한 리렌더링이 발생한다. 따라서 컨텍스트를 '여러 조각'으로 나눌 필요가 있다.

### 5.2 useReducer로 하나의 상태를 만들고 여러 개의 컨텍스트로 전파하기

단일 상태를 만들고 여러 컨텍스트를 사용해 상태 조각을 배포하는 방법도 있습니다.

```jsx
type Action = { type: "INC1" } | { type: "INC2" };

const Count1Context = createContext < number > 0;
const Count2Context = createContext < number > 0;
const DispatchContext = createContext < Dispatch < Action >> (() => {});

function countReducer(state, action: Action) {
  switch (action.type) {
    case "INC1":
      return { ...state, count1: state.count1 + 1 };
    case "INC2":
      return { ...state, count2: state.count2 + 1 };
    default:
      return state;
  }
}

function CountProvider({ children }) {
  const [state, dispatch] = useReducer(countReducer, { count1: 0, count2: 0 });

  return (
    <Count1Context.Provider value={state.count1}>
      <Count2Context.Provider value={state.count2}>
        <DispatchContext.Provider value={dispatch}>
          {children}
        </DispatchContext.Provider>
      </Count2Context.Provider>
    </Count1Context.Provider>
  );
}
```

이 예제에서는 한 컨텍스트는 값에 대해, 다른 컨텍스트는 실행 함수를 위해 사용됩니다. 리듀서 때문에 코드가 길어졌지만 요점은 중첩된 공급자가 각 상태 조각과 하나의 실행 함수를 제공한다는 것입니다.

## 6. 컨텍스트 사용을 위한 모범 사례

### 6.1 사용자 정의 훅과 공급자 컴포넌트 만들기

```typescript
type CountContextType = [number, Dispatch<SetStateAction<number>>];

const Count1Context = createContext<CountContextType | null>(null);

export const Count1Provider = ({ children }: { children: ReactNode }) => (
  <Count1Context.Provider value={useState(0)}>
    {children}
  </Count1Context.Provider>
);

export const useCount1 = () => {
  const context = useContext(Count1Context);
  if (context === null) {
    throw new Error("useCount1 must be used within a Count1Provider");
  }
  return context;
};
```

### 6.2 사용자 정의 훅이 있는 팩토리 패턴

```typescript
const createStateContext = <State>(useValue: (init: State) => State) => {
  const StateContext = createContext<State | null>(null);

  const StateProvider = ({
    initialValue,
    children,
  }: {
    initialValue: State;
    children: ReactNode;
  }) => (
    <StateContext.Provider value={useValue(initialValue)}>
      {children}
    </StateContext.Provider>
  );

  const useContextState = () => {
    const value = useContext(StateContext);
    if (value === null) throw new Error("Provider missing");
    return value;
  };

  return [StateProvider, useContextState] as const;
};
```

이 팩토리 패턴은 타입스크립트에서 잘 작동합니다. `createStateContext` 함수는 상태를 가져오는 사용자 정의 훅과 공급자 컴포넌트 튜플을 반환합니다.

### 6.3 reduceRight를 이용한 공급자 중첩 방지

```jsx
const App = () => {
  const providers = [
    [Count1Provider, { initialValue: 10 }],
    [Count2Provider, { initialValue: 20 }],
    [Count3Provider, { initialValue: 30 }],
    [Count4Provider, { initialValue: 40 }],
    [Count5Provider, { initialValue: 50 }],
  ] as const;

  return providers.reduceRight(
    (children, [Component, props]) => createElement(Component, props, children),
    <Parent />
  );
};
```

이 방법을 사용하면 여러 개의 Provider를 간단하게 중첩할 수 있습니다.

## 결론

React 컨텍스트는 강력한 상태 관리 도구입니다. 적절히 사용하면 prop drilling을 피하고 효율적으로 데이터를 관리할 수 있습니다. 그러나 과도한 사용은 컴포넌트의 재사용성을 저해할 수 있으므로 주의가 필요합니다. 상황에 따라 적절한 패턴을 선택하여 사용하는 것이 중요합니다.
