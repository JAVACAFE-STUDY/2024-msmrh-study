# 마이크로 상태 관리 이해하기

## 리액트에서의 상태

- 사용자 인터페이스를 나타내는 모든 데이터
- 상태는 시간이 지남에 따라 변할 수 있으며, 리액트는 상태와 함께 렌더링할 컴포넌트를 처리함

## 리액트 훅이 나오기 전 상태 관리

- 상태 관리를 위한 범용 프레임워크(Redux, XState, MobX 등)를 사용해 개발자가 해당 프레임워크 내에서 목적에 맞게 해결
- 중앙 집중형 상태 관리 라이브러리를 사용
- 상태관리 라이브러리에 의존하는만큼 실제 사용하지 않는 기능까지 포함될 수 있음
- \*리액트 훅이 도입된 이후에 리액트를 사용했기 때문에 훅 도입 이전에는 어떻게 상태관리했을지 감이 잘 잡히지 않는다.

## 리액트 훅 등장 이후 상태 관리

- 상태 관리를 위해 기본적인 훅을 사용할 수 있음 (useState, useReducer 등)
- 특정 목적에 따라 다양한 해결책을 제공해줄 수 있게 되었음
- 리액트 훅을 기반으로 다양한 상태(폼, 서버 캐시, 내비게이션 등)을 해결하기 위한 라이브러리 등장

## 마이크로 상태 관리란?

- 범용적인 상태 관리를 위한 방법은 가벼워야함
- 요구사항에 따라 적절한 방법으로 리액트 상태를 관리하는 것
- 각 상태 관리 방법마다 서로 다른 기능을 가지게 됨
- \*최소한의 상태 관리 기능을 충족한 상태 관리 훅을 통해, 다양한 상태들(폼, 내비게이션, 서버 캐시 등)의 목적에 맞는 상태 관리 훅을 만들어 통일성 있게 상태를 관리하는 것

## 마이크로 상태 관리를 위한 필수적인 기능

- 상태관리의 기본 기능

  - 상태 읽기
  - 상태 갱신
  - 상태 기반 렌더링

- 부수적인 추가 기능
  - 리렌더링 최적화
    - \*useCallback(), useMemo()
  - 다른 시스템과의 상호 작용
    - \*react-router-dom의 훅들
    - \*useSearchParams()
  - 비동기 지원
    - \*react-query 훅들
  - 파생 상태
    - \*recoil의 selector로 선택하는 상태들
  - 간단한 문법

# 리액트 훅으로 마이크로 상태관리 하기

## 상태 관리 방법 구현을 위한 기본 리액트 훅

### useState

- 지역 상태를 생성하는 기본적인 함수
- 로직을 캡슐화하고 재사용 가능
- useState 기반으로 다양한 사용자 정의 훅을 만들 수 있음

### useReducer

- 지역 상태를 생성할 수 있음
- useState를 대체하는 용도

### useEffect

- 리액트 렌더링 프로세스 바깥에서 로직을 실행할 수 있음
- 리액트 컴포넌트 생명 주기와 함께 작동하는 기능을 구현할 수 있음
- 전역 상태를 다루기 위한 상태 관리 라이브러리를 개발할 때 중요

## 사용자 정의 훅(custom hook)을 통한 상태 관리

- 일반적으로는 useState를 wrapping하여 정의
- ui 컴포넌트와 상태관리 로직 추출, 명확한 함수명을 통한 가독성 향상의 목적
- \* `const { toggleItem, isItemSelected, selectedIds, setSelectedIds } = useMultipleSelect(list)`;

## 데이터 불러오기를 위한 서스펜스와 동시성 렌더링

- 리액트 훅은 서스펜스 혹은 동시성 렌더링과 함께 작동하도록 설계 및 개발되었다.

### [서스펜스 (suspense)](https://react.dev/reference/react/Suspense)

- 데이터 불러오기를 위한 서스펜스는 기본적으로 비동기 처리에 대한 걱정 없이 컴포넌트를 코딩할 수 있는 방법
- \*데이터 로딩 상태를 처리할 때 주로 사용
- \*lazy : 동적 임포트를 통해 컴포넌트를 필요한 시점에 로드하는 기능
- \*fallback : 컴포넌트가 로딩 중일 때 보여줄 UI를 설정하는 prop

### [동시성 렌더링 (concurrent rendering)](https://react.dev/blog/2022/03/29/react-v18#new-feature-transitions)

- 렌더링 프로세스를 청크 단위로 분할해서 중앙 처리 장치(CPU)가 장시간 차단되는 것을 방지하는 방법
- \*동시성 : 한번에 둘 이상의 작업이 동시에 진행되는 것
- \*startTransition, useTransition, useDeferredValue
- \*Automatic Batching : UI를 변경시키는 여러개의 상태를 한번에 업데이트하여 렌더링 횟수를 줄이는 방법
- [추가 예제](https://beomy.github.io/tech/react/concurrent-rendering/)

## 리액트 훅을 올바르게 사용하는 법

- state나 ref 객체를 직접 변경해서는 안된다.
  - 직접 변경하는 경우 리렌더링 되지 않거나, 너무 많은 렌더링이 발생하거나, 부분적인 리렌더링이 발생하는 등 예기치 않은 동작이 발생할 수 있다.
- 리액트 훅 함수와 컴포넌트 함수는 순수함수여야한다.
  - 함수가 여러 번 호출되더라도 일관되게 동작할 수 있어야한다.
- 이 규칙을 위반하는 경우 비동시성 렌더링에서는 문제없이 작동할 수 있어도 동시성 렌더링에서는 간헐적 문제가 발생할 수 있음

# 전역 상태 탐구하기

## 지역상태

- 컴포넌트에서 정의되고 컴포넌트 트리 내에서 사용되는 상태

## 전역상태

- 어플리케이션 내 서로 멀리 떨어져 있는 여러 컴포넌트에서 사용하는 상태
- 공유상태라고 부르기도함
- 전역상태는 싱글턴이 필요는 없음
- \*싱글턴 : 프로그램 내에서 객체가 단 하나만 존재하게 만드는 디자인 패턴, 자바스크립트에서는 전역 변수를 만들 경우 최상위 객체인 window에 포함되어 싱글턴처럼 사용할 수 있음

### 리액트에서 전역 상태 구현 및 관리가 어려운 이유

- 컴포넌트 모델에서는 서로가 격리되고 재사용이 가능하다는 것을 의미하는 `지역성(locality)`이 중요
- 컴포넌트가 컴포넌트 외부에 의존하는 경우 동작이 일관되지 않을 수 있으므로 재사용이 어려울 수 있음
- 따라서 컴포넌트 자체는 전역상태에 가급적 의존하지 않는 것이 좋다.

# useState 사용하기

## 값으로 상태 갱신하기

- `setState(1)`
- `setState(state+1)` > 렌더링 된 값에 기반함

## 함수로 상태 갱신하기

- `setState((prev)⇒prev+1)` > 이전 값에 기반함

## 지연 초기화

- useState는 첫 번째 렌더링에서만 평가되는 `초기화 함수`를 받을 수 있음

- useState 인자로 함수를 전달하는 경우의 예시

  useState 초기값으로 함수를 전달할 수 있는데 아래 2가지 경우에 따라 다르게 동작시킬 수 있다.

  ```jsx
  const init = () => {
    return initData;
  };

  // #1const [data, setData] = useState(init())

  // #2const [data, setData] = useState(init)
  ```

  - 1의 경우 : 해당 컴포넌트가 리랜더링 될 때마다 함수가 호출됨
  - 2의 경우 : 함수의 형태로만 전달하면 didmount와 같이 컴포넌트가 처음 랜더링 되는 시점에서만 함수가 호출됨(지연 평가)

- \*bail out - 리렌더링을 발생시키지 않는 것 (상태가 변하지 않으면 리렌더링이 발생하지 않는다.)

# useReducer 사용하기

## [기본 사용법](https://react.dev/reference/react/useReducer)

- `const [state, dispatch] = useReducer(reducer, initialArg, init?)`
- 복잡한 상태에 유용함

## 특징

- useReducer를 사용하면 미리 정의된 리듀서 함수와 초기 상태를 매개변수로 받아 리듀서 함수를 정의할 수 있음
- 훅 외부에서 리듀서 함수를 정의하면 코드를 분리할 수 있으며 테스트 용이성 측면에서 이점이 있음
- 리듀서는 순수 함수이기 때문에 동작을 `테스트` 하기가 더 쉽다.

## 지연초기화

- useReducer는 지연 초기화를 위해 init이라는 세 번째 선택적인 매개변수를 받을 수 있다.
- 세번째 매개변수의 init 함수는 컴포넌트가 마운트될 때 한 번만 호출되므로 무거운 연산을 포함할 수 있음
- useState와 달리 init 함수는 useReducer의 두 번째 인수인 initialArg를 인자로 받게됨

# useState와 useReducer 유사점과 차이점

- 리액트 내부에서는 useState는 useReducer로 구현되어 있으며 useState로 useReducer와 유사하게 구현할 수도 있다. 따라서 useState와 useReducer는 서로 상호 교환이 가능하여 선호도 혹은 프로그래밍 스타일에 따라 둘 중 하나를 선택하면 된다.

## useState 사용 시 상태 업데이트 예시

```jsx
function App() {
  const [todoValue, setTodoValue] = useState("");
  const [todoList, setTodoList] = useState<Todo[]>([]);

  const changeTodoValue = (value: string) => {
    setTodoValue(value);
  };

  const addTodo = (todo: Todo) => {
    setTodoList([...todoList, todo]);
  };

  const initTodoValue = () => {
    setTodoValue("");
  };

  const deleteTodo = (id: number) => {
    setTodoList(todoList.filter((t) => t.id !== id));
  };

  const toggleTodo = (id: number) => {
    setTodoList(
      todoList.map((t) => {
        if (t.id === id) {
          return { ...t, done: !t.done };
        }
        return t;
      })
    );
  };

  return (
    <div className="App">
      <div>
        <h1>Todo List</h1>
        <form
          onSubmit={(e) => {
            e.preventDefault();
            addTodo({ id: Date.now(), text: todoValue, done: false });
            initTodoValue();
          }}
        >
          <input
            type="text"
            value={todoValue}
            onChange={(e) => changeTodoValue(e.target.value)}
          />
          <button type="submit">Add</button>
        </form>

        <ul>
          {todoList.map((todo) => (
            <li key={todo.id}>
              <input
                type="checkbox"
                checked={todo.done}
                onChange={() => toggleTodo(todo.id)}
              />

              <span>{todo.text}</span>
              <button onClick={() => deleteTodo(todo.id)}>Delete</button>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}
```

## useReducer 사용 시 상태 업데이트 예시

```jsx
type TextInputAction =
  | {
      type: "CHANGE",
      payload: string,
    }
  | {
      type: "INIT",
    };

const textInputReducer = (state: string, action: TextInputAction) => {
  switch (action.type) {
    case "CHANGE":
      return action.payload;
    case "INIT":
      return "";
    default:
      return state;
  }
};

type TodoAction =
  | {
      type: "ADD",
      payload: Todo,
    }
  | {
      type: "DELETE" | "TOGGLE",
      payload: number,
    };

const todoReducer = (state: Todo[], action: TodoAction) => {
  switch (action.type) {
    case "ADD":
      return [...state, action.payload];
    case "DELETE":
      return state.filter((t) => t.id !== action.payload);
    case "TOGGLE":
      return state.map((t) => {
        if (t.id === action.payload) {
          return { ...t, done: !t.done };
        }
        return t;
      });
    default:
      return state;
  }
};

function App() {
  const [todoValue, dispatchTodoValue] = useReducer(textInputReducer, "");
  const [todoList, dispatchTodoList] = useReducer(todoReducer, []);

  return (
    <div className="App">
      <div>
        <h1>Todo List</h1>
        <form
          onSubmit={(e) => {
            e.preventDefault();
            dispatchTodoList({
              type: "ADD",
              payload: { id: Date.now(), text: todoValue, done: false },
            });
            dispatchTodoValue({ type: "INIT" });
          }}
        >
          <input
            type="text"
            value={todoValue}
            onChange={(e) =>
              dispatchTodoValue({ type: "CHANGE", payload: e.target.value })
            }
          />
          <button type="submit">Add</button>
        </form>

        <ul>
          {todoList.map((todo) => (
            <li key={todo.id}>
              <input
                type="checkbox"
                checked={todo.done}
                onChange={() =>
                  dispatchTodoList({ type: "TOGGLE", payload: todo.id })
                }
              />

              <span>{todo.text}</span>
              <button
                onClick={() =>
                  dispatchTodoList({ type: "DELETE", payload: todo.id })
                }
              >
                Delete
              </button>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}
```
