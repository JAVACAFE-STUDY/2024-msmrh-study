# 사용사례 시나리오 1: Zustand

- Zustand는 주로 리액트의 모듈 상태를 생성하도록 설계된 라이브러리
- 상태 객체를 수정할 수 없고 항상 새로 만들어야 하는 불변 갱신 모델을 기반으로 한다.

## 모듈 상태와 불변 상태 이해하기

- Zustand는 상태를 유지하는 store를 만드는데 사용되는 라이브러리
- 모듈에서 store를 정의하고 내보냄
- 상태 객체 속성을 갱신할 수 없는 불변 상태 모델 기반
- 상태 변경을 위해서는 새 객체를 생성해서 대체 > 상태 객체의 참조에 대한 동등성만 확인하면 변경 여부를 알 수 있음 > 객체의 값 전체를 확인할 필요 없음

```JSX
import { create } from "zustand";
const store = create(() => ({ count: 0, text:"hello" }));
const countState = store.getState();
console.log(countState); // { count: 0, text:"hello" }

store.setState({ count: 1 });
const countState2 = store.getState();
console.log(countState2); // { count: 1, text:"hello" }
```

- store.setState()는 새 상태와 이전 상태를 병합하여 설정하려는 속성만 지정해도 된다.
- Object.assign()으로 구현된다.

#### Object.assign()

```js
const target = { a: 1, b: 2 };
const source = { b: 4, c: 5 };

const returnedTarget = Object.assign(target, source);

console.log(target);
// Expected output: Object { a: 1, b: 4, c: 5 }

console.log(returnedTarget === target);
// Expected output: true
```

#### store.subscribe(cb)

store의 상태가 변경될 때마다 호출되는 콜백 함수를 등록할 수 있다.

```jsx
const App = () => {
  const countState = useSyncExternalStore(store.subscribe, () =>
    store.getState()
  );
  return (
    <div>
      {countState}
      <button
        onClick={() => {
          store.setState({ count: countState + 1 });
        }}
      >
        +1
      </button>
    </div>
  );
};
```

## 리액트 훅을 이용한 리렌더링 최적화

```jsx
const count = useStore((state) => state.count);
return (
  <div>
    {count}
    <button
      onClick={() => {
        useStore.setState({ count: count + 1 });
      }}
    >
      +1
    </button>
    <Child />
  </div>
);
```

- 선택자 함수를 useStore의 인자로 넣어 수동 렌더링 최적화 진행
- 구독하는 상태의 메모리값이 변했으면 리렌더링이 일어남, object.assign()으로 상태를 병합하기 때문에 변하지 않은 상태가 객체형태라도 참조가 같아 리렌더링이 일어나지 않음

## 읽기 상태와 갱신 상태 사용하기

```tsx
const useStore = create<StoreState>((set) => ({
  count1: 0,
  count2: 0,
  inc1: () => set((prev) => ({ count1: prev.count1 + 1 })),
  inc2: () => set((prev) => ({ count2: prev.count2 + 1 })),
}));

const selectCount1 = (state: StoreState) => state.count1;
const selectInc1 = (state: StoreState) => state.inc1;

const Counter1 = () => {
  const count1 = useStore(selectCount1);
  const inc1 = useStore(selectInc1);

  return (
    <div>
      {count1}
      <button onClick={inc1}>+1</button>
    </div>
  );
};
```

- 상태 갱신 로직을 상태 값과 가깝게 배치하는 것은 Zustand의 setState가 이전 상태와 새로운 상태를 병합하기 위함 (Object.assign)

```tsx
//1)
const Total = () => {
  const count1 = useStore(selectCount1);
  const count2 = useStore(selectCount2);
  return <div>{count1 + count2}</div>;
};
//2)
const selectTotal = (state: StoreState) => state.count1 + state.count2;
const Total = () => {
  const total = useStore(selectTotal);
  return <div>{total}</div>;
};
//3)
const useStore = create<StoreState>((set) => ({
  count1: 0,
  count2: 0,
  total: 0,
  inc1: () =>
    set((prev) => ({
      count1: prev.count1 + 1,
      total: prev.count1 + 1 + prev.count2,
    })),
  inc2: () =>
    set((prev) => ({
      count2: prev.count2 + 1,
      total: prev.count1 + 1 + prev.count2,
    })),
}));
```

- 같은 값 만큼 count1이 + count2가 - 될 때 2,3은 리렌더링 되지 않음
- 3은 total을 동기화 시키는 로직 때문에 관리 포인트가 추가되어서 별로인 것 같다.

## 구조화된 데이터 다루기

- 상태를 불변방식으로 렌더링하는 것을 유념하고 상태 갱신하기

## 라이브러리와 접근 방식의 장단점

- 읽기상태 : 리렌더링 최적화를 위해 선택자 함수 사용
- 쓰기 상태 : 불변 상태 모델 기반
  장점 : 리액트와 동일한 불변 상태 모델 사용, 번들 크기가 작음
  단점 : 선택자 이용한 수동 렌더링 최적화, 선택자 코드를 위한 보일러플레이트 코드,
