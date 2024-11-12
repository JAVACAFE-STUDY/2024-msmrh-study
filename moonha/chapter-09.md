- `변경 가능한 갱신 모델(mutating update model)`을 기반으로 하는 전역 상태 관리 라이브러리
- 리렌더링을 최적화
    - `Zustand` 등 앞서 설명한 라이브러리와 다르게 리렌더링을 제어하기 위한 선택자가 필요하지 않음
    - `상태 사용 추적(state usage tracking)`이라는 기법을 기반으로 자동 렌더링 최적화를 함
        - 상태의 어느 부분이 사용되는지 감지할 수 있음
        - 사용된 부분이 변경될 경우에만 컴포넌트를 리렌더링되게 할 수 있음

## 또 다른 모듈 상태 라이브러리인 Valtio 살펴보기

- 상태를 불변으로 갱신하기 위해 `setState`를 사용하는 Zustand와 다르게 객체를 직접 수정함

```tsx
++moduleState.count;
```

- 프락시
    - 객체 연산을 감지하기 위한 핸들러를 만드는데 활용할 수 있음
        
        ```tsx
        const proxyObject = new Proxy({
        	count: 0,
        	text: "hello",
        }, {
        	set: (target, prop, value) => {
        		console.log("start setting", prop);
        		
        		target[prop] = value;
        		
        		console.log("end setting", prop);
        	},
        });
        ```
        
        - `++proxyObject.count` 을 실행하면 값이 변경되면서 `set` 핸들러의 코드가 실행됨

## 프락시를 활용한 변경 감지 및 불변 상태 생성하기

- `스냅숏(snapshot)`
    - 프락시를 사용해 변경 가능한 객체에서 생성된 변경 불가능한 객체

```tsx
import { proxy } from "valtio";

const state = proxy({ count: 0 });

const snap1 = snapshot(state);
```

- `proxy` 함수에서 반환하는 `state` 객체는 변경을 감지하는 프락시 객체이므로 불변 객체를 생성할 수 있음
    - `state`
        - 프락시로 감싼 변경 가능한 객체
        - state가 변경되면 Valtio에서 자동으로 추적
- `snapshot` 함수를 사용하여 불변 객체를 생성함
    - `snap1`
        - `Object.freeze`로 동결되어 변경 불가능한 객체
        - 컴포넌트에서 구독하는 값, 구독하는 값이 바뀔 때 리렌더링됨

```tsx
++state.count

const snap2 = snapshot(state);
```

- `snap1`과 `snap2`는 불변이므로 `snap1 === snap2`로 동등성을 확인하고 객체에서 어떤 부분이 다른지 알 수 있음

- 중첩된 객체에서도 작동하며 스냅숏 생성을 최적화함
    - `snapshot` 함수는 필요한 경우, 즉 속성이 변경될 때만 새 스냅숏을 생성함

- 최적화의 책임
    - `Zustand` 등 다른 라이브러리에서도 최적화를 수행할 수 있지만, 불변 상태를 적절하게 생성하는 것은 개발자의 책임임
    - `Valitio`는 최적화를 내부에서 실행함

```tsx
`Valtio`의 최적화는 이전 스냅숏에 대한 캐싱을 기반으로 함
- 즉, 캐시 공간이 하나라는 의미
- `++state.count`로 카운트를 늘린 다음, `--state.count`로 줄이면 새로운 스냅숏이 생성됨
```

## 프락시를 활용한 리렌더링 최적화

- `useSnapshot`
    - 구현은 `snapshot` 함수와 이를 감싸는 다른 프락시를 기반으로 함
    - `snapshot` 프락시는 `proxy` 함수에서 사용되는 프락시와 다른 목적이 있음
        - `snapshot` 프락시는 스냅숏 객체의 속성 접근을 감지하는데 사용됨

```tsx
const state = proxy({
  count1: 0,
  count2: 0,
});
```

- 초기 객체를 받아 새로운 프락시 객체를 반환함
- 원하는 대로 `state` 객체를 변경할 수 있음

- `state` 객체를 사용하고 `count1` 속성을 표시하는 컴포넌트를 정의

```tsx
const Counter1 = () => {
  const snap = useSnapshot(state);
  const inc = () => ++state.count1;
  return (
    <>
      {snap.count1} <button onClick={inc}>+1</button>
    </>
  );
};
```

- `useSnapshot`에서 반환하는 값의 변수명을 `snap`으로 설정하는 것이 `Valitio`의 관례
- `inc`는 `state` 객체를 변경하는 함수
- `snap` 객체는 `Object.freeze`로 동결되며 기술적으로 변경할 수 없음
- `snap1.count1` 은 `state` 객체의 `count1`  속성에 접근함
- 접근은 `useSnapshot` 훅에 의해 추적 정보로 감지되며, 추적 정보를 기반으로 `useSnapshot` 훅은 필요한 경우에만 리렌더링을 감지함

```tsx
const Counter2 = () => {
  const snap = useSnapshot(state);
  const inc = () => ++state.count2;
  return (
    <>
      {snap.count2} <button onClick={inc}>+1</button>
    </>
  );
};

const App = () => (
  <>
    <div>
      <Counter1 />
    </div>
    <div>
      <Counter2 />
    </div>
  </>
);
```

- 첫 렌더링에서 `state` 객체는 `{ count1: 0, count2: 0 }` 이며, 스냅숏 객체도 똑같은 값을 가짐
- `Counter1` 컴포넌트는 스냅숏 객체의 `count1` 속성에 접근하고 `Counter2` 컴포넌트는 스냅숏 객체의 `count2` 속성에 접근함
- 각 컴포넌트의 `useSnapshot` 훅은 추적 정보를 기억하고 알아낼 수 있음
    - 추적 정보는 컴포넌트가 어떤 속성에 접근했는지를 말함

- `Counter1` 컴포넌트의 버튼을 클릭하면 `state` 객체의 `count1` 속성이 증가
    - `state` 객체는 `{ count1: 1, count2: 0 }`
        - `Counter1` 컴포넌트는 새로운 숫자 1로 리렌더링됨
        - `count2`가 여전히 0이고 변경되지 않았기 때문에 `Counter2` 컴포넌트는 리렌더링되지 않음

- 추적 정보로 리렌더링을 최적화할 수 있음

- `Valtio`는 중첩된 객체와 배열을 지원함

## 작은 애플리케이션 만들어 보기

- 방법1
    - TodoList에서 useSnapshot 사용
        
        ```tsx
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
        

- 방법2
    - TodoItem에서 useSnapshot 사용
        
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
        ```
        
        - id 속성을 기반으로 todoState를 찾은 후 todoState를 사용해 useSnapshot에서 title, done 속성을 가져 옴
        - id, title, done 속성이 변경된 경우에만 리렌더링됨
    - TodoList
        
        ```tsx
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
        
        - TodoList 컴포넌트는 배열에서 id의 순서가 변경되거나 id가 추가 또는 제거되는 경우에만 리렌더링됨
        - 기존 요소에서 done 상태만 변경되는 경우에는 리렌더링되지 않음

## 이 접근 방식의 장단점

- 멘탈 모델
    - Valtio는 두 가지 상태 업데이트 모델이 있음
        - 불변 갱신, 변경 가능한 갱신
    - 자바스크립트 자체는 변경 가능한 갱신을 허용하지만 리액트는 불변 상태를 중심으로 만들어짐
        - 두 모델을 같이 사용하는경우 혼동하지 않도록 주의
    - Valito의 상태와 리액트의 상태를 명확하게 분리하여 멘탈 모델 전환을 쉽게 할 수 있음

- 장점
    - 변경 가능한 갱신의 가장 큰 장점
        - 네이티브 자바스크립트 함수를 사용할 수 있다는 것
            
            ```tsx
            array.splice(index, 1)
            
            [...array.slice(0, index), ...array.slice(index + 1)]
            ```
            
        - 중첩된 객체에서 값을 변경하는 경우
            
            ```tsx
            state.a.b.c.text = "hello";
            
            {
            	...state,
            	a: {
            		...state.a,
            		b: {
            			...state.b,
            			c: {
            				...state.a.b.c,
            				text: "hello",
            			},
            		},
            	},
            }
            ```
            

- 프락시 기반 렌더링 최적화를 사용해 애플리케이션 코드를 줄이는 데도 도움이 됨
    
    ```tsx
    const state = proxy({ count: 0, text: "hello" });
    
    // Valtio
    const Component = () => {
    	const { count } = useSnapshot(state);
    	
    	return <>{count}</>
    };
    
    // Zustand
    const Component = () => {
    	const count = useStore((state) => state.count));
    	
    	return <>{count}</>
    };
    ```
    
    ```tsx
    // Valtio
    const Component = ({ showText }) => {
    	const snap = useSnapshot(state);
    	
    	return <>{snap.count} {showText ? snap.text : ""}</>;
    };
    
    // Zustand
    const Component = () => {
    	const count = useStore((state) => state.count));
    	const text = useStore(
    		(state) => showText ? state.text : ""
    	);
    	
    	return <>{count} {text}</>
    };
    ```
    
    - 조건이 많아질수록 훅이 더 많이 필요해짐

- 단점
    - 프락시 기반 렌더링 최적화의 단점은 예측 가능성이 떨어짐
    - 렌더링 최적화를 내부적으로 처리하기 때문에 동작을 디버깅하기 더 어려울 수 있음