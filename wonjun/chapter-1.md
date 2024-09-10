# 01 <br><br>리액트 훅과<br>마이크로 상태 관리

## 마이크로 상태 관리 이해하기

- React 에서 상태는 UI를 나타내는 모든 데이터를 의미 한다
  - 상태는 시간이 지남에 따라 변할 수 있다.
- React 는 상태와 함께 렌더링 할 컴포넌트를 처리한다.
- React Hook 이전에는 Redux 와 같은 중앙 집중형 상태 관리 라이브러리를 사용했다.
  - 장점 : 한개의 상태로 다양한 상황을 포괄적으로 지원할 수 있었다.
  - 단점 : 다양한 특성을 가지는 상태를 모두 관리하기가 어려웠다.
    - Form : 전역 상태와 별도로 처리해야 한다.
    - API Cache : refetching 와 같은 고유한 특성이 있다.
    - 내비게이션 상태 : 원 상태가 브라우저에 있기에 단일 상태에서 처리하기 적합하지 않다.
- 이 문제를 해결하기 위해 React Hook 이 탄생하게 되었다.
  - Form / API Cache와 같은 다양한 상태를 목적에 맞게 '각자' 처리하는 방향으로 설계되었다.
- 범용적인 상태 관리가 필요한 부분도 분명히 필요하다. -> 마이크로 상태 관리 의 탄생
  - 기본적으로 상태 읽기, 상태 갱신, 상태 기반 렌더링이 가능해야 한다.
  - (필요하다면) 비동기 지원, 파생 상태, 간단한 문법과 같은 기능이 붙을 수 있다.
  - 사용 사례가 적기 때문에, 기능을 사용하기 위한 학습 곡선이 낮아야 한다.

## React Hook 사용하기

- 기본으로 제공되는 React Hook
  - `useState` : 지역 상태를 생성한다.
  - `useReducer` : useState 와 비슷하지만, 다른 부분이 있다.
  - `useEffect` : 렌더링 프로세스 외부에서 로직을 실행할 수 있다.
- React Hook 은 UI 컴포넌트에서 로직을 추출할 수 있어 책임 소재의 분리가 가능하다.
- `Counter` 에서 count 로직을 `useCount` hook 으로 분리하는 경우
  - count 로직이 `useCount` 로 분리되어 이름이 명확해졌다. -> 이를 통해 코드의 가독성이 좋아진다.
  - `Counter` 가 `useCount` 의 구현과 분리되었다. -> 컴포넌트를 건드리지 않고도 기능을 추가할 수 있다.

## 데이터 불러오기를 위한 Suspense 와 동시성 렌더링

- Suspense 는 비동기 처리에 대한 걱정 없이 컴포넌트를 개발할 수 있는 방법이다. ([#](https://react.dev/reference/react/Suspense))
- 동시성 렌더링은 렌더링 프로세스를 청크 단위로 분할해 CPU가 차단되는 것을 방지하는 방법이다. ([#](https://react.dev/blog/2021/12/17/react-conf-2021-recap#react-18-and-concurrent-features))
- React Hook 의 조건
  - 기존 state 객체나 ref 객체를 직접 변경할 수 없다.
  - 함수가 여러번 호출 되어도 함수의 결과가 같아야 한다 (= 순수 함수여야 한다)

```tsx
function Component() {
  const [state, setState] = useState(0);

  // 이렇게 값을 변경해서는 안된다
  state = 1;

  // 이렇게 값을 변경 해야한다
  setState(1);
}
```

## 전역 상태 탐구하기

- `useState` 를 사용하면 Component 트리에서 사용하는 상태를 정의할 수 있는데, 이를 지역 상태 라고 한다.

```tsx
function Component1() {
  const [state, setState] = useState(0);
}

function Component2() {
  const [state, setState] = useState(0);
}
```

- 전역 상태 (또는 공유 상태) 는 어플리케이션 내에서 서로 멀리 떨어져있는 여러 컴포넌트에서 사용하는 상태이다.
  - 언제나 전역 상태가 singleton 일 필요는 없다.

```tsx
function Component1() {
  const [state, setState] = useGlobalState();
}

function Component2() {
  const [state, setState] = useGlobalState();
}
```

## `useState` 사용하기

### 값으로 상태 갱신하기

```tsx
function Component1() {
  const [state, setState] = setState(0);

  return (
    <button
      onClick={() => {
        setCount(1);
      }}
    >
      {state}
    </button>
  );
}
```

- 위 코드에서 버튼을 2번 이상 눌러도 렌더링은 1번만 발생한다.
  - 베일아웃 : 값이 동일할 경우 여러번 호출되어도 렌더링을 하지 않는 것

```tsx
function Component1() {
  const [state, setState] = useState({ count: 0 });

  return (
    <button
      onClick={() => {
        setState({ count: 1 });
      }}
    >
      {state}
    </button>
  );
}
```

- 위 코드에서는 2번 이상 클릭하면 리렌더링이 발생한다.
- 왜냐하면 setState 에 들어가는 객체가 언제나 새롭게 생성되기 때문이다.

```tsx
function Component1() {
  const [state, setState] = useState(0);

  return (
    <button
      onClick={() => {
        setState(state + 1);
      }}
    >
      {state}
    </button>
  );
}
```

- 이 경우에는 버튼을 빠르게 여러번 클릭해도 값이 한번만 증가한다.
  - state 가 변경되지 않았기 때문이다.
  - 이를 해결하기 위해 갱신 함수를 사용해야 한다

```tsx
function Component1() {
  const [state, setState] = useState(0);

  return (
    <button
      onClick={() => {
        setState((prev) => prev + 1);
      }}
    >
      {state}
    </button>
  );
}
```

- setState 함수 내에 `(prev) => prev + 1` 이 추가되었다
  - 이는 화면에서 state 를 가져오지 않고 내부 상태에서 state를 가져온다.
  - 따라서 함수를 통한 베일아웃도 가능하다.

### 지연 초기화

```tsx
const init = () => 0;

function Component1() {
  const [state, setState] = useState(init);

  return (
    <button
      onClick={() => {
        setState((prev) => prev + 1);
      }}
    >
      {state}
    </button>
  );
}
```

- 이렇게 함수를 초기 값으로 넣을 수 있는데, 이 경우에는 컴포넌트가 마운트 될 때 한번만 호출되게 된다.

## `useReducer` 사용하기

```tsx
function reducer(state, action) {
  switch (action.type) {
    case "ADD": {
      return { ...state, count: state.count + 1 };
    }

    default: {
      return state;
    }
  }
}

function Component1() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <button
      onClick={() => {
        dispatch({ type: "ADD" });
      }}
    >
      {state}
    </button>
  );
}
```

- 기본적으로 이렇게 사용할 수 있다. (redux의 reducer와 많이 닮아 있다)

```tsx
function reducer(count, delta) {
  if (delta < 0) {
    throw new Error("can not be nagative");
  }

  if (delta > 10) {
    return count;
  }

  if (count < 10) {
    return count + delta + 10;
  }

  return count + delta;
}
```

- 이런 방식으로 객체 없이 원시값에 대해서도 사용할 수 있다.

## `useState` / `useReducer` 차이점

- useState / useReducer 는 서로가 서로를 대체해서 사용할 수 있다.
- useReducer 는 reducer 함수와 init 값을 외부에서 정의할 수 있다.

```tsx
const init = (count) => ({ count });
const reducer = (prev, delta) => ({ ...prev, count: prev.count + delta });

function ComponentWithReducer() {
  const [state, dispatch] = useReducer(reducer, 0, init);

  // ...
}

function ComponentWithState() {
  const [state, setState] = useState(() => init(0));
  const dispatch = (delta) => setState((prev) => reducer(prev, delta));

  // ...
}
```
