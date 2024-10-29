# Zustand Concepts

- 리액트의 모듈 상태를 생성하도록 설계된 작은 라이브러리.
- 상태 객체를 수정할 수 없고 항상 새로 만들어야 하는 불변 갱신 모델을 기반으로 함 ⇒ 객체 변경 여부를 알기 쉬움
- 선택자를 사용해 수동 렌더링 최적화
- store 생성자 인터페이스가 있어 모듈에서 store를 정의하고 내보낼 수 있음
- 4장 ‘구독을 이용한 모듈 상태 공유’의 아이디어와 유사하다.

# 사용 사례

## store 생성

```jsx
import { create } from "zustand";

export const store = create(() => ({ count: 0 }));
```

create를 통해 store를 생성할 수 있다.

## store 변경/조회

store는 getState, setState, subscribe 같은 기능을 사용할 수 있다.

```jsx
console.log(store.getState()); // ---> {count: 0}
store.setState({ count: 1 });
console.log(store.getState()); // ---> {count: 1}
```

상태가 불변이기 때문에 ++state.count처럼 변경하는 것은 불가능하다. 이렇게 변경한 경우 새로운 상태가 이전 상태와 동일한 참조를 가지기 때문에 라이브러리가 변경 사항을 제대로 감지할 수 없다.

따라서 setState({count: 2})와 같이 새로운 객체를 이용해 갱신해야 한다. 함수를 통해 갱신하는 것도 가능하다. 이 경우 파라미터에 이전 상태가 들어온다.

```jsx
store.setState((prev) => ({ count: prev.count + 1 }));
```

store.setState()는 새 상태와 이전 상태를 병합한다. 따라서 상태가 여러 속성을 가지고 있을 경우, 설정하려는 속성만 지정해도 된다.

```jsx
export const store = create(() => ({
  count: 0,
  text: "hello",
}));

store.setState({
  count: 2,
});

console.log(store.getState()); // ---> {count: 2, text: 'hello'}
```

setState는 내부적으로 Object.assign()으로 구현되며, 기존 state 객체와 인자로 들어온 객체를 병합한 새 객체를 state에 반환한다.

## store 구독

store.subscribe() 함수를 사용하면 store의 상태가 변경될 때마다 호출되는 콜백 함수를 등록할 수 있다.

```jsx
store.subscribe(() => {
  console.log("store state is changed");
});
store.setState({ count: 3 });
```

store.setState 문을 사용하면 구독으로 인해 ‘store state is changed’라는 메시지가 콘솔에 출력된다.

# 리액트 훅을 이용한 리렌더링 최적화

기본적으로 다음과 같이 store를 사용할 때,

```jsx
import { create } from "zustand";

const useStore = create(() => ({
  count: 0,
  text: "hello",
}));

const Component = () => {
  const { count } = useStore();
  return <div>count: {count}</div>;
};
```

이 컴포넌트는 화면과 관련 없는 text 값을 변경하더라도 리렌더링이 된다. [관련 stackblitz 링크](https://stackblitz.com/edit/react-ts-535ke4?file=package.json)

리렌더링을 피하기 위해서 useStore에 선택자 함수를 지정할 수 있다.

```tsx
const Component = () => {
  const count = useStore((state) => state.count);
  return <div>count: {count}</div>;
};
```

이렇게 할 경우, count 값이 변경될 때만 컴포넌트가 리렌더링된다. ⇒ 선택자 기반 리렌더링 제어(**수동 렌더링 최적화**)

이 방식에서는 선택자 함수가 반환하는 결과가 달라질 때에만 리렌더링된다.

<aside>
💡

선택자 함수를 정의할 때, 안정적으로 결과를 반환하도록 주의해야 한다. 예를 들어 다음과 같이 선택자 함수가 새로운 객체를 생성하도록 구현할 경우, 리렌더링 최적화의 효과를 볼 수 없다.

</aside>

```tsx
const Component = () => {
  const [{ count }] = useStore((state) => ({ count: state.count }));
  return <div>{count}</div>;
};
```

# 읽기 상태와 갱신 상태 사용하기

Zustand에서 상태를 읽고 갱신하는 몇 가지 패턴이 있다.

### 1. store에 함수 미리 정의하기

create 함수에 전달되는 store 생성자 함수는 몇 가지 인수를 받는데, 첫 번째 인수는 store의 setState 함수이다. 이 기능으로 다음과 같이 store 생성 시점에서 state를 핸들링하는 함수를 정의할 수 있다.

```tsx
type StoreState = {
  count1: number;
  count2: number;
  inc1: () => void;
  inc2: () => void;
};

const useStore = create<StoreState>((set) => ({
  count1: 0,
  count2: 0,
  inc1: () => set((prev) => ({ count1: prev.count1 + 1 })),
  inc2: () => set((prev) => ({ count2: prev.count2 + 1 })),
}));
```

이는 다음과 같이 사용된다.

```tsx
const selectCount1 = (state: StoreState) => state.count1;
const selectInc1 = (state: StoreState) => state.inc1;

const Counter1 = () => {
  const count1 = useStore(selectCount1);
  const inc1 = useStore(selectInc1);
  return (
    <div>
      count1: {count1} <button onClick={inc1}>+1</button>
    </div>
  );
};
```

만약 파생 상태를 생성하려면, 파생 상태에 대한 선택자 함수를 사용하면 된다.

```tsx
const selectTotal = (state: StoreState) => state.count1 + state.count2;

const Total = () => {
  const total = useStore(selectTotal);
  return <div>total: {total}</div>;
};
```

혹은 다음과 같이 store에서 합계를 별도의 값으로 저장할 수 있다.

```tsx
const useStore = create<StoreState>((set) => ({
  count1: 0,
  count2: 0,
  total: 0,
  inc1: () =>
    set((prev) => ({
      ...prev,
      count1: prev.count1 + 1,
      total: prev.count1 + 1 + prev.count2,
    })),
  inc2: () =>
    set((prev) => ({
      ...prev,
      count2: prev.count2 + 1,
      total: prev.count2 + 1 + prev.count1,
    })),
}));
```

# 구조화된 데이터 처리하기

숫자를 다루는 간단한 예제가 아닌, 객체 및 배열 조합을 처리하는 Todo 애플리케이션 예제를 Zustand를 통해 구현하면 다음과 같다.

[예제 Stackblitz 코드](https://stackblitz.com/edit/react-ts-ynlnn8?file=App.tsx)

주목할 점은 다음과 같다.

- addTodo, removeTodo, toggleTodo 함수는 todos state와 함께 store create 시점에서 정의된다.
- TodoItem은 memo()로 감싸서 todos 배열이 변경될 때마다 불필요한 리렌더링이 일어나지 않게 한다.
- selectTodos, selectAddTodo와 같은 셀렉터 함수는 컴포넌트 외부에 정의한다.

# Zustand 라이브러리의 장단점

### 장점

Zustand의 읽기 및 쓰기 상태는 다음과 같다.

- 읽기 상태 : 리렌더링을 최적화하기 위해 선택자 함수 사용
- 쓰기 상태 : 불변 상태 모델 기반

이는 React의 useState도 갖고 있는 객체 불변성 규칙(객체 내부의 값을 직접 수정해도 참조값이 달라지지 않으면 변경이 일어나지 않은 것으로 간주하고 리렌더링을 하지 않는 규칙)과 완전히 일치한다.

⇒ 리액트와 동일한 모델을 사용해 라이브러리의 단순성과 번들 크기가 작다는 이점

### 단점

Zustand는 선택자를 이용한 수동 렌더링 최적화를 사용하므로, 객체 참조 동등성을 이해해야 하며 선택자 코드를 위한 보일러플레이트 코드를 많이 작성해야 한다는 단점이 있다.

### 정리

Zustand는 리액트 원칙에서 간단한 기능을 추가한 라이브러리로,

1. 작은 번들 크기를 가진 라이브러리가 필요하거나
2. 참조 동등성 및 메모이제이션에 익숙하거나
3. 수동 렌더링 최적화를 선호하는 경우

Zustand를 추천한다.

# 덧. 리덕스와의 차이점

### 1. 단일 스토어 vs 다수 스토어

Redux는 단일 스토어 원칙을 따르며, 애플리케이션의 전체 상태를 하나의 스토어에서 관리하는 것을 권장한다.

반면 Zustand는 여러 개의 store를 만들고 사용하는 것이 가능하며, 각 기능이나 도메인별로 store를 분리할 수 있어 코드 관리가 용이하고 리셋/제거 메커니즘도 간단하다.

### 2. Context Provider의 유무

Redux는 Context Provider로 감싸줘야 하지만, Zustand는 그럴 필요가 없다.

### 3. 상태 업데이트 방식

Redux는 액션을 디스패치 → 순수 함수인 리듀서가 이를 처리 → 새로운 상태가 반환되는 식으로 업데이트가 이루어진다.

Zustand는 복잡한 데이터 흐름 없이 set 함수를 사용하여 직접적으로 상태를 업데이트할 수 있으며, 이는 더 간단하고 직관적인 API를 제공한다.
