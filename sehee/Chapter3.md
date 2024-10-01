# 3. 리액트 컨텍스트를 이용한 컴포넌트 상태 공유

## 컨텍스트를 사용하는 이유

props를 사용하지 않고도 부모 컴포넌트에서 트리 아래에 있는 자식 컴포넌트로 값을 전달할 수 있게 하기 위함

## 컨텍스트의 특징

- 컨텍스트 공급자는 중첩될 수 있다.
- useContext가 있는 컴포넌트는 컴포넌트 트리 중에서 가장 가까운 공급자를 선택해 값을 가져온다.
- 공급자로 둘러싸여 있지 않는 컴포넌트에서 useContext를 호출했을 경우, 컨텍스트 생성 시 넣은 기본값을 반환한다.
  - 대부분의 경우 공급자를 감싸지 않고 사용하는 것은 의도하지 않은 것이므로, 기본값에 null을 넣고 이를 useContext 로직에서 검사하여 에러로 핸들링하기도 한다.

## 컨텍스트 이해하기

컨텍스트 공급자가 새로운 컨텍스트 값을 갖게 되면, 모든 컨텍스트 소비자(useContext가 있는 컴포넌트)는 새로운 값을 받고 리렌더링된다.

[리액트 훅을 활용한 마이크로 상태관리 - example 1 - StackBlitz](https://stackblitz.com/edit/react-ndoh3p?file=src/App.js)

내용 끌어올리기 기법이나 memo를 활용하면 부모 컴포넌트의 렌더링으로 인한 리렌더링을 막을 수 있지만, 컨텍스트 변경으로 인한 리렌더링은 막을 수 없다.

- 위 예시에서 부모에 의해 리렌더링이 발생하는 Dummy 컴포넌트는 memo를 통해 리렌더링을 방지할 수 있지만, 컨텍스트 변경에 의해 리렌더링이 발생하는 Color 컴포넌트는 memo를 붙인 것과 붙이지 않은 것이 똑같이 리렌더링됨.

### 컨텍스트에 객체를 사용할 때의 한계점

컴포넌트가 컨텍스트의 객체 중 일부만 사용할 경우, 사용하지 않는 데이터가 변경될 때에도 컨텍스트 소비자가 리렌더링된다.

[리액트 훅을 활용한 마이크로 상태관리 - 객체를 사용할 때의 한계점 - StackBlitz](https://stackblitz.com/edit/react-szgqfr?file=src/App.js)

위 예시에서 각 counter는 객체로 이루어진 컨텍스트 데이터의 일부만 사용하고 있지만, 사용하고 있지 않은 쪽의 데이터가 변경될 때 전부 리렌더링되는 모습을 보인다.

## 컨텍스트로 전역 상태를 구현하기 위한 패턴

위의 리렌더링 문제를 해결할 수 있는 두 가지 패턴이 있다.

- 작은 상태 조각 만들기
- useReducer로 하나의 상태를 만들고 여러 컨텍스트로 전파하기

### 작은 상태 조각 만들기

합쳐진 큰 객체를 사용하는 대신, 전역 상태를 여러 조각으로 나눠 각각에 대한 컨텍스트와 전역 상태를 만들면 불필요한 리렌더링 문제를 없앨 수 있다.

[리액트 훅을 활용한 마이크로 상태관리 3 - 작은 상태 조각 만들기 - StackBlitz](https://stackblitz.com/edit/react-ts-mkubmx?file=App.tsx)

위 예시에서는 counter1와 counter2에서 사용하는 count 상태를 분리하고 컨텍스트와 공급자를 각각 따로 만들어 컨텍스트별로 리렌더링이 되는 모습을 볼 수 있다.

### useReducer로 하나의 상태를 만들고 여러 개의 컨텍스트로 전파하기

객체로 이루어진 단일 상태를 만들고 여러 컨텍스트를 사용해 이를 배포하는 방법도 있다.

[리액트 훅을 활용한 마이크로 상태관리 3 - useReducer로 하나의 상태를 만들고 여러 개의 컨텍스트로 전파하기 - StackBlitz](https://stackblitz.com/edit/react-ts-khzju9?file=App.tsx)

위 예시에서, 상태는 {count1, count2} 형태의 객체이지만 각 count마다 컨텍스트를 따로 만들었다. 이 경우 각 카운트가 변경되면 해당 컨텍스트를 구독하는 컴포넌트만 리렌더링된다.

단일 상태를 사용할 때의 장점은 하나의 액션으로 객체에 연결된 여러 데이터를 갱신할 수 있다는 것이다.

## 컨텍스트 사용을 위한 모범 사례

### 사용자 정의 훅과 공급자 컴포넌트 만들기

컨텍스트 값에 접근하기 위해 필요한 반복되는 로직을 사용자 정의 훅으로 묶어서 사용할 수 있다.

예를 들어 아래와 같은 컨텍스트가 있을 때,

```tsx
type CountContextType = [number, Dispatch<SetStateAction<number>>];

const Count1Context = createContext<CountContextType | null>(null);

export const Count1Provider = ({ children }: { children: ReactNode }) => (
  <Count1Context.Provider value={useState(0)}>
    {children}
  </Count1Context.Provider>
);
```

이를 활용하는 useContext(Count1Context) 로직을 다음과 같은 훅으로 뺄 수 있다.

```tsx
export const useCount1 = () => {
  const value = useContext(Count1Context);
  if (value === null) throw new Error("Provider missing");
  return value;
};
```

위 훅은 컨텍스트 기본값이 null일 경우 명시적인 에러를 던지게 해서 개발자들이 빠르게 버그를 찾을 수 있게 한다.

활용하는 쪽에서는 다음과 같이 쓴다.

```tsx
const Counter1 = () => {
  const [count1, setCount1] = useCount1();
  return (
    <div>
      Count1: {count1}{" "}
      <button onClick={() => setCount1((c) => c + 1)}>+1</button>
    </div>
  );
};

const App = () => (
  <Count1Provider>
    <Counter1 />
  </Count1Provider>
);
```

CountProvider 쪽에서 useState와 같은 [state, setState] 형식으로 value를 반환하고 있으므로, 이를 사용자 정의 훅에서 그대로 활용할 수 있다.

일반적으로 컨텍스트와 관련된 코드는 `contexts/count1.jsx`와 같은 파일에 별도로 두고, useCount1 훅 및 Count1Provider와 같은 컴포넌트만 내보낸다. Context 자체는 내보내지 않는다.

### 사용자 정의 훅이 있는 팩토리 패턴

위와 같은 사용자 정의 훅과 공급자 컴포넌트를 만드는 것은 반복적인 작업이므로, 이를 수행하는 함수인 `createStateContext`를 만들 수 있다.

createStateContext 함수는 useValue 함수(초깃값을 받아 상태를 반환하는 사용자 정의 훅)를 인자로 받고, 공급자 컴포넌트와 상태를 가져오는 사용자 정의 훅으로 이루어진 튜플을 반환한다. 코드로 나타내면 다음과 같다.

```tsx
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

위와 같이 사용할 경우 Provider에 선택적인 initialValue를 넘길 수 있다.

위 컨텍스트 함수를 사용하는 코드는 다음과 같다.

```tsx
const useNumberState = (init?: number) => useState(init || 0);

const [Count1Provider, useCount1] = createStateContext(useNumberState);
const [Count2Provider, useCount2] = createStateContext(useNumberState);

const Counter1 = () => {
  const [count1, setCount1] = useCount1();
  return (
    <div>
      Count1: {count1}{" "}
      <button onClick={() => setCount1((c) => c + 1)}>+1</button>
    </div>
  );
};

const Counter2 = () => {
  const [count2, setCount2] = useCount2();
  return (
    <div>
      Count2: {count2}{" "}
      <button onClick={() => setCount2((c) => c + 1)}>+1</button>
    </div>
  );
};

const Parent = () => (
  <div>
    <Counter1 />
    <Counter1 />
    <Counter2 />
    <Counter2 />
  </div>
);

const App = () => (
  <Count1Provider>
    <Count2Provider>
      <Parent />
    </Count2Provider>
  </Count1Provider>
);
```

위와 같이 createStateContext를 활용하면 반복적인 코드를 피하면서 원하는 만큼 상태 컨텍스트를 만들 수 있다.

createStateContext의 인자로 들어가는 사용자 훅에서는 useState뿐 아니라 useReducer를 활용할 수도 있으며, useEffect를 활용한 더 복잡한 훅을 넣을 수도 있다. 또한 createStateContext와 사용자 훅에 타입을 적용하면 결과값에도 타입이 적용된다.

### reduceRight를 이용한 공급자 중첩 방지

아래와 같이 여러 상태가 만들어져 공급자가 중첩되었을 경우,

```tsx
const [Count1Provider, useCount1] = createStateContext(useNumberState);
const [Count2Provider, useCount2] = createStateContext(useNumberState);
const [Count3Provider, useCount3] = createStateContext(useNumberState);
const [Count4Provider, useCount4] = createStateContext(useNumberState);
const [Count5Provider, useCount5] = createStateContext(useNumberState);

const App = () => (
  <Count1Provider initialValue={10}>
    <Count2Provider initialValue={20}>
      <Count3Provider initialValue={30}>
        <Count4Provider initialValue={40}>
          <Count5Provider initialValue={50}>
            <Parent />
          </Count5Provider>
        </Count4Provider>
      </Count3Provider>
    </Count2Provider>
  </Count1Provider>
);
```

보기에 복잡하고 코딩할 때 불편하다.

이러한 스타일을 완화하기 위해 reduceRight와 React의 createElement 함수를 사용할 수 있다.

<aside>
💡

[**reduceRight**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduceRight) : Array 메소드로 reduce()의 반대 방향 버전이다.

배열의 맨 끝 원소부터 시작해서 모든 요소에 대해 리듀서 콜백 함수를 실행하고 이를 단일 값으로 누적시킨다.

</aside>

<aside>
💡

[**createElement**](https://ko.react.dev/reference/react/createElement) : JSX를 작성하는 대신 React 엘리먼트를 생성할 수 있는 함수이다. 순서대로 유효한 React 컴포넌트, props, children을 인수로 받아 React 엘리먼트를 생성한다.

</aside>

reduceRight를 활용한 코드는 다음과 같다.

```tsx
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
