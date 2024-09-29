# 리액트 컨텍스트를 이용한 컴포넌트 상태 공유

## Context

- 상태와는 관련 없지만 props를 대신해서 컴포넌트 간에 데이터를 전달하는 것이 가능
- 컨텍스트를 컴포넌트 상태와 결합하면 전역 상태를 제공할 수 있다.
- 상태가 갱신될 때 모든 컨텍스트 소비자가 리렌더링 된다는 문제가 있다.

## useState와 useContext 탐구하기

### useContext 없이 useState 사용하기

- 격리 된 곳에서 공유받을 상태를 상위 컴포넌트로 부터 전달 받는다.
- 앱의 규모가 커질 수록 상태 공유로인해 props drilling이 빈번하게 일어날 수 있으며 불필요한 컴포넌트 의존도가 높아질 수 있다.

### 정적 값을 이용해 useContext 사용하기

- 컨텍스트를 사용하면 props를 사용하지 않고도 부모 컴포넌트에서 트리 아래에 있는 자식 컴포넌트로 값을 전달하는 것이 가능하다.
- 리액트 컨텍스트에는 다양한 값을 제공하는 여러 Provider가 있다. Provider는 중첩이 될 수 있으며 Consumer는 가장 가까운 Provider의 컨텍스트 값을 가져온다.

#### useContext 기본 사용 방법

```jsx
import React, { createContext, useContext } from "react";

// 컨텍스트 생성 및 defaultValue 작성
const ColorContext = createContext("red");

const Component = () => {
  // 사용하고 싶은 context를 useContext의 인자로 전달
  const color = useContext(ColorContext);
  return <div style={{ color }}>Hello {color}</div>;
};

const App = () => {
  return (
    <div>
      <Component />
      <ColorContext.Provider value="blue">
        <Component />
      </ColorContext.Provider>
      <ColorContext.Provider value="green">
        <Component />
        <ColorContext.Provider value="skyblue">
          <Component />
        </ColorContext.Provider>
      </ColorContext.Provider>
      <Component />
    </div>
  );
};

export default App;
```

#### useState와 useContext를 함께 사용하기

- 대부분의 경우 기본값이 그다지 유용하지 않기 때문에 정적인 값 대신 상태가 필요하다.
- useContext() 인자로 넘겨주는 defaultValue는 타입스크립트에서 타입을 유추하기 용이하다.

```jsx
const CountStateContext = createContext({ count: 0, setCount: () => {} });

const Child1 = () => {
  const { count, setCount } = useContext(CountStateContext);
  return (
    <div>
      Child1 {count}
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
};

const Child2 = () => {
  const { count, setCount } = useContext(CountStateContext);
  return (
    <div>
      Child2 {count}
      <button onClick={() => setCount(count + 2)}>+2</button>
    </div>
  );
};

const Parent = () => {
  const { count, setCount } = useContext(CountStateContext);
  return (
    <>
      Parent {count} <button onClick={() => setCount(count - 1)}>-1</button>
      <div>
        <Child1 />
        <Child2 />
      </div>
    </>
  );
};

const App = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <CountStateContext.Provider value={{ count, setCount }}>
        <Parent />
      </CountStateContext.Provider>
    </div>
  );
};

export default App;
```

- Provider를 사용해 격리된 카운트 상태를 제공할 수 있다.

## 컨텍스트 이해하기

### 컨텍스트 전파 작동 방식

- Provider가 새로운 컨텍스트 값을 갖게 되면 모든 컨텍스트 Consumer는 새로운 값을 받고 리렌더링 된다.
- 이는 Provider의 공급자의 값이 모든 소비자에게 전파됨을 의미한다.
- 자식 컴포넌트가 리렌더링 되는 이유는 부모 컴포넌트와 컨텍스트 때문이다.
- 컨텍스트가 변경되지 않았는데 리렌더링이 되는 것을 방지하려면 (즉, 부모컴포넌트의 리렌더링에 영향을 받지 않으려면) 내용 끌어올리기 또는 memo를 사용하면 된다.
- 다만 memo는 Context Consumer가 리렌더링되는 것은 막지 못한다. (그렇지 않으면 일관되지 않은 컴텍스트 값을 가질 수 있기 때문)
- 💡 [React memo](https://ko.react.dev/reference/react/memo)

### 컨텍스트에 객체를 사용할 때 한계점

```JSX
const CountContext = createContext({ count1: 0, count2: 0 });
```

- 객체에는 여러 가지 값을 포함할 수 있으며, Consumer는 모든 값을 사용하지 않을 수 있다.
- 특정 컴포넌트 내에서 Provider가 제공하는 객체의 값 중 일부만 소비함에도 불구하고 컴포넌트 내에서 사용하지 않는 컨텍스트 값이 변경되면 해당 컴포넌트가 리렌더링 되는 문제가 있다.
- 💡 추가적인 리렌더링

  추가적인 리렌더링은 기술적으로 피해야 하는 불필요한 연산이지만, 사용자가 추가 리렌더링을 알아차리지 못하는 경우가 많아 성능에 큰 문제가 없다면 대체로 괜찮다. 몇 번의 추가 리렌더링을 피하기 위해 오버엔지니어링을 하는 것은 현실적으로 해결할 가치가 없을 수도 있다.

## 전역 상태를 위한 컨텍스트 만들기 (불필요한 리렌더링 피하기)

### 작은 상태 조각 만들기

- 합쳐진 큰 객체의 사용 대신, 각 조각에 대한 컨텍스트와 전역 상태를 만든다.

### useReducer로 하나의 상태를 만들고 여러 컨텍스트 전파하기

[예시](https://github.com/wikibook/msmrh/blob/main/chapter03/07_creating-one-state-with-userreducer-and-propagate-with-multiple-contexts/src/App.tsx)

- 단일 상태를 만들고 여러 컨텍스트를 사용해 상태 조각을 배포한다.
- 상태를 갱신하는 dispatch 함수를 배포하는 것은 별도의 컨텍스트로 한다.
- 하나의 액션으로 여러 상태 조각을 갱신할 수 있다는 장점이 있다.

## 컨텍스트 사용을 위한 모범 사례

### Custom hook과 Provider 컴포넌트 만들기

```JSX
type CountContextType = [number, Dispatch<SetStateAction<number>>];

const Count1Context = createContext<CountContextType | null>(null);

export const Count1Provider = ({ children }: { children: ReactNode }) => (
  <Count1Context.Provider value={useState(0)}>
    {children}
  </Count1Context.Provider>
);

export const useCount1 = () => {
  const value = useContext(Count1Context);
  if (value === null) throw new Error("Provider missing");
  return value;
};
```

- 위 예제와 같이 작성 시 도메인에 맞는 value가 들어가야하는 Provider와 Provider로 감싸지 않고 컨텍스트를 사용할 때 발생하는 에러를 명시적으로 지정해 줄 수 있다.

### Custom hook이 있는 팩토리 패턴

```JSX
const createStateContext = <Value, State>(
  useValue: (init?: Value) => State
) => {
  const StateContext = createContext<State | null>(null);
  const StateProvider = ({
    initialValue,
    children,
  }: {
    initialValue?: Value;
    children?: ReactNode;
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

- Context를 활용할 때 특정 상태 관리를 위해 명시적인 Custom hook 과 Provider를 만드는 작업을하는 함수를 만들어 반복 작업을 줄일 수 있다.

### reduceRight을 이용한 공급자 중첩 방지

```JSX
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

- reduceRight을 활용해 중첩되는 컴포넌트를 간단하게 작성해볼 수 있다.
