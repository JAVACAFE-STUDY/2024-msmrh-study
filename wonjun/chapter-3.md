### `Context` 의 등장

- Context API는 React 16.3 버전에 출시되었다.
- 같은 내용의 prop을 하위 컴포넌트에 계속 내리는 이른바 props drilling 을해결할 수 있음을 기대할 수 있다.

### 기초적인 사용 방법

```tsx
const Context = createContext("black");

// 선언부
function Provider() {
  return (
    <Context.Provider value="red">
      <Consumer />
    </Context.Provider>
  )
}

// 사용처
function Consumer() {
  const color = useContext(Context);
  
  return (
    /* Color : red */
    <h1>Color : {color}</h1>
  )
}
```

- `createContext` 에서 Context 와 기본 값을 정의한다.
- `Context.Provider` 에서 Context 하위로 내려줄 값을 정의한다.
- `useContext` 에서 내려온 Context 의 값을 가져온다.

### 객체를 사용할 때의 문제점

```tsx
const Context = createContext({
  counter1: 0,
  counter2: 0,
});

// 선언부
function Provider() {
  const [counter1, setCounter1] = useState(0);
  const [counter2, setCounter2] = useState(0);

  return (
    <Context.Provider
      value={{
        counter1,
        counter2,
        setCounter1,
        setCounter2
      }}
    >
      <ConsumerA />
      <ConsumerB/>
    </Context.Provider>
  )
}

// 사용처
function ConsumerA() {
  const { counter1 } = useContext(Context);

  return (
    <>
      <h1>counter1 : {counter1}</h1>
    </>
  )
}

function ConsumerB() {
  const { counter2 } = useContext(Context);

  return (
    <>
      <h1>counter2 : {counter2}</h1>
    </>
  )
}
```

- 만약 여기서 counter1의 값이 바뀌면 ConsumerB 도 리렌더링 될까?
- **답은 Yes 이다.**
- 왜일까? : 1장에서 설명하였듯이, React는 객체를 얕은 비교를 통해 비교하기에, counter1이 바뀌든 counter2가 바뀌든 객체가 바뀌었다고 인지하여 Context전체를 리렌더링 한다.

### 리렌더링 문제를 해결하기 위해서는?

1. counter1과 counter2의 Context 를 분리한다.
2. 더 나아가서 counter1의 값 / counter2의 값 / count를 업데이트하는 함수로Context를 분리한다.

## 모범 사례

### 사용자 정의 훅이 있는 팩토리 패턴

```tsx
// 정의부
const createStateContext = (
  useValue: (init) => State
) => {
  const StateContext = useContext(null);
  const StateProvider = ({ initialValue, children }) => {
    return (
      <StateContext.Provider value={initialValue}>
        {children}
      </StateContext.Provider>
    )
  }
  const useStateContext = () => {
    const value = useContext(StateContext);
    if (!value) {
      throw new Error("provider is missing")
    }
    return value;
  }

  return [StateProvider, useStateContext] as const
}

// 사용처
const [CounterProvider1, useCount1] = createStateContext(useState)
const [CounterProvider2, useCount2] = createStateContext(useState)

const MultipleCounter = () => {
  return (
    <CounterProvider1 initialValue={0}>
      <CounterProvider2 initialValue={10}>
        <Consumer />
      </CounterProvider2>
    </CounterProvider1>
  )
}
```

- 이렇게 createStateContext 를 선언하여 팩토리의 형태로 Context를 정의 할 수 있다.
- (나비 의견) 근데 개인적으로 이렇게 했을때 파일 구조가 복잡해지면 어떻게 할거지? 라는 근본적인 의문과 함께 이렇게까지 쓸 이유가 있을까? 라는 생각이 든다.

### (더 나아가서) reduceRight를 사용한 반복 최소화

```tsx
const [CounterProvider1, useCount1] = createStateContext(useState)
const [CounterProvider2, useCount2] = createStateContext(useState)
const [CounterProvider3, useCount3] = createStateContext(useState)
const [CounterProvider4, useCount4] = createStateContext(useState)
const [CounterProvider5, useCount5] = createStateContext(useState)

const MultipleCounter = () => {
  const providers = [
    CounterProvider1, { initialValue: 10 },
    CounterProvider2, { initialValue: 20 },
    CounterProvider3, { initialValue: 30 },
    CounterProvider4, { initialValue: 40 },
    CounterProvider5, { initialValue: 50 },
  ] as const

  return (
    <>
      {providers.reduceRight(
        (children, [Provider, props]) =>
          createElement(Provider, props, children),
        <Consumer />
      )}
    </>
  )
}
```

- 이렇게 Provider의 내용이 많을 경우 reduceRight 를 사용하여 반복문을 줄일수도 있다. 근데 굳이? 그래야 하려나? 잘 모르겠다.
