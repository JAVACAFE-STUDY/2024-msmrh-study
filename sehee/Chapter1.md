# 리액트 훅을 이용한 마이크로 상태 관리

## 상태란?

사용자 인터페이스(UI)를 나타내는 모든 데이터.

시간이 지남에 따라 변할 수 있으며 리액트는 상태와 함께 렌더링할 컴포넌트를 처리한다.

## 상태 관리 패러다임의 변화

1. 리액트 훅이 나오기 전까지는 중앙 집중형 상태 관리 라이브러리를 사용하는 것이 일반적이었음.

   중앙 집중형 상태 관리 라이브러리 : Redux, MobX 등 애플리케이션의 상태를 한 곳에서 관리하는 방식을 제공하는 라이브러리. 다양한 상황을 포괄적으로 지원할 수 있다는 장점이 있으나 대체로 사용되지 않는 기능까지 포함되어 있어 무겁다는 단점이 있다.

2. 리액트 훅이 등장한 이후, 상태 관리를 마이크로화할 수 있는 가능성이 제기됨 ⇒ 목적 지향적이며 특정한 코딩 패턴과 함께 사용되는 마이크로 상태 관리의 등장
   - 특정 목적(ex: 폼 상태, 서버 캐시 상태 등)을 해결하기 위해 기존의 중앙 집중형 라이브러리와는 다른 해결책을 제공할 수 있게 됨(ex: Formik, React Query)
3. 그렇지만 목적 지향적인 방법으로 처리할 수 없는 상태도 있기에 여전히 범용적인 상태 관리가 필요한 상태
   - 가벼워야 하며, 개발자가 요구사항에 따라 적절한 방법을 선택할 수 있어야 하는 **마이크로 상태 관리**의 필요성 대두됨

즉, **마이크로 상태 관리**란

1. 리액트의 가벼운 상태 관리이며,
2. 각 상태 관리 방법마다 서로 다른 기능을 가지고,
3. 개발자는 애플리케이션 요구사항에 따라 적합한 방법을 선택할 수 있는 방법론이다.

## 마이크로 상태 관리가 갖춰야 할 기능

마이크로 상태 관리는 기본적인 상태 관리 기능이 필요하며, 다음과 같은 작업을 수행할 수 있어야 함

- 상태 읽기
- 상태 갱신
- 상태 기반 렌더링

하지만 다른 작업을 수행하기 위해 다음과 같은 추가적인 기능이 필요할 수 있다.

- 리렌더링 최적화
- 다른 시스템과의 상호 작용
- 비동기 지원
- 파생 상태
- 간단한 문법

또한 쉬운 학습 곡선을 가져야 한다.

## 리액트 훅 사용하기

useState, useReducer, useEffect 등은 기본 리액트 훅이다.

- useEffect : 리액트 렌더링 프로세스 바깥에서 로직을 실행할 수 있다. 리액트 컴포넌트 생명 주기와 함께 작동하는 기능을 구현하기 위해 필요하다.

리액트 훅을 사용하면 UI 컴포넌트에서 로직을 추출할 수 있으며, 이를 통해

1. 로직에 명확한 이름을 붙여 가독성을 향상시킬 수 있고
2. 기능 로직을 UI 로직과 분리하여 컴포넌트를 건드리지 않고도 기능을 추가할 수 있게 한다.

리액트 훅을 사용할 땐 잘못 사용하지 않도록 주의해야 한다.

ex) 기존 state 객체나 ref 객체를 직접 변경해서는 안 됨. 이 경우 리렌더링되지 않거나, 너무 많은 리렌더링이 발생하거나, 부분적인 리렌더링이 발생하는 등 예기치 않은 동작이 발생할 수 있다.

## 데이터 불러오기를 위한 서스펜스와 동시성 렌더링

- 서스펜스 : 비동기 처리(async)에 대한 걱정 없이 컴포넌트를 코딩할 수 있는 방법

  React 최신 버전에 릴리즈된 \<Suspense />는 자식 요소가 로드되기 전까지 화면에 대체 UI를 보여주는 기능을 가진 컴포넌트이다.

- 동시성 렌더링 : 렌더링 프로세스를 청크(chunk)라는 단위로 분할해서 중앙 처리 장치(CPU)가 장시간 차단되는 것을 방지하는 방법

## 전역 상태 탐구하기

전역 상태 : 애플리케이션 내 서로 멀리 떨어져 있는 여러 컴포넌트에서 사용하는 상태. 다음 코드에서 useGlobalState()가 전역 상태를 활용하는 예시 형태이다.

```jsx
const Component1 = () => {
  const [state, setState] = useGlobalState();
  return <div>{JSON.stringify(state)}</div>;
};

const Component2 = () => {
  const [state, setState] = useGlobalState();
  return <div>{JSON.stringify(state)}</div>;
};
```

리액트는 지역성이 중요한 컴포넌트 모델에 기반하기 때문에 전역 상태에 대한 직접적인 해결책을 제공하지 않으며, 따라서 개발자와 커뮤니티들이 제안해 온 수많은 해결책들이 존재한다.

## useState 사용하기

### 기본 사용법

```jsx
const Component = () => {
  const [count, setCount] = useState(0);
  return (
    <div>
      {count}
      <button onClick={() => setCount(1)}>Set Count to 1</button>
    </div>
  );
};
```

### TIL

- 현재 count와 동일한 값으로 setCount를 호출하면 **베일아웃**되어 컴포넌트가 다시 렌더링되지 않는다.
  - 베일아웃 : 리액트 기술 용어로, 리렌더링을 발생시키지 않음을 의미함
- state로 객체를 다룰 경우, 아래와 같이 객체 내부를 수정하면 마찬가지로 state 객체가 실제로 변경되지 않았기 때문에 베일아웃되어 리렌더링이 발생하지 않는다.
  ```jsx
  const Component = () => {
    const [state, setState] = useState({ count: 0 });
    return (
      <div>
        {state.count}
        <button
          onClick={() => {
            state.count = 1;
            setState(state);
          }}
        >
          Set Count to 1
        </button>
      </div>
    );
  };
  ```
- 새로운 값을 setState에 전달해 상태를 갱신할 수도 있지만, 함수를 전달하여 상태를 갱신할 수도 있다. 이러한 갱신 함수는 이전 값을 기반으로 갱신하는 경우에 유용하다. (즉, setState 호출 버튼을 빠르게 여러 번 누를 때 누른 만큼 count가 증가해야 할 경우)
  ```jsx
  const Component = () => {
    const [count, setCount] = useState(0);
    return (
      <div>
        {count}
        <button onClick={() => setCount((c) => c + 1)}>Increment Count</button>
      </div>
    );
  };
  ```
- useState는 첫 번째 렌더링에서만 평가되는 초기화 함수(init 함수)를 받을 수 있다. 초기화 함수는 useState가 호출되기 전까지 평가되지 않고 느리게 평가되며, 컴포넌트가 마운트될 때 한 번만 호출된다.

  ```jsx
  const init = () => 0;

  const Component = () => {
    const [count, setCount] = useState(init);
    return (
      <div>
        {count}
        <button onClick={() => setCount((c) => c + 1)}>Increment Count</button>
      </div>
    );
  };
  ```

## useReducer 사용하기

### 기본 사용법

리듀서는 복잡한 상태에 유용하다.

```jsx
const reducer = (state, action) => {
  switch (action.type) {
    case "INCREMENT":
      return { ...state, count: state.count + 1 };
    case "SET_TEXT":
      return { ...state, text: action.text };
    default:
      throw new Error("unknown action type");
  }
};

const Component = () => {
  const [state, dispatch] = useReducer(reducer, { count: 0, text: "hi" });
  return (
    <div>
      {state.count}
      <button onClick={() => dispatch({ type: "INCREMENT" })}>
        Increment count
      </button>
      <input
        value={state.text}
        onChange={(e) => dispatch({ type: "SET_TEXT", text: e.target.value })}
      />
    </div>
  );
};
```

### syntax

```jsx
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

#### 매개변수

- `reducer` : state가 어떻게 업데이트 되는지 지정하는 함수. 리듀서 함수는 반드시 순수 함수여야 하며, state와 action을 인수로 받아야 하고, 다음 state를 반환해야 한다.
  useState와 마찬가지로 리듀서 함수에서 이전 상태와 같은 state를 반환할 경우, 베일아웃되어 리렌더링이 발생하지 않는다.
  ```jsx
  const reducer = (state, action) => {
    switch (action.type) {
      case "INCREMENT":
        return { ...state, count: state.count + 1 };
      case "SET_TEXT":
        if (!action.text) {
          // bail out
          return state;
        }
        /**
        만약 return {...state, text: action.text || state.text}
        와 같은 식으로 반환하면 같은 text가 들어와도
        새로운 객체가 생성되기 때문에
        베일아웃이 발생하지 않음. => 불필요한 리렌더링.
        **/
        return { ...state, text: action.text };
      default:
        throw new Error("unknown action type");
    }
  };
  ```
  리듀서 함수의 두 번째 인자로 들어오는 action은 객체가 아닌 숫자나 문자열 같은 원시 값이 들어올 수도 있다. (이 경우 파라미터 이름을 `delta`로 많이 쓰는 듯)
  ```jsx
  const reducer = (count, delta) => {
    if (delta < 0) {
      throw new Error("delta cannot be negative");
    }
    if (delta > 10) {
      // too big, just ignore
      return count;
    }
    if (count < 100) {
      // add bonus
      return count + delta + 10;
    }
    return count + delta;
  };
  ```
- `initialArg` : 초기 state가 계산되는 값. init 함수가 할당되지 않을 경우 이 값이 state의 초기값으로 설정된다.
- `init(optional)` : 초기 state를 반환하는 초기화 함수. 이 함수가 할당되었다면 초기 state는 `init(initialArg)`를 호출한 결과가 할당된다.

#### 반환값

useReducer는 2개의 엘리먼트로 구성된 배열을 반환한다.

1. 현재 state
2. dispatch 함수. dispatch는 state를 새로운 값으로 업데이트하고 리렌더링을 일으킨다. dispatch의 인수로 들어가는 것이 reducer 함수의 action으로 들어온다.

## useState와 useReducer의 유사점과 차이점

- useReducer로 useState를 구현하는 것은 100% 가능하다. useState로 useReducer를 구현하는 것 또한 거의 가능하나 미묘한 차이가 있다.
- useReducer에서는 reducer와 init을 훅이나 컴포넌트 외부에서 정의할 수 있지만 useState에서는 불가능하다.
  - useReducer의 init은 initialArg를 받게끔 설계되어 있지만, useState의 init은 인수를 받지 않는 함수로 설계되어 있기 때문
- 외부 변수에 의존할 수 있는 인라인 리듀서 함수는 useReducer에서만 가능하며 useState에서는 불가능하다.

  ```jsx
  //인라인 리듀서 함수의 예시
  const useScore = (bonus) =>
    useReducer((prev, delta) => prev + delta + bonus, 0);
  ```

  위 코드는 bonus와 delta가 거의 동시에 갱신될 경우에도 올바르게 작동한다.
  ⇒ useReducer가 렌더링 단계에서 리듀서 함수를 호출하기 때문. useState를 사용한다면 이전 렌더링에서 사용된 이전 bonus 값을 사용하므로 제대로 작동하지 않는다.

  그러나 이 기능은 일반적으로 사용되지 않으며, 이와 같은 특별한 기능만 제외하면 useReducer와 useState는 전반적으로 동일하다.
