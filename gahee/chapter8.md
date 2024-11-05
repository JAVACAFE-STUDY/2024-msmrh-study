### Jotai: 전역 상태 관리를 위한 가벼운 라이브러리

Jotai는 전역 상태 관리를 위해 **원자적(atomic) 접근 방식**을 도입한 작은 React 상태 관리 라이브러리입니다. 간단하게는 `useState` 대체 용도로, 복잡하게는 엔터프라이즈급 애플리케이션에서까지 사용할 수 있을 만큼 확장성이 뛰어납니다. 그럼에도 **라이브러리 크기가 작아 리소스를 적게 사용**하고, **불필요한 리렌더링을 줄여 성능을 최적화**하는 장점을 가지고 있습니다.

이 글에서는 Jotai의 특징과 기본 사용법을 살펴보고, 이 라이브러리의 내부 동작 방식을 이해하는 데 도움이 될 만한 자료를 소개하겠습니다.

---

### Jotai가 작은 이유는 무엇일까?

Jotai는 전역 상태를 관리하는 방식으로 **atom**을 사용합니다. **Atom**이란 하나의 상태를 나타내는 작은 상태 조각으로, 각 atom은 독립적이며 필요한 경우 서로 결합해 상태를 구성할 수 있습니다. 이러한 구조는 **필요한 상태만 관리**하고, **의존성에 따라 필요한 부분만 리렌더링**할 수 있게 하여 효율적인 상태 관리를 가능하게 합니다.

React Context를 사용할 때 흔히 발생하는 **불필요한 리렌더링 문제**나 **복잡한 메모이제이션**의 필요성을 없앰으로써, Jotai는 **최소한의 코드와 리소스로 전역 상태를 관리할 수 있습니다**. 또한, Jotai는 선언적인 프로그래밍 모델을 유지하며, 시그널과 유사한 개발자 경험을 제공합니다.

---

### Atom: Jotai의 기본 단위

Jotai는 상태를 atom이라는 최소 단위로 쪼갭니다. **Atom은 독립적인 하나의 상태**로, 각 atom이 변경될 때마다 해당 atom을 사용하는 컴포넌트만 리렌더링됩니다. 즉, atom은 **리렌더링을 감지하는 최소 단위**로 작동하여, 필요한 부분만 업데이트하도록 최적화합니다.

```js
import { atom, useAtom } from "jotai";
const countAtom = atom(0);

const Count1 = () => {
  const [count] = useAtom(countAtom);
  return <div>{count}</div>;
};

const Count2 = () => {
  const [count] = useAtom(countAtom);
  return <div>{count}</div>;
};

const App = () => {
  return (
    <>
      <Count1 />
      <Count2 />
    </>
  );
};
```

이 예제에서 `countAtom`이라는 atom을 정의하고, `useAtom` 훅을 통해 `Count1`과 `Count2` 컴포넌트에서 동일한 `countAtom` 상태를 공유합니다. **Provider 없이도 두 컴포넌트가 같은 상태를 공유**하며, 상태 변경 시 필요한 컴포넌트만 리렌더링됩니다. React Context를 사용할 때는 `Provider`로 감싸야 하는 번거로움이 있지만, Jotai는 이러한 과정 없이 간결하게 상태를 공유할 수 있습니다.

### 불변 상태 모델과 구독 방식

Jotai는 **불변 상태 모델**을 채택하여 atom의 상태가 변경될 때마다 새로운 상태를 생성하고, **필요한 컴포넌트만 업데이트**하도록 설계되었습니다. `useAtom` 훅을 사용하면 atom의 현재 값을 읽고, 상태가 변경될 때 이를 구독하여 필요한 컴포넌트만 자동으로 리렌더링합니다. 이를 통해 불필요한 리렌더링을 방지하고 **성능을 최적화**합니다.

---

### Jotai의 기본 사용법

Jotai는 React Context와 비교했을 때, 다음과 같은 코드 간소화를 제공합니다.

#### React Context를 사용한 상태 관리 코드 예제

```js
import { createContext, useContext, useState } from "react";

const CountContext = createContext();

const CountProvider = ({ children }) => {
  return (
    <CountContext.Provider value={useState(0)}>
      {children}
    </CountContext.Provider>
  );
};

const App = () => {
  return (
    <CountProvider>
      <Count1 />
      <Count2 />
    </CountProvider>
  );
};

const Count1 = () => {
  const [count] = useContext(CountContext);
  return <div>{count}</div>;
};

const Count2 = () => {
  const [count] = useContext(CountContext);
  return <div>{count}</div>;
};
```

#### Jotai를 사용한 상태 관리 코드 예제

```js
import { atom, useAtom } from "jotai";

const countAtom = atom(0);

const Count1 = () => {
  const [count] = useAtom(countAtom);
  return <div>{count}</div>;
};

const Count2 = () => {
  const [count] = useAtom(countAtom);
  return <div>{count}</div>;
};

const App = () => {
  return (
    <>
      <Count1 />
      <Count2 />
    </>
  );
};
```

Jotai를 사용하면 **Provider 없이도 상태를 관리**할 수 있으며, 코드가 훨씬 간결해집니다. 필요한 경우에만 선택적으로 `Provider`를 사용해 여러 스토어를 생성할 수 있습니다. 예를 들어, 하위 트리마다 서로 다른 상태를 사용해야 할 때는 `Provider`를 통해 별도의 스토어를 생성하여 다른 값을 가진 atom을 사용할 수 있습니다.

---

### Jotai의 내부 구조 이해

Jotai의 내부 동작 원리는 GitHub 코드로 살펴볼 수 있습니다. Jotai는 전역적으로 관리되는 기본 스토어를 사용하며, 모든 atom은 이 스토어에 등록됩니다.

- **Atom 생성**: `atom` 함수는 새로운 상태 단위를 만들고, 기본 스토어에 등록할 수 있는 객체를 생성합니다. (참고: [atom.ts 코드](https://github.com/pmndrs/jotai/blob/main/src/vanilla/atom.ts))
- **기본 스토어 관리**: Provider 없이도 자동으로 생성되는 전역 스토어에서 atom을 관리합니다. (참고: [store.ts 코드](https://github.com/pmndrs/jotai/blob/main/src/vanilla/store.ts))

---

### 동적 atom 생성과 메모리 관리

Jotai의 또 다른 강점은 **React 컴포넌트 생명주기에 따라 atom을 동적으로 생성하고 소멸**할 수 있다는 점입니다. 이를 위해 Jotai는 **WeakMap 객체**를 사용하여 atom을 관리합니다.

1. **WeakMap으로 atom 관리**: WeakMap은 더 이상 사용되지 않는 atom을 자동으로 메모리에서 해제하므로, **메모리 사용을 효율적으로 관리**할 수 있습니다.
2. **atom 함수로 atom 생성**: `atom` 함수는 새로운 atom을 만들고, 이 atom은 WeakMap에 저장됩니다.
3. **useAtom 훅을 통한 atom 구독**: `useAtom` 훅을 사용하면 해당 atom을 구독하게 되어, atom 값이 변경될 때 자동으로 리렌더링됩니다. 이를 통해 **불필요한 리렌더링을 줄여 성능을 최적화**합니다.

---

### Jotai의 렌더링 최적화

Jotai는 전역 상태 관리를 위해 **상향식 접근법을 사용**합니다. Jotai에서의 상향식 접근이란, 필요한 상태만을 작은 단위로 분리하고, **atom의 의존성을 추적하여 필요한 컴포넌트만 업데이트**하는 방식입니다. 이를 통해 **불필요한 리렌더링을 방지하고 성능을 최적화**할 수 있습니다.

### 상향식 접근법이란?

상향식 접근법에서는 **상태를 세밀하게 쪼개고, 필요한 데이터만 선택적으로 가져오는 방식**을 사용합니다. Jotai에서는 atom을 사용하여 상태를 아주 작은 단위로 관리하고, atom 간의 의존성을 자동으로 추적하여 필요한 부분만 리렌더링합니다. 이 방식의 장점은 다음과 같습니다.

- **필요한 데이터만 사용**: 각 컴포넌트는 필요한 atom만 구독하기 때문에, 변경이 발생할 때 필요한 컴포넌트만 리렌더링됩니다.
- **의존성 추적을 통한 최적화**: atom 간 의존 관계가 자동으로 관리되므로, 복잡한 상태 변경이 있어도 **Jotai가 알아서 필요한 부분만 업데이트**합니다.
- **메모이제이션 불필요**: 상태 변경 시 atom이 필요한 컴포넌트만 리렌더링하므로, React Context처럼 복잡한 메모이제이션이나 `useMemo`, `useCallback` 등을 사용할 필요가 없습니다.

### 파생 atom (Derived Atom)

Jotai에서 **파생 atom**(Derived Atom)은 다른 atom을 기반으로 새로운 값을 계산하는 atom입니다. 이를 통해 기존 atom의 일부 값만을 선택하거나, 특정 계산을 수행하여 파생된 값을 사용할 수 있습니다. 예를 들어, 전체 사용자 목록에서 특정 조건을 만족하는 사용자만 필터링하여 보여주는 atom을 만들 수 있습니다.

```javascript
import { atom, useAtom } from "jotai";

const userListAtom = atom([
  { id: 1, name: "Alice", active: true },
  { id: 2, name: "Bob", active: false },
]);

// 파생 atom: active 상태인 사용자만 필터링
const activeUserListAtom = atom((get) =>
  get(userListAtom).filter((user) => user.active)
);

const ActiveUsers = () => {
  const [activeUsers] = useAtom(activeUserListAtom);
  return (
    <ul>
      {activeUsers.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};
```

위 코드에서 `activeUserListAtom`은 `userListAtom`을 기반으로 **활성화된 사용자만 필터링**하는 파생 atom입니다. `userListAtom`이 변경되면 Jotai가 이를 추적하여 `activeUserListAtom`을 자동으로 업데이트하며, `ActiveUsers` 컴포넌트만 리렌더링합니다.

### Jotai에서의 렌더링 최적화

1. **작은 단위로 쪼갠 atom**:
   - Jotai에서는 상태를 atom으로 작게 나누어 필요한 부분만 구독하게 합니다. 이렇게 하면 상태가 변경될 때 해당 atom을 사용하는 컴포넌트만 리렌더링됩니다.
2. **의존성 추적**:
   - atom 간의 의존 관계를 자동으로 추적하여, 필요한 경우에만 파생 atom이 재평가되고, 이를 사용하는 컴포넌트만 업데이트됩니다.

## Jotai가 아톰 값을 저장하는 방식

### 구체적인 동작 방식

1. **Atom은 상태 값 자체가 아니라 상태를 정의하는 객체**

   - Jotai에서 `atom`은 **상태를 직접 저장하는 컨테이너가 아니라**, 상태가 어떻게 정의되어야 하는지를 설명하는 **구성 객체**입니다.
   - 예를 들어, `const countAtom = atom(0);`이라고 선언하면, `countAtom`은 초기값 `0`을 가지고 상태를 정의한 구성 객체가 됩니다. 이때 실제 값 `0`은 `countAtom`에 직접 저장되는 것이 아니라, **Jotai 내부의 기본 스토어**에 저장됩니다.
   - `countAtom`은 단지 **어떤 상태가 필요하고, 초기값이 무엇인지 정의하는 역할**을 합니다.

2. **실제 값은 Jotai의 기본 스토어에 저장**

   - Jotai는 **기본 스토어(WeakMap 기반)를 통해 모든 atom의 실제 값을 저장하고 관리**합니다. 이 스토어는 atom을 키로, 해당 atom의 실제 상태 값을 값으로 저장합니다.
   - 예를 들어, `countAtom`을 `useAtom` 훅을 통해 사용할 때, Jotai는 내부 스토어에서 `countAtom`에 연결된 실제 값을 가져와서 컴포넌트에 전달합니다.

3. **`useAtom`으로 atom의 값을 접근하는 방식**
   - 컴포넌트에서 `useAtom(countAtom)`을 호출하면, Jotai는 내부 스토어에서 `countAtom`에 연결된 실제 값을 가져와 컴포넌트에 전달합니다.
   - 이렇게 함으로써 atom 자체는 상태를 직접 가지고 있지 않고, 그 상태가 어떻게 정의되고 관리될지를 설명하는 역할만 합니다. **상태 값 자체는 Jotai의 스토어에 저장되고 컴포넌트가 `useAtom`으로 접근할 때 읽어와 사용**하는 구조입니다.

## 배열 구조 추가하기
`atoms in atom` 패턴은 **Jotai에서 배열 같은 리스트 데이터를 효율적으로 관리**하기 위해 사용합니다. 배열의 각 항목을 **독립적인 작은 atom**으로 관리하고, 이를 **하나의 큰 atom 안에 배열 형태로 묶어서** 관리하는 방식입니다.

### 왜 `atoms in atom` 패턴이 필요한가?

보통 배열 데이터를 하나의 큰 atom에 모두 넣어 관리하면, 배열의 일부가 변경될 때도 **전체 배열이 리렌더링**됩니다. 예를 들어, 배열의 첫 번째 항목만 바꾸고 싶은데 전체 배열이 다시 렌더링된다면, 이는 비효율적입니다.

`atoms in atom` 패턴을 사용하면 **배열의 각 항목을 독립적으로 관리**할 수 있어, 특정 항목이 변경될 때 **그 항목만 리렌더링**됩니다. 이렇게 하면 **불필요한 리렌더링을 줄여 성능을 최적화**할 수 있습니다.

### `atoms in atom` 패턴의 동작 방식

- **기본 아이디어**: 배열의 각 항목을 독립된 atom으로 만든 뒤, 이 작은 atom들을 배열처럼 모아주는 **상위 atom**을 생성합니다.
- 예를 들어, `[item1, item2, item3]`라는 배열이 있다면 `item1`, `item2`, `item3` 각각을 작은 atom으로 만들고, 이들을 묶어주는 큰 atom(`listAtom`)을 만들어서 배열을 관리하는 것입니다.

### 어떤 점이 개선되는가?

1. **리렌더링 최적화**: 배열 중 하나의 항목만 변경될 때도, 그 항목에 해당하는 atom만 업데이트되므로 **전체 배열을 다시 렌더링할 필요가 없습니다**.
2. **배열 요소 추가 및 삭제가 쉬워짐**: 각 항목이 독립적인 atom으로 관리되기 때문에, 배열에 요소를 쉽게 추가하거나 삭제할 수 있습니다.

### 간단한 예

```javascript
import { atom, useAtom } from 'jotai';

// 1. 각 항목을 독립적인 atom으로 만듦
const createItemAtom = (initialValue) => atom(initialValue);

// 2. 여러 item atom을 묶어 배열로 관리하는 상위 atom
const listAtom = atom(() => [createItemAtom(1), createItemAtom(2), createItemAtom(3)]);
```

위 코드에서는 `listAtom`이 `[item1, item2, item3]`처럼 각 항목을 별도의 atom으로 포함하고 있습니다. 이제 `listAtom`을 사용해 배열처럼 접근할 수 있습니다.

```javascript
// 배열의 개별 항목에 접근 및 리렌더링 최적화 예시
const ListItem = ({ itemAtom }) => {
  const [value, setValue] = useAtom(itemAtom);
  return (
    <div>
      <span>{value}</span>
      <button onClick={() => setValue(value + 1)}>+</button>
    </div>
  );
};

const List = () => {
  const [itemAtoms] = useAtom(listAtom);
  return (
    <div>
      {itemAtoms.map((itemAtom, index) => (
        <ListItem key={index} itemAtom={itemAtom} />
      ))}
    </div>
  );
};
```


