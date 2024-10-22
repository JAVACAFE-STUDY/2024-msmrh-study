# 전역 상태 관리 라이브러리 소개

## 전역 상태 관리 문제 해결하기

### 전역 상태 관리의 문제

전역 상태는 여러 값을 가질 수 있고, 전역 상태를 사용하는 컴포넌트는 전역 상태의 모든 값이 필요하지 않은 경우가 있다. 전역 상태가 바뀌면 리렌더링이 발생하는데, 변경된 값이 컴포넌트와 관련 없는 경우에도 리렌더링이 발생한다. > 리렌더링 최적화가 필요하다.

### 상태에 값을 넣거나 갱신하는 방법

전역 상태가 중첩된 객체인 경우 하나의 전역 변수를 가지고 직접 값을 변경하는 것은 좋은 방법이 아닐 수 있다.

전역 변수에서 하나의 프로퍼티를 개발자가 직접 값을 변경하는 예

```JS

let globalVariable = {

a:1,

b:{

c:2,

d:3,

},

e:[4,5,6],

}

globalVariable.b.d = 9;

```

위와 같은 방식은 상태의 변경사항을 감지하지 못하고 리렌더링이 발생하지 않는다.
따라서 전역 상태 변경을 감지하기 위해서는 전역 상태를 변경하는 함수를 제공해야한다.
또한 변수가 직접 변경될 수 없도록 클로저에서 변수를 숨기는 경우도 있다.

```js

const createContainer = () => {

let state = {a:1, b:2};

const getState = () => state;

const setState = (...) => {...};

return {getState, setState}

}

const globalContainer = createContainer();

globalContainer.setState(...);

```

## 데이터 중심 접근 방식과 컴포넌트 중심 접근 방식 사용하기

### 데이터 중심 접근 방식 이해하기

- 컴포넌트를 정의한 후 데이터와 컴포넌트를 연결하는 방식
- 데이터 모델이 싱글턴으로 리액트 의존 없이 자바스크립트 메모리에 있는 모듈 상태를 사용
- 모듈 상태를 생성하고 모듈 상태를 리액트 컴포넌트에 연결하는 API를 제공
- 모듈 상태는 보통 상태 변수에 접근하고 갱신하는 메서드를 가진 store 객체로 감싼다.
- 리액트 컴포넌트의 라이프사이클과 상관 없이 모듈 상태는 존재
- 데이터 중심 접근 방식 전역 상태 라이브러리는 팩토리 함수를 제공 > 리액트 컴포넌트에서 사용할 전역 상태 초기화 함수 생성

### 컴포넌트 중심 접근 방식 이해하기

- 컴포넌트를 상태보다 먼저 설계할 수 있고 데이터 모델이 컴포넌트와 강한 의존성을 가지고 있음
- 컴포넌트 생명 주기 내에서 전역 상태를 유지 > 의존하는 컴포넌트가 모두 마운트 해제되면 전역 상태도 함께 사라짐
- 자바스크립트 메모리에 두 개 이상의 동일한 전역 상태를 둘 수 있음
- 불필요한 props drilling이 발생하는 경우 해당 방식을 도입할 수 있음
- 데이터 중심 접근 방식의 전역 상태 라이브러리를 사용할 때는ㄴ 팩토리 함수 내 전역 상태 초기화 함수를 통해 생명주기 처리

### 데이터/컴포넌트 중심 접근 방식의 예외

꼭 두 가지 접근 방식 중 하나만 선택해야 하는 것은 아니다. 실제로는 두 가지 접근 방식 중 하나를 사용하거나 두 가지 접근 방식을 함께 사용할 수 있다.

- 모듈 상태는 대체로 싱글턴 패턴으로 구현되지만 하위 트리에 대해 여러 모듈 상태를 만들 수 있다.
- 모듈 상태의 생명 주기를 제어할 수도 있다.
- 공급자 컴포넌트를 트리의 최상위에 두고 트리가 하나만 있으면 사실상 싱글턴 패턴이라고 볼 수 있다.
- 컴포넌트 상태는 대체로 useState훅으로 구현되지만 변경 가능한 변수나 store가 필요한 경우 useRef 훅으로도 구현이 가능하다.

## 리렌더링 최적화

```jsx
let state = {
  a: 1,

  b: { c: 2, d: 3 },

  e: { f: 4, g: 5 },
};

const ComponentA = () => {
  return <>value: {state.b.c}</>;
};

const ComponentB = () => {
  return <>value: {state.e.g}</>;
};

++state.a;
```

- 이 경우 state의 속성을 변경하지만 state.b.c 또는 state.e.g는 변경하지 않는다. 이 경우 두 컴포넌트를 리렌더링할 필요가 없다.
- 리렌더링 최적화의 핵심은 컴포넌트에서 state의 어느 부분이 사용될지 지정하는 것이다.

## state의 일부분을 지정하는 접근 방식

### 선택자 함수 사용

```jsx
const Component = () => {
  const value = useSelector((state) => state.b.c);

  return <>{value}</>;
};
```

```JSX

const Component = () => {

const value = useSelector((state) => state.b.c * 2);

return <>{value}</>

}

```

- 이제 이 컴포넌트가 state.b.c에만 관심이 있다는 것을 알고 있으므로 state.a가 변경된 경우에는 리렌더링을 피해야한다.
- 선택자 함수는 매우 유연해서 상태의 일부뿐만 아니라 파생된 값도 반환할 수 있다.
- 선택자 함수가 반환하는 값이 숫자와 같은 원시 값이 아니라 파생된 객체 값을 반환하는 경우에는 메모이제이션을 사용해 동일한 객체를 반환하도록 해야 한다.
- 선택자 함수는 컴포넌트의 어느 부분을 사용할지 명시적으로 지정하는 방법이므로 이를 수동 최적화라고 한다.

### 속성 접근 감지

상태 사용 추적(state usage tracking)은 속성 접근을 감지하고 감지한 정보를 렌더링 최적화에 사용할 수 있다.

```jsx
const Component = () => {
  // useTrackedState 훅은 상태 사용 추적 기능이 있다고 가정

  const trackedState = useTrackedState();

  return <>{trackedState.b.c}</>;
};
```

- trackedState가 .b.c 속성에 접근했음을 감지할 수 있고 .b.c 속성 값이 변경될 때만 useTrackedState가 리렌더링을 발생시킨다.
- 따라서 useTrackedState는 자동 렌더링 최적화라고 한다.
- useTrackedState를 구현하려면 상태 객체에 대한 속성 접근을 확인하기 위한 proxy가 필요하다.

#### useSelector와 useTrackedState의 차이점

```jsx
const Component = () => {
  const isSmall = useSelector((state) => state.a < 10);

  return <>{isSmall ? "small" : "big"}</>;
};
```

```jsx
const Component = () => {
  const isSmall = useTrackedState().a < 10;

  return <>{isSmall ? "small" : "big"}</>;
};
```

기능 면에서 두 예시 다 잘 작동하지만 useTrackedState는 state.a가 변경될 때마다 리렌더링되고 useSelector를 사용하면 isSmall이 변경될 때만 리렌더링된다.

### 아톰 사용

- 아톰은 리렌더링을 발생시키는데 사용되는 최소 상태 단위이다.
- 아톰을 사용하면 좀 더 세분화해서 구독하는 것이 가능하다.

```jsx
const globalState = {
  a: atom(1),

  b: atom(2),

  c: atom(3),
};

// atom 함수는 state 객체에서 아톰을 생성할 수 있다.

const Component = () => {
  // useAtom은 atom만 구독한다.

  const value = useAtom(globalState.a);

  return <>{value}</>;
};
```

아톰이 완전히 분리돼 있다면 별도의 전역 상태를 갖는 것과 거의 같다고 볼 수 있다. 하지만 아톰으로 파생 값을 만들 수 있다.

```js
const sum = globalState.a + globalState.b;
```

이 작업을 수행하기 위해서는 의존성을 추적해, 아톰이 갱신될 때마다 파생 값을 다시 평가해야 한다.

- 아톰과 파생 값의 정의는 명시적이지만 의존성 추적은 자동으로 된다.
- 아톰을 사용하는 접근 방식은 수동 최적화와 자동 최적화의 중간 정도로 볼 수 있다.

## 정리

특정 사용 사례에 대해 어떤 라이브러리가 적합한지 이해하기 위해 전역 상태 라이브러리를 선택할 때 라이브러리가 전역 상태를 읽고 작성하는 방법, 저장하는 위치, 리렌더링을 최적화하는 방법을 파악하는 것이 중요하다.
