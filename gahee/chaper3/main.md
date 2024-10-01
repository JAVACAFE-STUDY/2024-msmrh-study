# 리액트를 이용한 컴포넌트 상태 공유

Context란?

- 리액트 16.3버전부터 등장
- props를 대신해 컴포넌트 간에 데이터를 전달할 수 있다.
- 단, 하나의 상태라도 갱신된다면, 컨텍스트를 사용하고 있는 컴포넌트는 재렌더링이 일어날 수 있다.

## useState와 useContext 탐구하기

간략한 목차

- useState만을 사용한 상태 전달 (우리 모두가 아는것..)
- 정적 값에 대해 useContext가 어떻게 동작하는지?
- useState와 useContext함께 사용하기

### useContext없이 useState 사용하기

- 상태 끌어올리기를 사용한다.

```jsx
const App = () => {
  const [count, setCount] = useState(0);
  return <Parent count={count} setCount={setCount} />;
};

const Parent = ({ count, setCount }) => {
  return (
    <>
      <Component1 count={count} setCount={setCount} />
      <Component2 count={count} setCount={setCount} />
    </>
  );
};

const Component1 = ({ count, setCount }) => {
  return <button onClick={() => setCount(count + 1)}>증가</button>;
};

const Component1 = ({ count, setCount }) => {
  return <button onClick={() => setCount(count + 2)}>증가</button>;
};
```

- Parent 컴포넌트는 count와 setCount를 사용하지 않지만, props로 받아서 자식 컴포넌트에 전달한다.
  그러므로, props drilling이 발생한다.

### 정적 값을 이용해 useContext 사용하기

context를 사용하면, props drilling을 해결할 수 있다.
Context에는 공급자와 소비자가 있다.
이 공급자는 중첩될 수 있으며, 소비자는 컴포넌트 트리 중에서 가장 가까운 공급자를 찾아 사용한다.
[useContext 사용예제](https://codesandbox.io/p/sandbox/hz9spk)

### useContext와 함께 useState 사용하기

[useContext와 useState 사용예제](https://codesandbox.io/p/sandbox/d2jl9g)

이제 useState만을 사용했을때와 달리 Parent 컴포넌트가 App의 컨텍스트 공급자에 있더라도 count의 상태의 존재에 대해 신경쓰지 않아도 된다.
즉, props drilling을 해결할 수 있다.

## 컨텍스트 이해하기

자식 컴포넌트가 리렌더링 되는 두가지 원인

- 부모 컴포넌트가 리렌더링 되었을 때
- Context의 값이 변경되었을 때

즉, Context는 공급자(Provider)의 상태가 변경 되면, 모든 소비자(Consumer)는 새로운 값을 받고 재렌더링이 일어난다.

### 컨텍스트 전파의 작동 방식

그렇다면, 리렌터링을 방지하는 방법은 무엇일까?

- 내용 끌어올리기
- React.memo 사용하기

[React.memo 사용예제](https://codesandbox.io/p/sandbox/y752w2)

1. 처음 렌더링 시, App 컴포넌트가 렌더링 되고 count는 모두 1이 된다.
2. 텍스트 입력 필드에서 값을 변경하면, useState를 통해 App 컴포넌트가 리렌더링 된다.
3. ColorContext.Provider는 새로운 값을 받고, 동시에 Parent 컴포넌트도 리렌더링 된다.
4. DummyComponent는 리렌더링 되지만, MemoidDummyComponent는 리렌더링 되지 않는다.
5. ColorComponent는 두 가지 이유로 렌더링 된다. 1. 부모가 리렌더링 됐고, 2. Context의 값이 변경되었기 때문이다.
6. MemoidColorComponent는 컨텍스트가 변경되었기 때문에 리렌더링 된다.

**_ 중요한점 _**
memo가 내부 소비자가 렌더링되는 것을 막지 못한다는 것.

### 컨텍스트에서 객체를 사용할 때 주의할 점

[컨텍스트에서 객체 사용예제](https://codesandbox.io/p/sandbox/sj7mns)

객체를 사용할때 Counter1컴포넌트는 count2 객체의 속성에 영향을 받아, 렌더링이 일어난다.

즉, 불필요한 렌더링이 발생한다.

## 전역 상태를 위한 컨텍스트 만들기

이런 컨텍스트의 리렌더링을 위한 해결책으로는 두가지를 제시하고 있다.

- 작은 상태 조각으로 분리하기
- useReducer로 하나의 상태를 만들고 여러 컨텍스트로 전파하기

### 작은 상태 조각으로 분리하기

큰 객체를 사용하는 대신 각 조각에 대한 컨텍스트와 전역 상태를 만들 수 있다.

[작은 상태 조각으로 분리하기 예제](https://codesandbox.io/p/sandbox/vfl75t)

App 컴포넌트에는 두 개의 공급자 컴포넌트가 중첩돼 있다. 이 방식은 곤급자 컴포넌트가 많을쓰록 중첩 Depth가 깊어지는 단점이 있지만,
리렌더링이 발생하지 않는다.

### useReducer로 하나의 상태를 만들고 여러 컨텍스트로 전파하기

useReducer 훅과 Context API를 결합하여 애플리케이션 전반에 걸쳐 상태를 관리하고 분배하는 패턴이다.
이 방법은 특히 복잡한 상태 로직을 처리하는데 유용하며, 전역 상태를 효율적으로 분할하여 관리할 수 있게 해준다.

- useReducer는 React에서 상태 업데이트 로직을 외부화할 수 있게 해주는 훅입. 이를 통해 컴포넌트 로직을 더 깔끔하게 유지하고, 다양한 상태 변화를 보다 예측 가능하게 만들 수 있다. Context API와 결합하면, useReducer로 생성된 상태를 애플리케이션의 여러 부분으로 쉽게 전달할 수 있다.

- 구현 방법

1. 상태 정의와 Reducer 함수 구현

- 먼저, useReducer를 사용하여 애플리케이션의 상태를 정의함. 이 예제에서는 두 개의 카운터 상태 count1과 count2를 관리하는 간단한 예를 들 수 있다.
- Reducer 함수는 상태를 변화시키는 로직을 담고 있으며, 각 액션 타입에 따라 상태를 어떻게 변경할지 결정한다.

2. Context 생성

- 각 상태 값과 그 상태를 업데이트하는 함수를 전역적으로 접근 가능하게 하기 위해, Context API를 사용한다.
- 별도의 Context를 각 상태 값과 dispatch 함수에 대해 생성함.

3. Provider 컴포넌트 구성

- Provider 컴포넌트 내에서 useReducer를 사용하고, 생성된 상태와 dispatch 함수를 Context Provider를 통해 하위 컴포넌트에 전달.

4.  컴포넌트에서 Context 사용

- 하위 컴포넌트에서는 useContext 훅을 사용하여 필요한 상태 값과 dispatch 함수를 추출하고 사용.
- 이를 통해 각 컴포넌트는 필요한 상태만을 선택적으로 받아와서 사용할 수 있으며, 상태 관리 로직을 컴포넌트 밖으로 분리할 수 있다.

  로직의 중앙 집중화: 모든 상태 업데이트 로직이 하나의 장소에서 관리
  재사용성 증가: 다른 컴포넌트에서 동일한 상태 관리 로직을 재사용할 수 있음
  성능 최적화: 불필요한 렌더링을 줄이고, 컴포넌트의 리렌더링을 최소화.

[useReducer로 하나의 상태를 만들고 여러 컨텍스트로 전파하기 예제](https://codesandbox.io/p/sandbox/qfyv6z)

## 컨텍스트 사용을 위한 모범 사례

### 사용자 정의 훅과 공급자 컴포넌트 만들기

이전 장에서는 useContext 훅을 직접 사용하여 컨텍스트 값을 가져왔다. 하지만 이 방식은 규모가 커질수록 코드의 가독성과 유지보수성이 떨어질 수 있다.

**_ 왜 사용자 정의 훅을 사용할까? _**

- 가독성 향상: 컨텍스트에 접근하는 로직을 하나의 훅으로 캡슐화하여 코드가 더 깔끔해진다.
- 재사용성 증가: 여러 컴포넌트에서 동일한 로직을 재사용할 수 있다.
- 오류 감소: 컨텍스트 사용 시 발생할 수 있는 실수를 줄일 수 있다.

간단한 카운터 애플리케이션을 예로 들어보자

1. 컨텍스트 생성

```javascript
import { createContext } from "react";

export const CountContext = createContext();
```

2. 리듀서와 초기 상태 정의

```javascript
const initialState = { count: 0 };

const countReducer = (state, action) => {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + 1 };
    case "DECREMENT":
      return { count: state.count - 1 };
    default:
      throw new Error("Unhandled action type");
  }
};
```

3. 공급자 컴포넌트 만들기

```javascript
import { useReducer } from "react";
import { CountContext } from "./CountContext";

export const CountProvider = ({ children }) => {
  const [state, dispatch] = useReducer(countReducer, initialState);

  return (
    <CountContext.Provider value={{ state, dispatch }}>
      {children}
    </CountContext.Provider>
  );
};
```

4. 사용자 정의 훅 생성
   컨텍스트에 쉽게 접근할 수 있는 useCount 훅

```javascript
import { useContext } from "react";
import { CountContext } from "./CountContext";

export const useCount = () => {
  const context = useContext(CountContext);
  if (!context) {
    throw new Error("useCount must be used within a CountProvider");
  }
  return context;
};
```

5. 컴포넌트에서 사용하기

```javascript
import { useCount } from "./useCount";

const Counter = () => {
  const { state, dispatch } = useCount();

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>증가</button>
      <button onClick={() => dispatch({ type: "DECREMENT" })}>감소</button>
    </div>
  );
};
```

6. 앱에 공급자 적용하기

```javascript
import { CountProvider } from "./CountProvider";
import Counter from "./Counter";

function App() {
  return (
    <CountProvider>
      <Counter />
    </CountProvider>
  );
}

export default App;
```

- 공급자 컴포넌트는 상태와 디스패치 함수를 컨텍스트로 전달.
- 사용자 정의 훅 useCount를 통해 컴포넌트는 간단하게 컨텍스트에 접근할 수 있다.

이 방식은 코드의 가독성과 유지보수성을 크게 향상 시킬 수 있다.

### 사용자 정의 훅이 있는 팩토리 패턴

첫 번째 방식의 단점은 반복적인 작업이 다소 필요하다는 것이다.
매번 새로운 컨텍스트와 공급자, 그리고 해당 컨텍스트에 접근하기 위한 훅을 생성해야 한다. 그러나 이러한 반복 작업을 줄이기 위해 팩토리 패턴을 활용하여 이 과정을 자동화할 수 있다.

#### 팩토리 함수로 컨텍스트 생성 자동화하기

1. 컨텍스트 생성 팩토리 함수 만들기

컨텍스트, 공급자 컴포넌트, 그리고 접근 훅을 생성하는 함수를 작성.

```javascript
// createContextFactory.ts
import React, {
  createContext,
  useContext,
  useReducer,
  Dispatch,
  ReactNode,
} from 'react';

type Reducer<S, A> = (state: S, action: A) => S;

function createContextFactory<S, A>(
  reducer: Reducer<S, A>,
  initialState: S
): [
  React.FC<{ children: ReactNode }>, // Provider 컴포넌트 타입
  () => [S, Dispatch<A>]             // 상태와 디스패치를 반환하는 훅 타입
] {
  const StateContext = createContext<S | undefined>(undefined);
  const DispatchContext = createContext<Dispatch<A> | undefined>(undefined);

  const Provider: React.FC<{ children: ReactNode }> = ({ children }) => {
    const [state, dispatch] = useReducer<Reducer<S, A>>(reducer, initialState);

    return (
      <DispatchContext.Provider value={dispatch}>
        <StateContext.Provider value={state}>
          {children}
        </StateContext.Provider>
      </DispatchContext.Provider>
    );
  };

  const useCombinedContext = (): [S, Dispatch<A>] => {
    const state = useContext(StateContext);
    const dispatch = useContext(DispatchContext);
    if (state === undefined || dispatch === undefined) {
      throw new Error('useCombinedContext must be used within a Provider');
    }
    return [state, dispatch];
  };

  return [Provider, useCombinedContext];
}

export default createContextFactory;

```

- 이 팩토리 함수를 사용하면, 상태 관리가 필요한 각 경우에 대해 새로운 컨텍스트와 공급자, 그리고 접근 훅을 생성할 때 반복되는 코드를 줄일 수 있다.
- 예를 들어, 특정 상태와 리듀서를 정의한 후에 createContextFactory를 사용하여 Provider와 useCombinedContext를 생성 가능.
- 이렇게 생성된 Provider를 애플리케이션의 루트 또는 필요한 부분에 적용하고, useCombinedContext 훅을 사용하여 상태와 디스패치 함수에 점근함

2. 팩토리 함수를 사용하여 컨텍스트 생성
   이제 이 팩토리 함수를 사용하여 필요한 컨텍스트와 공급자, 훅을 간단하게 생성할 수 있습니다.

```javascript
// countReducer.ts
export interface StateType {
  count: number;
}

export const initialState: StateType = { count: 0 };

export type Action =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' };

export const countReducer = (state: StateType, action: Action): StateType => {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    default:
      throw new Error(`Unhandled action type: ${action.type}`);
  }
};


// CountContext.ts
import createContextFactory from './createContextFactory';
import { countReducer, initialState, StateType, Action } from './countReducer';

export const [
  CountProvider,
  useCount
] = createContextFactory<StateType, Action>(countReducer, initialState);

```

3. 컴포넌트에서 사용하기

```jsx
// Counter.tsx
import React from 'react';
import { useCount } from './CountContext';

const Counter: React.FC = () => {
  const [state, dispatch] = useCount();

  return (
    <div>
      <p>현재 카운트: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>증가</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>감소</button>
    </div>
  );
};

export default Counter;

// App.tsx
import React from 'react';
import { CountProvider } from './CountContext';
import Counter from './Counter';

const App = () => (
  <CountProvider>
    <Counter />
  </CountProvider>
);

export default App;
```

### reduceRight를 이용한 공급자 중첩 방지

여러 개의 컨텍스트 공급자를 사용하다 보면, 공급자들이 중첩되어 코드의 깊이가 깊어지고 가독성이 떨어질 수 있다.
이를 해결하기 위해 JavaScript의 reduceRight 함수를 이용하여 공급자들의 중첩을 방지하고, 더 간결하고 관리하기 쉬운 코드를 작성할 수 있다.

```javascript
const App = () => (
  <ProviderA>
    <ProviderB>
      <ProviderC>
        <ProviderD>
          <Component />
        </ProviderD>
      </ProviderC>
    </ProviderB>
  </ProviderA>
);
```

이렇게 되면 공급자들의 중첩으로 인해 코드의 가독성이 떨어지고, 유지보수가 어려워 진다.

#### reduceRight를 이용한 공급자 결합

reduceRight 함수를 사용하여 공급자들을 배열로 관리하고, 이를 통해 공급자들을 동적으로 결합할 수 있다.
이를 통해 위와 같은 중첩을 방지하고, 더 간결한 코드를 작성할 수 있다.

```javascript
const providers = [ProviderA, ProviderB, ProviderC, ProviderD];

const AppProviders = ({ children }) =>
  providers.reduceRight(
    (acc, Provider) => <Provider>{acc}</Provider>,
    children
  );

const App = () => (
  <AppProviders>
    <Component />
  </AppProviders>
);
```

- reduceRight는 배열의 끝에서부터 시작하여 공급자들을 중첩 없이 결합합니다.
- acc는 누적 값으로, 초기값은 children입니다.
- 각 Provider는 acc를 자식으로 감싸며, 최종적으로 모든 공급자가 결합된 컴포넌트를 반환합니다.
