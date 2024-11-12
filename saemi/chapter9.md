# 사용 사례 시나리오 3: Valtio

- Valtio는 `변경 가능한 갱신 모델`(mutating update model)을 기반으로 하는 전역 상태 관리 라이브러리다.
- `모듈 상태용`으로 사용된다.
- 리액트와 통합을 위해 `프락시를 사용`해 `변경 불가능한 스냅숏`을 가져온다.
- API는 자바스크립트만으로 이뤄져 있으며 모든 작업이 내부에서 처리된다.
- 프락시를 활용해 상태 사용 추적(state usage tracking)이라는 기법을 기반으로 `자동 렌더링 최적화`를 한다.

## Valtio 살펴보기

- Valtio는 주로 모듈 상태에서 사용 되는 라이브러리라는 점에서 Zustand와 동일하다.
- 프락시 : 자바스크립트의 특수한 객체로서 객체 연산을 감지하기 위한 핸들러를 만드는데 활용할 수 있다.

```JS
const proxy = new Proxy(
  {
    count: 0,
    text: "hello",
  },
  {
    set: (target, prop, value) => {
      console.log("start setting", prop);
      target[prop] = value;
      console.log("end setting", prop);
    },
  }
);
```

- 프락시는 모든 변경을 감지할 수 있기 때문에 기술적으로 Zustand의 setState와 유사한 동작을 수행할 수 있다.

## 프락시를 활용한 변경 감지 및 불변 상태 생성하기

- Valtio는 프락시를 사용해 변경 가능한 객체에서 스냅숏이라고 하는 변경 불가능한 객체를 생성한다.

```JS
import { proxy, snapshot } from "valtio";
const state = proxy({ count: 0 });
const snap1 = snapshot(state);
```

- proxy 함수를 사용하면 프락시 객체로 감싼 변경 가능한 객체를 생성한다.
- snapshot 함수를 사용하면 불변 객체를 생성한다.
- state와 snap1은 {count:0}이지만 서로 다른 참조를 가진다.
  state는 프락시로 감싼 변경 가능한 객체인 반면, snap1은 Object.freeze로 동결되어 변경 불가능한 객체이다.

```JS
import { proxy, snapshot } from "valtio";
const state = proxy({ obj1: {c:0}, obj2: {c:0} });

const snap1 = snapshot(state)

++state2.obj1.c;

const snap2 = snapshot(state)

console.log(snap1 === snap2) // false
console.log(snap1.obj2 === snap2.obj2) // true

```

- snap21.obj2와 snap22.obj2의 참조가 동일하다는 것은 메모리를 공유한다는 의미다.
- Valtio는 필요한 경우에만 스냅숏을 생성해 메모리 사용량을 최적화한다.
- Valtio의 최적화는 이전 스냅숏에 대한 캐싱을 기반으로 한다. 즉, 캐시 공간이 하나라는 의미다. 따라서 ++state.count로 카운트를 늘린다음, --state.count로 줄이면 새로운 스냅숏이 생성된다.

## 프락시를 활용한 리렌더링 최적화

```js
import { proxy, useSnapshot } from "valtio";
const state = proxy({ count1: 0, count2: 0 });
const Counter1 = () => {
  const snap = useSnapshot(state);
  const inc = () => ++state.count1;

  return (
    <div>
      {snap.count1}
      <button onClick={inc}>+1</button>
    </div>
  );
};

export default Counter1;
```

- useSnapshot에서 반환하는 값의 변수명을 snap으로 설정하는 것이 Valtio의 관례다.
- snap 객체는 Object.freeze로 동결되며 기술적으로 변경할 수 없다.
- 접근은 useSnapshot 훅에 의해 추적 정보로 감지되며, 추적 정보를 기반으로 필요한 경우에만 리렌더링을 감지한다.

```js
const contrivedState = proxy({
  num: 123,
  str: "hello",
  arr: [1, 2, 3],
  nestedObj: { foo: "bar" },
  objArr: [{ a: 1 }, { b: 2 }],
});
```

- 일반적인 객체와 배열을 포함하는 객체, 깊게 중첩된 객체도 완전하게 지원된다.

## 작은 애플리케이션 만들어보기

```ts
const TodoItem = ({ id }: { id: number }) => {
  const todoState = state.todos.find((todo) => todo.id === id);

  if (!todoState) {
    throw new Error("Todo not found");
  }

  const { text, completed } = useSnapshot(todoState);

  return (
    <div>
      <input
        type="checkbox"
        checked={completed}
        onChange={() => toggleTodo(id)}
      />
      <span style={{ textDecoration: completed ? "line-through" : "none" }}>
        {text}
      </span>
      <button onClick={() => removeTodo(id)}>x</button>
    </div>
  );
};
```

- 리렌더링 최적화를 하기 위해 props 전달을 최소화하고 변동 가능성이 적은 ID를 기반으로 전역 상태에서 원하는 타겟을 찾아 렌더링하자

## 장단점

장점

- 변경 가능한 갱신 상태 모델으로 간단하게 상태를 변경할 수 있다.
- 프락시 기반 자동 렌더링 최적화로 선택자를 통한 렌더링 최적화 보다 코드량이 적음

단점

- 리액트와 다른 멘탈모델 -> Valtio의 상태와 리액트의 상태를 분리하자
- 프락시 기반 렌더링 최적화로 디버깅이 힘들어 예측 가능성이 낮아짐
