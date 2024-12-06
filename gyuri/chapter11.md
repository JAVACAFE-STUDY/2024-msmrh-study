# 세 가지 전역 상태 라이브러리의 유사점 과 차이점

Zustand, Jotai, Valtio

## Zustand와 Redux의 차이점

- 단방향 데이터 흐름 기반
- action을 실행하고 action을 통해 상태가 갱신된 후에 새로운 상태가 필요한 곳으로 전파됨

  - 디스패치와 전파를 분리하여 데이터 흐름을 단순화하고 시스템을 더 예축 가능하게 만든다.

- 상태를 갱신하는 방법

  - Redux 리듀서 기반 (리듀서: 이전 상태와 action을 받아 새로운 상태를 반환하는 순수 함수)
  - Zustand 반드시 리듀서를 사용해 상태 갱신할 필요는 없음

- 디렉터리 구조
  - Redux : features 디렉터리 구조, createSlice 함수는 기능 디렉터리 패턴을 따르도록 설계됨. 대규모 애플리케이션이 유용하다.
  - Zustand : 파일과 디렉터리 구조를 어떻게 가져갈지는 개발자의 몫.
- Immer
  - Redux : 사용함
  - Zustand : 사용하지 않음 (선택사항)
- 상태 전파
  - Redux : 컨텍스트 사용 (런타입 주입이 가능하여 일부 사용 사례에서는 더 효과적)
  - Zustand : 모듈 임포트 사용 (컨텍스트 사용은 선택적 지원)
- Redux Toolkit
  - Redux : 상태를 갱신하려면 액션을 디스패치 해야한다. (유지보수성, 확장성에 도움이됨)
  - Zustand : 데이터 흐름 측면에 대한 의견을 제시하지 않음. 라이브러리 지원이 없으므로 개발자가 처리해야한다.

## Jotai와 Recoil

- key 문자열의 존재. Jotai를 개발하는 큰 동기 중 하나는 key 문자열을 생략하는 것
  이다.
  구현 측면에서 Jotal는 WeakMap을 활용하고 아톰 객체의 참조에 의존한다.
  key 문자열의 장점은 직렬화(JSON stringify)가 가능하다
- Jotai의 atom 함수는 Recoil의 atom과 selector 두 가지 모두를 대체한다.
- Jotai의 공급자 제거 모드 （provider-less mode） : Provider 컴포넌트를 생략할 수 있게 해주는 기능

## Valtio와 MobX 사용하기

둘 다 변경 가능한 상태를 기반으로 하며 , 개발자가 직접 상태를 변경할 수 있다.

렌더링을 최적화하는 방법에 차이가 있다

Valtio는 훅을 사용하는 반면 , MobX 리액트는 고차 컴포넌트 （higher-order component; HoC) 를 사용한다．

> Valtio vs Immer
> 개념적으로는 비슷. Valtio는 변경 가능한 상태를 기반으로 함. 상태를 불변한 상태로 변환.
> Immer는 불변 상태를 기반으로 하며 변경 가능한 상태를 일시적으로 사용.

```tsx
import { proxy, useSnapshot } from "valtio";

// Valtio는 변경 가능한(mutable) 상태로 시작합니다
const state = proxy({
  count: 0,
  todos: [],
});

// 직접 상태를 변경할 수 있습니다
function addTodo(text) {
  state.todos.push({ text, completed: false }); // 직접 변경 가능
  state.count++;
}

// React 컴포넌트에서는 snapshot을 통해 불변 상태로 읽습니다
function TodoList() {
  const snap = useSnapshot(state);
  return (
    <div>
      <div>할 일 개수: {snap.count}</div>
      {snap.todos.map((todo) => (
        <div>{todo.text}</div>
      ))}
    </div>
  );
}
```

```tsx
import { produce } from "immer";
import { useState } from "react";

function TodoList() {
  // Immer는 불변(immutable) 상태로 시작합니다
  const [state, setState] = useState({
    count: 0,
    todos: [],
  });

  const addTodo = (text) => {
    // produce를 통해 임시로 변경 가능한 상태를 다룹니다
    setState(
      produce((draft) => {
        draft.todos.push({ text, completed: false });
        draft.count++;
      })
    );
  };

  return (
    <div>
      <div>할 일 개수: {state.count}</div>
      {state.todos.map((todo) => (
        <div>{todo.text}</div>
      ))}
    </div>
  );
}
```

## Zustand, Jotai, Valtio 비교하기

https://github.com/pmndrs

세 라이브러리가 공통으로 가지는 철학은 작은 API를 제공하는 것

### 상태가 어디에 위치하는가?

모듈 상태 (리액트에 속하지 않는 상태) : Zustand, Valtio
컴포넌트 상태 (리액트의 생명 주기에 제어되는 상태) : Jotai

### 상태 갱신 스타일은 무엇인가?

불변 상태 모델 : Zustand

- 객체 참조를 비교해서 변경 사항이 있는지 파악할 수 있다
  변경 가능한 상태 모델 : Valtio
- 객체가 깊이 중첩된 경우에 편리하다

Zustand와 Jotai에서도 상태 갱신을 할 때 상태에 직접 접근해서 변경할 수 있도록 Immer를 사용할 수 있다.
