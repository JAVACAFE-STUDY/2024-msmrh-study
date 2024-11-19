## React Tracked

- 속성 감지를 기반으로 자동으로 렌더링 최적화 수행
- 일반적으로 다른 상태관리 라이브러리(Redux, zustand)와 함께 사용
- 상태 관리 기능 제공 x, 렌더링 최적화 기능 제공

아래 코드는 불필요한 렌더링을 유발합니다.

```js
const NameContext = createContext([
  {
    firstName: "John",
    lastName: "Doe",
  },
  () => {},
]);

const NamePrinter = () => {
  const [name, setName] = useContext(NameContext);
  return (
    <div>
      {name.firstName} {name.lastName}
    </div>
  );
};

const useFirstName = () => {
  const [name, setName] = useContext(NameContext);
  return name.firstName;
};
```

위 코드는 잘 작동 하지만, firstName이 아닌 lastName을 변경해도 렌더링이 발생합니다.

이런 리렌더링 문제는 React Tracked를 사용하면 해결할 수 있다고 합니다.

useContext를 사용하는 대신 useTracked를 사용합니다.
useTracked는 proxy로 상태를 감싸고 사용을 추적합니다.

```js
const useFirstName = () => {
  const [{ firstName }] = useTracked();

  return firstName;
};
```

valtio와 거의 유사하다! (proxy-compare 라이브러리 사용)

### useState, useReducer와 함께 사용하는 방법

### useState와 함께 사용하기

```ts
import { useState } from "react";
import { createContainer } from "react-tracked";

const initialState = {
  count: 0,
  text: "hello",
};

const useMyState = () => useState(initialState);

export const { Provider: SharedStateProvider, useTracked: useSharedState } =
  createContainer(useMyState);
```

```ts
const Counter = () => {
  const [state, setState] = useSharedState();
  const increment = () => {
    setState((prev) => ({ ...prev, count: prev.count + 1 }));
  };
  return (
    <div>
      {state.count}
      <button onClick={increment}>+1</button>
      {Math.random()}
    </div>
  );
};
```

useState를 사용하는 것과 동일하게 사용할 수 있습니다.
createContext대신 createContainer를 사용하는 것으로도 리렌더링을 최적화할 수 있습니다.

=> react useReducer를 사용하는 예제또한 결국 같은 이야기를 하고 있기 때문에 생략하겠습니다.

결국 이절의 결론은, useContext를 대신해 useTracked를 사용하는 것으로 리렌더링을 최적화할 수 있다는 것입니다.


### react-redux와 함께 사용하기
redux는 useSelector라는 선택자를 이용해서 렌더링을 최적화 함
하지만 useSelector는 다음과 같은 이유로 한계가 있음(공식문서에서 설명된 내용)
Redux는 전체 애플리케이션의 상태를 하나의 큰 객치로 관리(이를, 단일 상태라 부름).
이 상태 객체는 많은 컴포넌트에서 사용이 될 수 있음.

문제는, 이 큰 상태의 객체가 변경될 때마다 모든 컴포넌트가 리렌더링 된다는 것이고. 이는 성능 저하로 이어질 수 있음
이를 해결하기 위해서 react-redux는 선택자를 제공함 이 선택자를 사용하면 특정 부분만 추출해서 사용할 수 있음. 

하지만, 선택자를 성능 향상을 위해서만 사용하는 건 좋지 않다. (공식문서에 나온 내용)
1. 복잡성 증가 
- 선택자를 사용하면 상태 관리 로직이 더 복잡해 질 수 있다. 
- 전체 어플리케이션 구조가 복잡해 진다. 

2. 이해의 어려움 
- 선택자를 사용한 성능 최적화를 위해서는 객체 참조 동등성에 대한 이해가 필요
여기서 객체 참조 동등성이란, 
JavaScript에서 객체는 참조 타입이기 때문에, 두 개의 변수가 동일한 객체를 가리킬 때만 참조 동등성이 성립합니다.

- 초보 개발자들에게 어렵고, 전문가들도 복잡한 상태 구조에서는 어려움을 겪을 수 있음

따라서, React Tracked와 같은 라이브러리를 사용하면 프록시를 통해 자동으로 상태 변경을 추적하고 re-render를 최적화 시킬 수 있다. 

react-redux를 사용할때, "createTrackedSelector"를 사용해서 선택자를 만들 수 있습니다. 
선택자 함수를 받아 선택자 함수의 결과를 반환하는 훅을 만들 수 있습니다.

 ### 향후 전망
 react-tracked내에서 사용하는 라이브러리는 아래 두개에 의존 합니다.
 - proxy-compare
 - use-sync-external-selector

이걸 쓰면 나중에 useCreateSelector와 비슷한 구현체가 리액트에서 제공되도 쉽게 마이그레이션 할 수 있을것이라고 한다. 





