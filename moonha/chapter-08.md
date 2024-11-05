- Zustand와 비교
    - 차이점
        - 컴포넌트 상태를 사용
    - 공통점
        - 불변 상태 모델
- 내부적으로 컨텍스트를 사용하고 아톰 자체는 값을 가지지 않기 때문에 모듈 상태와 달리 한 번 정의한 아톰을 재사용할 수 있음

→ "값을 가지지 않는다"

- 아톰은 값이 저장되는 "저장소" 역할이 아니라는 의미. 아톰은 일종의 "상태를 정의하는 규칙"이나 "상태의 열쇠"로서만 존재하고, 실제 값은 따로 저장소(`store`)에 담기게 됨
- 아톰은 단지 상태에 대한 규칙(식별자)만을 정의하고, 값은 저장소에서 가져오기 때문에 다른 컴포넌트나 모듈에서도 이 아톰을 재사용할 수 있음
- Jotai의 스토어는 `WeakMap` 객체로 구성되며, 여기에서 아톰을 키(key)로 사용해 실제 값을 저장
    - 예를 들어, 카운터 아톰이 있다고 하면, 이 아톰이 키가 되고, 현재 카운터 값(예: 0, 1, 2 등)이 WeakMap의 값이 됨
    - 상태를 아톰이 아닌 `WeakMap`에 담아두기 때문에, 아톰이 여러 컴포넌트나 모듈에서 사용될 때도 동일한 값을 참조하게 됨

```tsx
아톰 자체는 값이 아닌 상태 규칙만 정의하고,
실제 값은 WeakMap을 기반으로 한 Jotai 스토어에서 관리되며,
각 컴포넌트는 아톰을 통해 이 스토어에 접근해 실제 값을 가져오거나 업데이트함
```

```tsx
// Jotai 내부적으로 스토어가 WeakMap을 통해 아톰과 값을 저장
const store = new WeakMap();

// 예를 들어, counterAtom을 생성하면 store에 추가
store.set(counterAtom, 0); // 초기 값이 0으로 설정됨

// Counter 컴포넌트에서 setCount를 호출해 값을 1로 변경하면
store.set(counterAtom, 1); // WeakMap의 해당 아톰의 값이 업데이트됨
```

## Jotai 이해하기

### 구문 단순성

- 아톰은 하나의 상태를 나타냄
- 리렌더링을 감지하는 최소 단위
- atom 함수는 아톰 정의를 생성함

```tsx
const countAtom = atom(0);

const Counter1 = () => {
  const [count, setCount] = useAtom(countAtom);
  const inc = () => setCount((c) => c + 1);
  return (
    <>
      {count} <button onClick={inc}>+1</button>
    </>
  );
};
```

- 공급자가 필요하지 않음
    - 서로 다른 하위 트리에 각각 다른 값을 제공해야 하는 경우 공급자를 선택적으로 사용하면 됨

### 동적 아톰 생성

- 아톰은 리액트 컴포넌트 생명 주기에서 생성되거나 소멸될 수 있음
- 다중 컨텍스트 접근 방식
    - 새로운 상태를 추가한다는 것은 새로운 Provider 컴포넌트를 추가한다는 것을 의미하기 때문에 새로운 Provider가 추가되면서 기존 컴포넌트들이 다시 마운트되고, 일부 상태가 초기화될 위험이 있음
- Jotai 방식
    - 새로운 아톰을 정의해 상태를 생성할 수 있으므로, 컴포넌트가 마운트될 때 필요한 아톰을 동적으로 생성하고, 언마운트 시 자동으로 소멸되게 할 수 있음, 새로운 상태를 추가해도 다른 컴포넌트들이 영향을 받지 않음

## 렌더링 최적화

- 하향식(top-down) 접근법
    - 모든 것을 저장하는 store를 생성하고 필요에 따라 store에서 상태를 선택하는 방식
- 상향식(bottom-up) 접근법
    - 작은 아톰을 만들고 이를 결합해 더 큰 아톰을 만듦
    - 컴포넌트에 사용될 아톰만 추가해서 리렌더링을 최적화할 수 있음

- 파생 아톰
    - 기존 아톰에서 또 다른 아톰을 생성
        
        ```tsx
        const personAtom = atom((get) => ({
        	firstName: get(firstNameAtom),
        	lastName: get(lastNameAtom),
        	age: get(ageAtom),
        }));
        ```
        

- 선택자 접근 방식의 문제점
    
    ```tsx
    const selectFullName = (state) => ({
    	firstName: state.firstName,
    	lastname: state.lastName,
    });
    
    const PersonComponent = () => {
    	const person = useStoreSelector(store, selectFullName);
    	
    	return <>{person.firstName} {person.lastName}</>
    };
    ```
    
    - age가 변경되면 selectFullName 함수가 다시 평가되고 동일한 속성 값을 가진 새로운 객체를 반환함
        - 이로 인해 원하지 않은 리렌더링이 발생

- 아톰 모델의 장점
    - 아톰의 구성이 컴포넌트에 표시되는 것과 쉽게 연관지을 수 있다는 점
    - 리렌더링 제어가 간단함
    - 아톰을 이용한 리렌더링 최적화에는 사용 정의 동등 함수나 메모이제이션이 필요하지 않음


## Jotai가 아톰 값을 저장하는 방식 이해하기

- atom 함수로 만들어진 원시 아톰은 해당 값을 가지지 않음
- 값을 저장하는 store가 별도로 존재함
- store는 아톰 구성 객체를 키, 아톰 값을 값으로 가지는 WeakMap 객체임
- useAtom을 사용하면 기본 store에 접근하여 해당 값에 접근할 수 있음

→ 앞서 언급한 아톰은 값을 가지지 않는다, WeakMap으로 되어 있다는 얘기가 세 번 정도 반복됨    

- Provider라는 컴포넌트를 제공하며, 컴포넌트 레벨에서 store를 생성하는 것도 가능
    
    ```tsx
    const countAtom = atom(0);
    
    const Counter = () => {
      const [count, setCount] = useAtom(countAtom);
      const inc = () => setCount((c) => c + 1);
      return (
        <>
          {count} <button onClick={inc}>+1</button>
        </>
      );
    };
    
    const App = () => (
      <>
        <Provider>
          <h1>First Provider</h1>
          <div>
            <Counter />
          </div>
          <div>
            <Counter />
          </div>
        </Provider>
        <Provider>
          <h1>Second Provider</h1>
          <div>
            <Counter />
          </div>
          <div>
            <Counter />
          </div>
        </Provider>
      </>
    );
    ```
    
    - 동일한 countAtom을 사용하지만 두 Provider 컴포넌트는 서로 다른 스토어를 사용함
    - countAtom 자체가 값을 갖고 있지 않기 때문에 countAtom은 여러 Provider 컴포넌트에 재사용할 수 있음
        - 모듈 상태와 다른 주목할 만한 차이점

## 배열 구조 추가하기 (아톰 속 아톰들 (Atoms-in-Atom))

```tsx
type Todo = {
  title: string;
  done: boolean;
};

type TodoAtom = PrimitiveAtom<Todo>;

const todoAtomsAtom = atom<TodoAtom[]>([]);

const TodoItem = ({
  todoAtom,
  remove,
}: {
  todoAtom: TodoAtom;
  remove: (todoAtom: TodoAtom) => void;
}) => {
  const [todo, setTodo] = useAtom(todoAtom);
  return (
    <div>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={() => setTodo((prev) => ({ ...prev, done: !prev.done }))}
      />
      <span
        style={{
          textDecoration: todo.done ? "line-through" : "none",
        }}
      >
        {todo.title}
      </span>
      <button onClick={() => remove(todoAtom)}>Delete</button>
    </div>
  );
};

const MemoedTodoItem = memo(TodoItem);

const TodoList = () => {
  const [todoAtoms, setTodoAtoms] = useAtom(todoAtomsAtom);
  const remove = useCallback(
    (todoAtom: TodoAtom) =>
      setTodoAtoms((prev) => prev.filter((item) => item !== todoAtom)),
    [setTodoAtoms]
  );
  return (
    <div>
      {todoAtoms.map((todoAtom) => (
        <MemoedTodoItem
          key={`${todoAtom}`}
          todoAtom={todoAtom}
          remove={remove}
        />
      ))}
    </div>
  );
};

const NewTodo = () => {
  const [, setTodoAtoms] = useAtom(todoAtomsAtom);
  const [text, setText] = useState("");
  const onClick = () => {
    setTodoAtoms((prev) => [...prev, atom<Todo>({ title: text, done: false })]);
    setText("");
  };
  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={onClick} disabled={!text}>
        Add
      </button>
    </div>
  );
};

```

- 배열 아톰은 아톰이 요소인 배열을 보관하는 데 사용됨
- 아톰 구성은 문자열로 평가할 수 있으며 UID를 반환함
- 요소를 렌더링하는 컴포넌트는 각 컴포넌트에서 아톰 요소를 사용함, 이렇게 하면 요소의 값을 쉽게 변경할 수 있고 리렌더링을 자연스럽게 피할 수 있음

→ 아톰을 고유한 key값으로 사용할 수 있다는걸 처음 알게 됨
→ 여러 아톰을 하나의 단위로 관리, 동적 상태 관리에 유리, 리렌더링 최적화(개별 아톰 단위로 리렌더링을 제어하므로 일부 아톰만 변경되어도 다른 아톰은 리렌더링되지 않음) 등의 이점이 있어보임

## Jotai의 다양한 기능 사용하기

### 아톰의 write 함수 정의하기

```tsx
const doubledCountAtom = atom(
	(get) => get(countAtom) * 2,
	(get, set, arg) => set(countAtom, arg/2)
);
```

### 액션 아톰 사용하기

- 상태를 변경하는 코드를 위해 함수 또는 함수 집합을 만드는 경우
- atom의 두 번째 인수인 write 함수만 사용

```tsx
const countAtom = count(0);

const incrementCountAtom) = atom(
	null,
	(get, set, arg) => set(countAtom, (c) => c + 1)
);
```

### 아톰의 onMount 옵션 이해하기

- 아톰이 사용되기 시작할 때 특정 로직을 실행하고 싶은 경우
- ex) 외부 데이터를 구독
- onMount 옵션 사용

```tsx
const countAtom = atom(0);

countAtom.onMount = (setCount) => {
	console.log("count atom 사용을 시작합니다");

	const onUnmount = () => {
		console.log("count atom 사용이 끝났습니다");
	};

	return onUnmount;
};
```

### jotai/utils 번들 소개하기

- atomWithStorage

### 라이브러리 사용법 이해하기

- 스코프(scope)라는 개념을 제공

### 고급 기능 소개

- 서스펜스 지원
- 다른 라이브러리와의 통합을 지원

