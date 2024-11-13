# Valtio 살펴보기

## Main Concept 1. 프락시 활용

Valtio에서는 프락시를 사용하여 리렌더링을 감지한다.

> [!Note]
> **프락시** : ES6에서 도입된 JavaScript 기능. 객체의 기본적인 동작을 사용자가 재정의할 수 있게 해주는 객체이다.

프락시 API는 특별한 trap 메서드들을 정의한다. 이것들은 JavaScript가 제공하는 내부 메서드들을 가로채는 역할을 한다.

ex) set 트랩은 [[SET]] 내부 메서드를 가로챔.

예를 들어 set 핸들러를 추가한 프락시를 사용하면 객체의 값이 갱신되는 연산을 감지할 수 있다.

## Main Concept 2. 프락시 객체와 스냅숏

Valtio에서는 **프락시 객체**와 **스냅숏**이라고 불리는 두 가지 상태 업데이트 모델이 있다.

- 프락시 객체는 변경 가능한 원본 객체이다. `state.count++`같이 직접 수정이 가능하다.
- 스냅숏은 특정 시점의 프락시 객체 상태를 사진 찍듯이 고정시킨 불변 객체이다. `Object.freeze`로 동결되어 읽기만 가능하다.

snapshot 함수는 속성이 변경되었을 때만 새 스냅숏을 생성한다. 이는 중첩된 객체에 대해서도 작동한다. 예를 들어,

```tsx
const state2 = proxy({
  obj1: { c: 0 },
  obj2: { c: 0 },
});

const snap21 = snapshot(state2);

++state2.obj1.c;

const snap22 = snapshot(state2);
```

위와 같은 코드가 있을 때, snap21과 snap22는 서로 다른 참조를 가진다. 중첩된 객체의 경우 변경되지 않은 snap21.obj2와 snap22.obj2는 동일하게 유지되며, snap21.obj1과 snap22.obj1만 다른 참조를 가지게 된다.

> Valtio는 필요한 경우에만 스냅숏을 생성해서 메모리 사용량을 최적화하며, 최적화를 내부에서 실행하므로 개발자가 새로운 불변 상태를 생성하는 책임에서 자유롭다.

# How to use Valtio

Valtio를 활용한 코드 예제는 다음과 같다.

```tsx
import { proxy, useSnapshot } from "valtio";

const state = proxy({
  count1: 0,
  count2: 0,
});

const Counter1 = () => {
  // useSnapshot에서 반환하는 값의 변수명은 snap으로 설정하는 것이 Valtio의 관례이다.
  // useSnapshot에서 반환한 snap 객체는 Object.freeze로 동결되며 기술적으로 변경할 수 없다.
  const snap = useSnapshot(state);
  const inc = () => ++state.count1;
  return (
    <>
      {snap.count1} <button onClick={inc}>+1</button>
    </>
  );
};
```

위 코드를 보면, `proxy()`로 상태를 생성하고, `useSnapshot()`으로 이를 사용하고 있음을 알 수 있다.

**useSnapshot**은 `snapshot` 함수를 다른 프락시로 감싸서 구현한 함수로, 스냅숏 객체의 속성에 대한 접근을 감지하기 위해 사용된다.

return 부분에서 `snap.count1` 에 접근한 것은 useSnapshot 훅에 의해 추적 정보로 감지되며, 이를 기반으로 useSnapshot 훅은 필요한 경우에만 리렌더링을 유발한다.

> [!Note]
> 즉, 만약 snap.count1만 사용하는 Counter1과 snap.count2만 사용하는 Counter2가 있을 경우,
>
> - state.count1이 변경되면 → Counter1만 리렌더링
> - state.count2가 변경되면 → Counter2만 리렌더링

Valtio는 중첩된 객체와 배열을 지원하며, 이 경우에도 리렌더링 최적화가 완전하게 지원된다.

# 작은 애플리케이션 만들어 보기

Valtio는 애플리케이션 구조에 대해서는 관여하지 않는다.

전체 예제 코드는 [이 링크](https://stackblitz.com/edit/react-ts-fzbz1y?file=App.tsx)에 있으며 특징적인 부분만 남기면 다음과 같다.

### 상태 객체 정의 부분

```tsx
const state = proxy<{ todos: Todo[] }>({
  todos: [],
});
```

### 상태 객체 수정 부분(일반적인 JS 객체처럼 state에 접근)

```tsx
const createTodo = (title: string) => {
  state.todos.push({
    id: nanoid(),
    title,
    done: false,
  });
};

const removeTodo = (id: string) => {
  const index = state.todos.findIndex((item) => item.id === id);
  state.todos.splice(index, 1);
};

const toggleTodo = (id: string) => {
  const index = state.todos.findIndex((item) => item.id === id);
  state.todos[index].done = !state.todos[index].done;
};
```

### TodoItem, TodoList 호출 부분

```tsx
const TodoItem = ({
  id,
  title,
  done,
}: {
  id: string;
  title: string;
  done: boolean;
}) => {
  return (
    <div>
      <input type="checkbox" checked={done} onChange={() => toggleTodo(id)} />
      <span
        style={{
          textDecoration: done ? "line-through" : "none",
        }}
      >
        {title}
      </span>
      <button onClick={() => removeTodo(id)}>Delete</button>
    </div>
  );
};

const MemoedTodoItem = memo(TodoItem);

const TodoList = () => {
  const { todos } = useSnapshot(state);
  return (
    <div>
      {todos.map((todo) => (
        <MemoedTodoItem
          key={todo.id}
          id={todo.id}
          title={todo.title}
          done={todo.done}
        />
      ))}
    </div>
  );
};
```

만일 MemoedTodoItem에 todo 객체를 통째로 전달하면 useSnapshot이 속성 접근을 감지하지 못하므로, Valtio의 속성 추적 기능을 제대로 활용하기 위해 MemoedTodoItem에 각 속성을 분리해서 전달한다.

그러나 위와 같이 구현할 시 `todo.done` 속성에 대한 접근을 TodoList에서 감지하고 있으므로, 기존 요소에서 `toggleTodo` 를 통해 done 상태를 전환하면 대상 Item 컴포넌트뿐만 아니라 TodoList 컴포넌트도 리렌더링된다.

이를 개선하기 위해 TodoItem 컴포넌트에서 useSnapshot을 사용할 수 있다.

```tsx
const TodoItem = ({ id }: { id: string }) => {
  const todoState = state.todos.find((todo) => todo.id === id);
  if (!todoState) {
    throw new Error("invalid todo id");
  }
  const { title, done } = useSnapshot(todoState);
  return (
    <div>
      <input type="checkbox" checked={done} onChange={() => toggleTodo(id)} />
      <span
        style={{
          textDecoration: done ? "line-through" : "none",
        }}
      >
        {title}
      </span>
      <button onClick={() => removeTodo(id)}>Delete</button>
    </div>
  );
};

const MemoedTodoItem = memo(TodoItem);

const TodoList = () => {
  const { todos } = useSnapshot(state);
  const todoIds = todos.map((todo) => todo.id);
  return (
    <div>
      {todoIds.map((todoId) => (
        <MemoedTodoItem key={todoId} id={todoId} />
      ))}
    </div>
  );
};
```

변경된 구조에서 TodoList는 todo.id만 전달하고, TodoItem 컴포넌트에서 useSnapshot을 사용해 세부 속성에 접근한다.

이 경우 TodoList 컴포넌트는 배열에서 id의 순서가 변경되거나 id가 추가 혹은 제거되는 경우에만 리렌더링되므로, done 상태만 변경되는 경우에는 리렌더링되지 않는다.

이 두 접근 방식은 애플리케이션 규모가 크지 않은 이상 성능 차이가 미미하므로 개발자는 멘탈 모델에 따라 더 편한 코딩 패턴을 선택하면 된다.

# Valtio 접근 방식의 장단점

Valtio는 불변 갱신과 변경 가능한 갱신이라는 두 가지 상태 업데이트 모델이 있다.

자바스크립트는 변경 가능한 갱신을 허용하지만 리액트는 불변 상태를 중심으로 만들어졌으므로 두 모델을 같이 사용하는 경우 혼동하지 않아야 한다. 멘탈 모델 전환을 쉽게 할 수 있도록 Valtio의 상태와 리액트의 상태를 명확하게 분리하는 것이 도움이 된다.

## Valtio의 장점

Valtio 접근 방식의 장점으로는 네이티브 자바스크립트 함수를 사용할 수 있다는 것이다. 스프레드 연산자로 객체를 복잡하게 복사하는 대신, `state.a.b.text = 'hello';` 와 같이 간편하게 갱신할 수 있다.

또한 Valtio는 state를 사용할 때도 코드가 간편하다. 다음은 state를 활용할 때 Valtio와 Zustand 코드의 예시이다.

```tsx
//Valtio
const Component = ({ showText }) => {
  const snap = useSnapshot(state);
  return (
    <>
      {snap.count} {showText ? snap.text : ""}
    </>
  );
};

//Zustand
const Component = ({ showText }) => {
  const count = useStore((state) => state.count);
  const text = useStore((state) => (showText ? state.text : ""));

  return (
    <>
      {count} {text}
    </>
  );
};
```

선택자 기반 멘탈 모델에서는 조건이 많아질수록 훅이 더 많이 필요하므로 코드가 길어진다. 반면 프락시 기반 모델에서는 비교적 깔끔한 코드로 표현할 수 있다.

## Valtio의 단점

프락시 기반 렌더링 최적화의 단점은 예측 가능성이 떨어진다는 것이다. 프락시는 렌더링 최적화를 내부적으로 처리하기 때문에 동작이 명시적이지 않고, 디버깅하기가 어려울 수 있다.
