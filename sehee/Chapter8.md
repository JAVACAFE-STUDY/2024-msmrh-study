# Jotai의 특징

- 컨텍스트 및 구독 패턴을 기반으로 함
- 상태의 작은 조각인 ‘아톰’을 모델로 삼음
- 불변 상태 모델
- 내부적으로 컨텍스트를 사용하고 아톰 자체는 값을 가지지 않기 때문에, 모듈 상태와는 달리 한번 정의한 아톰을 재사용할 수 있음

# Simple Usage

### 상태 만들기

```jsx
const countAtom = atom(0);
```

Jotai에서 아톰은 작은 상태 조각이며 리렌더링을 감지하는 최소 단위이다. `atom()` 함수는 아톰 정의를 생성하며, `useState` 와 비슷한 문법을 사용한다. (초깃값을 인수로 받고, `[state, setState]` 형태의 튜플을 반환)

따라서 다음과 같이 atom을 적용할 수 있다.

```jsx
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

아톰을 사용할 때는 공급자가 필요하지 않다. 컨텍스트의 기본 스토어가 있기 때문이다.

서로 다른 하위 트리에 대해 각각 다른 값을 제공해야 하는 경우 공급자를 선택적으로 사용하면 된다.

```jsx
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

# Jotai의 이점 : 구문 단순성

Context를 활용하여 state를 정의할 땐 각 상태마다 컨텍스트를 만들어야 하므로, 상태를 추가할 때마다 다음과 같은 보일러 플레이트 코드가 필요하다.

```tsx
const CountProvider = ({ children }: { children: ReactNode }) => (
  <CountContext.Provider value={useState(0)}>{children}</CountContext.Provider>
);

const App = () => <CountProvider>...</CountProvider>;
```

그러나 Jotai 아톰으로는 Provider를 별도로 구현할 필요가 없으므로, 아톰이 많아지더라도 다음과 같이 각 아톰에 대한 코드를 한 줄만 추가하면 된다.

```tsx
const countAtom = atom(0);

//컴포넌트에서는 다음과 같이 사용
const [count, setCount] = useAtom(countAtom);
```

따라서 Jotai의 문법은 훨씬 단순하다.

# Jotai의 이점 : 동적 아톰 생성

Context 방식에서 새로운 상태를 추가한다는 것은 새로운 Provider 컴포넌트를 추가한다는 것을 의미하기 때문에, 리액트 컴포넌트 생명 주기에서 생성되거나 소멸되는 것이 불가능하다.

그러나 아톰은 리액트 컴포넌트 생명주기에서 생성되거나 소멸할 수 있다.

# Jotai의 이점 : 재사용성

유사한 아톰의 경우 props로 받아 별도의 컴포넌트 정의 없이 재사용할 수 있다.

```tsx
const count1Atom = atom(0);
const count2Atom = atom(0);

const Counter = ({ countAtom }) => {
  const [count, setCount] = useAtom(countAtom);
  const inc = () => setCount((c) => c + 1);
  return <button onClick={inc}>{count}</button>;
};
```

위와 같이 정의된 컴포넌트는 countAtom 구성과 유사한 모든 아톰에 대해 재사용할 수 있다.

> [!Note]
> 아톰은 props로 전달할 수도 있고, 모듈 수준의 상수나 컨텍스트, 또는 다른 아톰의 값 등 여러 가지 방법으로 전달할 수도 있다.

# Jotai에서의 렌더링 최적화 방식(+ 파생 아톰)

Zustand에서는 store에 모든 것을 저장하고 필요에 따라 상태를 선택하는 선택자 기반 방식으로 렌더링 최적화를 했다. 이러한 접근 방식을 **하향식(top-down) 접근법**이라고 한다.

Jotai에서는 반대로 **상향식(buttom-up) 접근법**을 활용한다. 즉, 작은 아톰을 만들고 이를 결합해 더 큰 아톰을 만드는 방식이다. 각 컴포넌트에서는 사용될 아톰만 추가해서 리렌더링을 최적화할 수 있다.

Jotai에서는 기존 아톰에서 또 다른 아톰을 만들 수 있는 **파생 아톰**이라는 개념을 제공한다.

```tsx
const firstNameAtom = atom("React");
const lastNameAtom = atom("Hooks");
const ageAtom = atom(3);

const personAtom = atom((get) => ({
  firstName: get(firstNameAtom),
  lastName: get(lastNameAtom),
  age: get(ageAtom),
}));
```

`atom()` 함수는 인수로 `read` 함수를 받을 수 있는데, 이 함수는 다른 아톰을 참조하고 그 값을 가져올 수 있는 `get` 을 인수로 받는다.

따라서 위에서 정의한 `personAtom`은 `firstNameAtom`, `lastNameAtom`, `ageAtom`의 값을 가져오며 각 아톰들이 변경될 때마다 갱신된다. (=의존성 추적)

> [!Note]
> 의존성 추적은 동적이며 조건부 평가에서도 작동한다. 예를 들어 read 함수가 `(get) ⇒ get(a) ? get(b) : get(c)`이면, 의존성은 a가 참일 경우 a와 b이고 a가 거짓일 경우 a와 c이다.

> [!Note]
> 파생 아톰은 read 함수의 결과이므로 읽기 전용이다. 따라서 파생 아톰 값을 설정한다는 개념은 없다.

이러한 상향식 모델의 장점은 아톰의 구성이 컴포넌트에 표시되는 것과 쉽게 연관 지을 수 있다는 점이다.

예를 들어 스토어와 선택자 접근 방식으로 리렌더링 최적화를 구현할 때, 다음 코드는 원하는 대로 동작하지 않는다.

```tsx
const selectFullName = (state) => ({
  firstName: state.firstName,
  lastName: state.lastName,
});

//컴포넌트 안에서 다음과 같이 사용
const person = useStoreSelector(store, selectFullName);
```

만약 `state.age`가 변경되면 `selectFullName` 함수가 다시 평가되고, 동일한 속성 값을 가진 새로운 객체를 반환하므로, `useStoreSelector`는 리렌더링을 유발한다. 이를 방지하기 위해서는 사용자가 직접 동등 함수를 만들거나 메모이제이션을 사용해야 한다.

그러나 아톰 모델에서는 이러한 접근 방식을 채택하지 않기 때문에 아톰을 이용한 렌더링 최적화에는 사용자 정의 동등 함수나 메모이제이션이 필요하지 않다.

# Jotai가 아톰 값을 저장하는 방식

아톰 구성은 **직접 해당 값을 가지지 않으며**, 아톰 값을 저장하는 store는 따로 있다.

store에는 키가 아톰 구성 객체이고 값이 아톰 값인 [WeakMap](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/WeakMap) 객체가 있다.

> [!Note]
> WeakMap : 자바스크립트 표준 내장 객체. 키가 반드시 객체이며, 객체 키가 가비지 컬렉터에 의해 정리될 때 연결된 데이터도 함께 사라지는 동작 방식을 갖고 있음.

useAtom은 기본적으로 모듈 수준에서 정의된 기본 store를 사용하지만, Jotai가 제공하는 Provider 컴포넌트를 사용해서 컴포넌트 레벨에서 store를 생성하는 것도 가능하다. [예시](https://stackblitz.com/edit/react-ts-ujntev?file=App.tsx)

> 아톰 자체가 값을 갖고 있지 않으므로, 아톰은 여러 Provider 컴포넌트에 재사용할 수 있다.

# 배열 구조 추가하기

Jotai에서는 아톰을 작게 쪼갤 수 있기 때문에, 배열 구조의 state를 쉽게 구현할 수 있다. **아톰 속 아톰(Atoms-in-Atom)** 패턴을 통해 이를 구현할 수 있다.

일반적으로 Todo 배열을 아톰으로 정의한 예시 코드를 보면 [다음](https://stackblitz.com/edit/react-ts-9pbawd?file=App.tsx)과 같다.

이렇게 구현할 때, 개발자적 관점에서는 두 가지 우려되는 점이 있다.

1. 단일 요소를 변경하기 위해 todos 배열 전체를 갱신해야 한다. toggleTodo 함수에서는 모든 요소를 순회하여 하나의 요소만 변경하는데, 하나의 요소에만 접근해서 간단히 변경하는 게 바람직하다.
2. 요소에 key로 사용할 id를 따로 넣어주고 있는데, 이를 사용하지 않는 것이 좋다.

이 문제는 Atoms-in-Atom 패턴으로 해결할 수 있다. 즉, 각 Todo 요소를 아톰으로 만들고 해당 요소들을 넣는 전체 리스트 배열 역시 아톰 값으로 정의하는 패턴이다.

아톰 요소와 리스트는 다음과 같이 정의하고 사용한다.

```tsx
type Todo = {
  title: string;
  done: boolean;
};

type TodoAtom = PrimitiveAtom<Todo>;

const todoAtomsAtom = atom<TodoAtom[]>([]);
```

전체 코드는 [다음](https://stackblitz.com/edit/react-ts-gfayss?file=App.tsx)과 같다.

TodoList 구현을 보면, key로 아톰을 넣고 있는 것을 볼 수 있다.

아톰 구성은 문자열로 평가될 때 유일한 식별자(unique identifier, UID)를 반환하므로 별도의 id 문자열을 관리할 필요가 없다.

또한, 배열 요소 중 하나가 toggleTodo를 통해 갱신되더라도 todoAtomsAtom은 변경되지 않으므로 TodoList 컴포넌트의 리렌더링이 줄어든다.

Atoms-in-Atom 패턴을 사용할 때의 차이점을 요약하면 다음과 같다.

- 배열 아톰은 아톰으로 이루어진 배열을 보관하는 데 사용된다.
- 배열에 새로운 요소를 추가하려면 새로운 아톰을 생성해서 추가해야 한다.
- 아톰 구성은 문자열로 평가할 수 있으며 UID를 반환한다.
- 요소를 렌더링하는 컴포넌트는 각 컴포넌트에서 아톰 요소를 사용한다. 이럴 경우 요소의 값을 쉽게 변경할 수 있고 리렌더링을 자연스럽게 피할 수 있다.

# Jotai의 다양한 기능 사용하기

## 아톰의 write 함수 정의하기

atom 함수는 첫 번째 인수인 read 함수 외에 선택적인 두 번째 인수인 write 함수를 받을 수 있다.

파생 아톰을 만들었던 read 함수가 다음과 같이 쓰인다면,

```tsx
const countAtom = atom(0);
const doubledCountAtom = atom((get) => get(countAtom) * 2);
```

write함수를 사용해 doubledCountAtom 함수에 쓰기가 가능하도록 구현할 수 있다.

```tsx
const doubledCountAtom = atom(
  (get) => get(countAtom) * 2,
  (get, set, arg) => set(countAtom, arg / 2)
);
```

write 함수는 세 개의 인수를 받는다.

- get : 아톰의 값을 반환하는 함수
- set : 아톰의 값을 설정하는 함수
- arg : 아톰을 갱신할 때 받을 임의의 값(이 경우 doubledCountAtom)

이렇게 생성된 아톰은 마치 원시 아톰처럼 사용할 수 있다. 만약 갱신 함수까지 사용할 수 있게끔 만들고 싶다면 다음과 같이 구현할 수 있다.

```tsx
const anotherCountAtom = atom(
  (get) => get(countAtom),
  (get, set, arg) => {
    const nextCount = typeof arg === "function" ? arg(get(countAtom)) : arg;
    set(countAtom, nextCount);
  }
);
```

## 액션 아톰 사용하기

상태를 변경하는 코드를 위해 함수 또는 함수 집합을 만드는 경우가 있다. 이를 목적으로 아톰을 사용할 수 있다.

액션 아톰은 atom 함수의 두 번째 인수인 write 함수만 사용한다. 예시 코드는 다음과 같다.

```tsx
const countAtom = count(0);
const incrementCountAtom = atom(null, (get, set, arg) =>
  set(countAtom, (c) => c + 1)
);
```

이 아톰은 다음과 같이 평범한 아톰처럼 사용하며, 반환값은 무시한다.

```tsx
const IncrementButton = () => {
  const [, incrementCount] = useAtom(incrementCountAtom);
  return <button onClick={incrementCount}>Click</button>;
};
```

## 아톰의 onMount 옵션 이해하기

아톰이 사용되기 시작할 때 특정 로직을 실행하고 싶을 수 있다. 이 작업은 useEffect 훅으로 수행할 수도 있지만, 아톰 수준에서 로직을 작성할 수 있게끔 onMount 옵션이 제공된다.

onMount 함수는 아톰을 변경하는 setAtom 함수를 인수로 받으며 언마운트될 때 실행되는 onUnmount 함수를 리턴한다.

```tsx
const anAtom = atom(1)
anAtom.onMount = (setAtom) => {
  console.log('atom is mounted in provider')
  setAtom(c => c + 1) // increment count on mount
  return () => { ... } // return optional onUnmount function
}
```

## jotai/utils 번들 소개하기

Jotai 라이브러리는 기본 번들에서 두 가지 기본 함수인 atom과 useAtom, 그리고 Provider 컴포넌트를 제공한다. 다양한 유틸리티 함수는 jotai/utils라는 별도의 번들에서 제공된다.

## 라이브러리 사용법 이해하기

만약 내부적으로 Jotai를 사용하는 두 라이브러리를 사용하는 애플리케이션을 개발하면 이중 공급자 문제가 발생한다. (Jotai 아톰은 참조로 구분되기 때문에 첫 번째 라이브러리의 아톰이 실수로 두 번째 라이브러리의 공급자에 연결될 수 있음)

Jotai는 특정 공급자에 연결하는 스코프(scope)라는 개념을 제공하며, Provider 컴포넌트와 useAtom 훅에 동일한 스코프 변수를 전달하면 이 문제를 예방할 수 있다.

## 고급 기능 소개

- 서스펜스 기능 지원 : Jotai는 리액트 서스펜스를 지원하여, 파생 아톰의 read 함수가 Promise를 반환하면 useAtom 훅이 일시 중단되고 리액트는 폴백을 표시한다.
- Jotai는 다른 라이브러리와 유연하게 통합될 수 있다. 외부 데이터 소스의 경우 onMount 옵션이 필요하다.
