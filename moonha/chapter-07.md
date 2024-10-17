- 리액트의 **모듈 상태를 생성**하도록 설계된 작은 라이브러리
- **상태 각체를 수정할 수 없고 항상 새로 만들어야 하는 불변 갱신 모델을 기반으로 함**
- **렌더링 최적화는 선택자를 사용해 수동으로 함**
- 간단하면서도 강력한 `store` 생성자 인터페이스가 잇음

## 모듈 상태와 불변 상태 이해하기

- 상태를 유지하는 store를 만드는 데 사용되는 라이브러리
- 주로 **모듈 상태**를 위해 설계 됐으므로 모듈에서 `store`를 정의하고 내보내는 것을 할 수 있음
- **상태 객체 속성을 갱신할 수 없는 불변 상태 모델을 기반으로 함**
    - 상태를 변경하기 위해서는 새 객체를 생성해서 대체해야 하며, 수정하지 않은 객체는 재사용해야 함
    - 불변 상태 모델의 장점
        - **상태 객체의 참조에 대한 동등성만 확인하면 변경 여부를 알 수 있으므로 객체의 값 전체를 확인할 필요가 없음**

- `store` 생성
    
    ```tsx
    export const store = create(() => ({ count: 0}));
    ```
    
- `getState`, `setState`
    
    ```tsx
    store.getState()
    store.setState({ count: 1 });
    ```
    
- 상태는 반드시 새로운 객체를 이용해 갱신해야 함
- 함수 갱신도 가능
    
    ```tsx
    store.setState((prev) => ({ count: prev.count + 1 }));
    ```
    
- `store.setState`는 새 상태와 이전 상태를 병합하므로 설정하려는 속성만 지정해도 됨
    
    ```tsx
    const store = create(() => ({
    	const: 0, 
    	text: 'hello',
    }));
    
    store.setState({
    	count: 2
    });
    ```
    
    - 내부적으로 `Object.assign()`으로 구현됨
        
        ```tsx
        Object.assign({}, oldState, newState);
        ```
        
        - oldState와 newState 속성을 병합해서 새 객체를 반환
- `store.subscribe`
    - `store`의 상태가 변경될 때마다 호출되는 콜백 함수를 등록할 수 있음
        
        ```tsx
        store.subscribe(() => {
        	console.log('store state is changed');
        });
        ```
        
- **불변 상태 모델 및 구독이라는 아이디어를 중심으로 설계된 작고 가벼운 라이브러리**

## 리액트 훅을 이용한 리렌더링 최적화

- store 생성 시 리액트 훅의 명명 규칙을 따르기 위해 store 대신 `useStore`로 지정
    
    ```tsx
    const useStore = create(() => ({
    	count: 0,
    	text: "hello",
    })));
    ```
    
- 리렌더링을 피해야 하는 경우 **`useStore`에 선택자 함수를 지정할 수 있음**
    
    ```tsx
    const Component = () => {
    	const count = useStore((state) => state.count);
    
    	return <div>count: {count}</div>
    };
    ```
    
    - **수동 렌더링 최적화**
        - **선택자 기반 리렌더링 제어**
        - 리렌더링을 피하기 위해 선택자는 선택자 함수가 반환하는 결과를 비교하는 방식으로 작동함
        - **안정적인 결과를 반환하도록 선택자 함수 정의 시 주의해야 함**
            
            ```tsx
            const Component = () => {
            	const [{ count }] = useStore(
            		(state) => [{ count: state.count }]
            	);
            	
            	...	
            };
            ```
            
            - 선택자 함수가 새 객체를 포함해 새로운 배열을 생성하기 때문에 count 값이 변경되지 않은 경우에도 컴포넌트가 리렌더링 됨
        - **수동 렌더링 최적화의 장점**
            - **선택자 함수를 명시적으로 작성하기 때문에 동작을 정확히 예측 가능**
        - **수동 렌더링 최적화의 단점**
            - **객체 참조에 대한 이해가 필요**

## 읽기 상태와 갱신 상태 사용하기

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

- store에 미리 setState를 정의할 수 있음

```tsx
const selectCount2 = (state: StoreState) => state.count2;
const selectInc2 = (state: StoreState) => state.inc2;

const Counter2 = () => {
  const count2 = useStore(selectCount2);
  const inc2 = useStore(selectInc2);
  return (
    <div>
      count2: {count2} <button onClick={inc2}>+1</button>
    </div>
  );
};
```

- 상태 갱신 로직을 상태 값에 가깝게 배치하는 것은 Zustand의 setState가 이전 상태와 새로운 상태를 병합하기 위해서임 (모듈 상태와 불변 상태 이해하기 참고)

- 파생 상태
    
    ```tsx
    const Total = () => {
    	const count1 = useStore(selectCount1);
    	const count2 = useStore(selectCount2);
    	
      return (
        <div>
          total: {count1 + count2}
        </div>
      );
    };
    ```
    
    - count1이 증가하고 count2가 같은 양만큼 감소하여 합계가 변경되지 않는 경우에도 리렌더링이 발생하는 엣지 케이스가 있음
    - 파생 상태에 대한 선택자 함수를 사용
        
        ```tsx
        const selectTotal = (state: StoreState) => state.count1 + state.count2;
        
        const Total = () => {
          const total = useStore(selectTotal);
          
          return (
            <div>
              total: {total}
            </div>
          );
        };
        ```
        
        - 합계가 변경될 때만 리렌더링됨
    - store에서 합계를 생성
        
        ```tsx
        const useStore = create<StoreState>((set) => ({
          count1: 0,
          count2: 0,
          total: 0,
          inc1: () => set((prev) => ({
        		...prev,
        	  count1: prev.count1 + 1,
        	  total: prev.count1 + 1 + prev.count2, 
          })),
          inc2: () => set((prev) => ({
        		...prev,
        	  count2: prev.count2 + 1,
        	  total: prev.count2 + 1 + prev.count1, 
          })),
        }));
        ```
        
        - 여러 속성을 동시에 계산한 후 동기화 상태를 유지
            - Jotai가 이 문제를 잘 처리함

## 구조화된 데이터 처리하기

```tsx
type Todo = {
  id: number;
  title: string;
  done: boolean;
};

type StoreState = {
  todos: Todo[];
  addTodo: (title: string) => void;
  removeTodo: (id: number) => void;
  toggleTodo: (id: number) => void;
};
```

```tsx
let nextId = 0;

const useStore = create<StoreState>((set) => ({
  todos: [],
  addTodo: (title) =>
    set((prev) => ({
      todos: [
	      ...prev.todos, 
	      { id: ++nextId, title, done: false }
      ],
    })),
  removeTodo: (id) =>
    set((prev) => ({
      todos: prev.todos.filter((todo) => todo.id !== id),
    })),
  toggleTodo: (id) =>
    set((prev) => ({
      todos: prev.todos.map((todo) =>
        todo.id === id ? 
	        { ...todo, done: !todo.done } : 
	        todo
      ),
    })),
}));

```

- 모든 함수가 불변 방식으로 구현
    - 기존 객체와 배열을 변경하지 않고 새로운 객체와 배열을 생성

```tsx
const selectRemoveTodo = (state: StoreState) => state.removeTodo;
const selectToggleTodo = (state: StoreState) => state.toggleTodo;

const TodoItem = ({ todo }: { todo: Todo }) => {
  const removeTodo = useStore(selectRemoveTodo);
  const toggleTodo = useStore(selectToggleTodo);
  
  return (
    <div>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={() => toggleTodo(todo.id)}
      />
      <span
        style={{
          textDecoration: todo.done ? "line-through" : "none",
        }}
      >
        {todo.title}
      </span>
      <button onClick={() => removeTodo(todo.id)}>Delete</button>
    </div>
  );
};
```

- TodoItem의 메모된 버전
    
    ```tsx
    const MemoedTodoItem = memo(TodoItem);
    ```
    
    - 불필요한 리렌더링을 피하려면 메모된 컴포넌트를 사용하는 것이 중요
    - store 상태를 불변 방식으로 갱신하기 때문에 todos 배열에 있는 대부분의 todo 객체는 변경되지 않음. todo 객체를 MemoedTodoItem 객체에 전달한 후 변경되지 않으면 컴포넌트가 리렌더링되지 않음
    - todos 배열이 변경될 때마다 TodoList 컴포넌트는 리렌더링되지만 MemoedTodoItem 컴포넌트는 todo 항목이 변경되는 경우에만 리렌더링 됨
        
        ```tsx
        const selectTodos = (state: StoreState) => state.todos;
        
        const TodoList = () => {
          const todos = useStore(selectTodos);
          return (
            <div>
              {todos.map((todo) => (
                <MemoedTodoItem key={todo.id} todo={todo} />
              ))}
            </div>
          );
        };
        ```
        

- 새로운 Todo를 추가하는 기능
    
    ```tsx
    const selectAddTodo = (state: StoreState) => state.addTodo;
    
    const NewTodo = () => {
      const addTodo = useStore(selectAddTodo);
      
      const [text, setText] = useState("");
      
      const onClick = () => {
        addTodo(text);
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
    

## 이 접근 방식과 라이브러리의 장단점

- 읽기 및 쓰기 상태
    - 읽기 상태
        - 리렌더링을 최적화하기 위해 선택자 함수를 사용
    - 쓰기 상태
        - 불변 상태 모델을 기반으로 함

- 장점
    - Zustand 상태 모델은 리액트의 객체 불변성 규칙과 일치하다는 단순성
    - 번들 크기가 작음
- 단점
    - 선택자를 이용한 수동 렌더링 최적화
    - 객체 참조 동등성의 이해가 필요함
    - 선택자 코드를 위해 보일러플레이트 코드를 많이 작성해야 할 필요가 있음