# React Tracked

## 1. 개요

- 자동 렌더링 최적화를 제공하는 상태 사용 추적 라이브러리
- 다른 상태 관리 라이브러리와 함께 사용 가능
- Context API의 불필요한 리렌더링 문제를 해결

## 2. 주요 개념

### 상태 사용 추적

```tsx
// 기존 Context API 사용
const Component = () => {
  const state = useContext(MyContext); // 전체 상태 변경 시 리렌더링
  return <div>{state.someValue}</div>;
};

// React Tracked 사용
const Component = () => {
  const [state] = useTracked(); // 사용하는 값만 추적하여 필요할 때만 리렌더링
  return <div>{state.someValue}</div>;
};
```

### 프록시 기반 추적

- `proxy-compare` 라이브러리를 사용하여 상태 접근을 감시
- 실제로 사용되는 속성만 추적하여 최적화

## 3. 사용 방법

### useState와 함께 사용

```tsx
const useValue = () =>
  useState({
    user: { name: "김철수", age: 25 },
    settings: { theme: "dark", language: "ko" },
  });

const { Provider, useTracked } = createContainer(useValue);

// 이름만 사용하는 컴포넌트
const UserName = () => {
  const [state] = useTracked();
  // user.name이 변경될 때만 리렌더링
  return <div>{state.user.name}</div>;
};

// 테마만 사용하는 컴포넌트
const ThemeSwitch = () => {
  const [state, setState] = useTracked();
  // settings.theme이 변경될 때만 리렌더링
  return (
    <button
      onClick={() =>
        setState((prev) => ({
          ...prev,
          settings: {
            ...prev.settings,
            theme: prev.settings.theme === "dark" ? "light" : "dark",
          },
        }))
      }
    >
      현재 테마: {state.settings.theme}
    </button>
  );
};
```

### Redux와 함께 사용

```tsx
// Redux 스토어 설정
const initialState = {
  todos: [],
  filter: "all",
};

// React Tracked 설정
const useTrackedState = createTrackedSelector<typeof initialState>(useSelector);

// 필터만 사용하는 컴포넌트
const FilterButtons = () => {
  const { filter } = useTrackedState();
  const dispatch = useDispatch();

  return (
    <div>
      <button
        onClick={() => dispatch({ type: "SET_FILTER", filter: "all" })}
        disabled={filter === "all"}
      >
        전체
      </button>
      <button
        onClick={() => dispatch({ type: "SET_FILTER", filter: "active" })}
        disabled={filter === "active"}
      >
        진행중
      </button>
    </div>
  );
};
```

## 4. 장점과 특징

1. **자동 최적화**

- 수동으로 최적화 로직을 작성할 필요가 없음
- 실제 사용되는 데이터만 추적하여 불필요한 리렌더링 방지

2. **유연성**

- useState, useReducer, Redux 등 다양한 상태 관리 방식과 호환
- 기존 코드를 크게 수정하지 않고도 도입 가능

3. **개발자 경험**

- 직관적인 API
- 보일러플레이트 코드 최소화

## 5. 사용 시 주의사항

1. **깊은 객체 구조**

```tsx
// 주의해야 할 패턴
const state = {
  deeply: {
    nested: {
      value: 123,
    },
  },
};

// 권장되는 플랫한 구조
const state = {
  deeplyNestedValue: 123,
};
```

2. **성능 고려사항**

- 매우 큰 상태 객체의 경우 프록시 생성 비용 고려
- 필요한 경우 상태를 더 작은 단위로 분할

이러한 특징들로 인해 React Tracked는 특히 중간 규모의 애플리케이션에서 Context API를 대체하거나 Redux와 함께 사용할 때 좋은 선택이 될 수 있습니다.
