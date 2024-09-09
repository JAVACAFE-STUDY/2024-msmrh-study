### **상태(state)란 무엇인가?**

**상태(state)**는 컴포넌트의 렌더링과 UI의 동작을 결정하는 동적인 데이터를 말합니다. 예를 들어, 사용자가 입력한 값, 버튼 클릭 여부, API로부터 받은 응답 등이 상태에 포함됩니다. 상태는 시간이 지나면서 변화하며, 그 변화에 따라 UI가 다시 렌더링됩니다. 이 동적인 특성이 바로 상태의 핵심입니다.

#### **데이터(data)와 상태(state)의 차이점**

- **데이터(data):** 변화하지 않는 고정된 값이나 외부에서 주어진 값입니다. 예를 들어, 서버로부터 가져온 JSON 응답, 정적인 텍스트, 기본 설정 값 등이 이에 해당합니다.
- **상태(state):** 특정 시점에서 UI와 상호작용하면서 변화할 수 있는 데이터입니다. 상태는 시간에 따라 달라지고, 사용자 인터랙션, API 호출, 이벤트 등에 반응하여 변경됩니다.

### **왜 '데이터'가 아니라 '상태'로 정의하는가?**

1. **동적인 변화:** 상태는 사용자 인터랙션에 따라 변화하며 UI의 변화를 유발합니다. 이는 데이터가 가지지 않는 동적인 특성입니다.
2. **컴포넌트의 책임:** 상태는 특정 컴포넌트가 관리하는 정보로, 그 컴포넌트의 책임과 동작을 정의합니다. 즉, 상태는 컴포넌트가 '어떻게 행동할지'를 결정하는 중요한 요소입니다.
3. **렌더링과 연결:** 상태가 변하면 React는 그에 맞춰 UI를 자동으로 다시 렌더링합니다. 반면, 데이터는 이러한 자동적이고 직접적인 UI 변화와는 연결되어 있지 않습니다.
4. **UI와 상호작용:** 상태는 사용자의 행동, 컴포넌트 간의 상호작용 등과 밀접하게 연관되어 있어 UI의 현재 상태를 반영합니다.

### **전통적인 상태 관리 방식 - 중앙 집중적인 방식**

전통적인 상태 관리 방식은 애플리케이션의 전체 상태를 한 곳에서 중앙 집중적으로 관리하여, 상태의 일관성과 데이터 흐름을 유지하려는 접근을 의미합니다. 이 방식은 특히 규모가 큰 애플리케이션에서 상태 관리의 복잡성을 줄이기 위해 사용되며, 대표적으로 Redux, MobX, Context API 등이 있습니다.

#### **장점**

- **예측 가능성:** 상태가 중앙에서 관리되므로, 상태의 흐름과 변화를 예측하고 추적하기 쉽습니다.
- **일관된 데이터 흐름:** 단방향 데이터 흐름으로 인해 데이터가 어떤 경로로 이동하고 변하는지 명확하게 이해할 수 있습니다.

#### **단점**

- **복잡한 설정:** 초기 설정이 복잡하고, 보일러플레이트 코드가 많아 코드 작성이 번거롭습니다.
- **성능 문제:** 모든 상태가 중앙에서 관리되다 보니, 불필요한 리렌더링이나 성능 저하가 발생할 수 있습니다.
- **작은 프로젝트에 부담:** 단순한 애플리케이션에는 과도한 도입이 될 수 있습니다.

### **마이크로 상태 관리란?**

**마이크로 상태 관리**는 개별 컴포넌트 또는 컴포넌트 그룹 내에서 필요한 작은 상태를 관리하는 접근 방식을 의미합니다. React의 기본 훅(`useState`, `useReducer` 등)을 사용하여 필요한 부분에서만 상태를 관리하는 것을 강조합니다.

#### **마이크로 상태 관리의 필요성**

1. **폼 상태의 특수성:** 폼 상태는 입력 필드의 값, 오류, 유효성 검사 결과 등 컴포넌트 내에서만 중요한 데이터를 관리합니다. 전역 상태로 관리할 경우, 잦은 상태 변경으로 인해 불필요한 리렌더링과 성능 저하가 발생할 수 있습니다.
2. **서버 캐시 상태의 특성:** 서버 캐시 상태는 서버로부터 받아온 데이터를 클라이언트에 저장하고, 필요할 때 이를 재사용하거나 업데이트합니다. 이러한 상태는 데이터의 동기화, 유효성, 리패칭 등의 고유한 관리 전략이 필요하며, 일반적인 전역 상태와 동일하게 처리하는 것은 부적합합니다.
3. **네비게이션 상태의 특수성:** 네비게이션 상태는 브라우저의 URL, 히스토리, 현재 페이지의 위치 등과 관련된 상태로, 브라우저가 직접 관리하는 영역입니다. 이 상태를 전역 상태로 관리하면 복잡한 설정이 필요하고 브라우저의 기본 동작과 충돌할 수 있습니다.

### **리액트 훅을 이용한 마이크로 상태 관리**

React 훅을 사용하면, 컴포넌트 내부에서 상태를 관리하고, 필요에 따라 사용자 정의 훅을 만들어 상태 관리 로직을 재사용할 수 있습니다.

1. **`useState`:** 컴포넌트 내부에서 상태를 생성하고 관리할 수 있습니다. 사용자 정의 훅을 만들어 로직을 재사용할 수 있습니다.
2. **`useReducer`:** 복잡한 상태 관리가 필요한 경우, `useReducer`를 사용하여 상태 관리 로직을 캡슐화할 수 있습니다.
3. **`useEffect`:** 컴포넌트의 생명 주기와 함께 작동하는 비동기 로직을 실행할 수 있습니다.

#### **예시: 폰 번호 입력 폼**

폰 번호 입력 필드에서 사용자가 입력한 값을 실시간으로 포맷팅하여 보여주는 예시를 생각해 봅시다. 이 로직을 반복적으로 사용해야 할 경우, 사용자 정의 훅을 만드는 것이 유리합니다.

```jsx
// 사용자 정의 훅: usePhoneNumber
import { useState } from 'react';

function usePhoneNumber(initialValue = '') {
  const [phoneNumber, setPhoneNumber] = useState(initialValue);

  // 입력 값을 포맷팅하고 상태를 변경하는 함수
  const handlePhoneNumberChange = (value) => {
    value = value.replace(/[^0-9]/g, ''); // 숫자만 남기기
    if (value.length > 3 && value.length <= 6) {
      value = value.replace(/(\d{3})(\d+)/, '$1-$2'); // 중간에 하이픈 추가
    } else if (value.length > 6) {
      value = value.replace(/(\d{3})(\d{3})(\d+)/, '$1-$2-$3'); // 전체 형식화
    }
    setPhoneNumber(value);
  };

  // 상태와 포맷팅 함수를 반환
  return [phoneNumber, handlePhoneNumberChange];
}

export default usePhoneNumber;

// 컴포넌트에서 사용
import React from 'react';
import usePhoneNumber from './usePhoneNumber';

function PhoneNumberForm() {
  const [phoneNumber, setPhoneNumber] = usePhoneNumber();

  return (
    <div>
      <input
        type="text"
        value={phoneNumber}
        onChange={(e) => setPhoneNumber(e.target.value)}
        placeholder="Enter your phone number"
      />
      <button type="submit">Submit</button>
    </div>
  );
}

export default PhoneNumberForm;
```

React에서 전역 상태를 구현하는 것이 간단하지 않은 이유는 React의 철학과 구조에서 기인합니다. React는 컴포넌트 기반 아키텍처를 중심으로 설계되었으며, 각 컴포넌트가 자신의 상태를 관리하도록 권장합니다. 전역 상태 관리는 프로젝트가 커질수록 더욱 복잡해지기 때문에, 이를 효과적으로 다루기 위해서는 올바른 전략과 도구를 사용하는 것이 중요합니다.

### **전역 상태 탐구하기**

React에서 전역 상태를 관리하는 것은 단순한 작업이 아닙니다. React의 기본적인 상태 관리 방식은 `useState`와 `useReducer` 같은 훅을 사용해 개별 컴포넌트에서 상태를 관리하는 것이며, 이는 React가 권장하는 방향이기도 합니다. 컴포넌트가 전역 상태에 의존하지 않는 것이 이상적이며, 다음과 같은 이유에서 전역 상태 구현은 주의가 필요합니다:

1. **컴포넌트의 독립성 유지:** 각 컴포넌트는 독립적으로 동작하도록 설계되어야 합니다. 전역 상태에 과도하게 의존하게 되면, 컴포넌트 간의 결합도가 높아져 유지보수가 어려워질 수 있습니다.
2. **상태의 예측 가능성 감소:** 전역 상태는 전체 애플리케이션에 걸쳐 영향을 미치기 때문에, 상태 변화를 추적하고 관리하기 어려워질 수 있습니다. 이는 특히 애플리케이션이 복잡해질수록 문제가 됩니다.
3. **리렌더링 최적화의 어려움:** 전역 상태의 변화는 많은 컴포넌트에 영향을 줄 수 있으며, 불필요한 리렌더링이 발생할 수 있습니다. 이는 성능 저하로 이어질 수 있습니다.

이러한 이유로 전역 상태 관리를 구현할 때는 React의 기본 훅을 적절히 활용하여 상태 관리를 분리하거나, Context API, Redux, Zustand와 같은 전역 상태 관리 라이브러리를 사용하는 것이 일반적입니다.

### **`useState`와 베일아웃(Bailout)**

**베일아웃**은 React의 상태 업데이트 최적화 기술로, 상태가 업데이트되었지만 실제 값이 변하지 않은 경우, 리렌더링을 하지 않도록 하는 메커니즘입니다. 이는 성능 최적화에 중요한 역할을 하며, 상태 변화가 없을 때 불필요한 렌더링을 방지할 수 있습니다.

#### **`useState`에서의 베일아웃 동작 방식**

- **값으로 상태 갱신하기:**

  - `useState`에서 상태를 업데이트할 때 새로운 값이 이전 값과 같다면, React는 리렌더링을 건너뜁니다.

    ```jsx
    const Component = () => {
      const [count, setCount] = useState(0);

      return (
        <div>
          {count}
          <button onClick={() => setCount(1)}>click</button>
        </div>
      );
    };
    ```

    위 예제에서 `count`가 이미 `1`인 상태에서 버튼을 클릭해도 상태가 실제로 변경되지 않기 때문에, React는 리렌더링을 수행하지 않습니다. 이를 베일아웃이라고 합니다.

- **객체나 배열로 상태 갱신하기:**

  - 만약 객체나 배열을 상태로 관리하는 경우, 새로운 객체가 이전 객체와 참조적으로 동일하지 않다면, React는 리렌더링을 수행합니다.

    ```jsx
    const Component = () => {
      const [state, setState] = useState({ count: 0 });

      return (
        <div>
          {state.count}
          <button onClick={() => setState({ count: 1 })}>click</button>
        </div>
      );
    };
    ```

    여기서는 `{ count: 1 }`이라는 새로운 객체가 매번 생성되므로, 리렌더링이 일어나며 베일아웃이 발생하지 않습니다. 이는 객체의 참조가 변경되기 때문입니다.

- **함수형 업데이트:**

  - 상태를 함수형으로 업데이트하면, 이전 상태를 기반으로 새로운 상태를 계산할 수 있어 정확한 베일아웃이 가능합니다.

    ```jsx
    <button onClick={() => setCount((prevCount) => prevCount + 1)}>
      Increment Count
    </button>
    ```

    이 방식은 특히 비동기적으로 여러 업데이트가 이루어질 때 유용하며, 상태 관리의 예측 가능성을 높입니다.

### **`useReducer`의 지연 초기화와 상태 관리**

`useReducer`는 `useState`의 대안으로, 상태가 복잡하거나 여러 액션에 의해 변경되어야 할 때 유용하게 사용됩니다. `useReducer`는 상태와 업데이트 로직을 분리할 수 있어, 로직이 명확하고 유지보수가 용이해집니다.

#### **지연 초기화 (Lazy Initialization)**

지연 초기화는 `useReducer`에서 상태 초기화 비용이 큰 경우에 사용됩니다. 상태를 초기화하는 함수는 첫 번째 렌더링에서만 호출되며 이후에는 호출되지 않아 성능을 최적화할 수 있습니다.

- **초기화 함수 사용:**

  ```jsx
  const init = (initialCount) => {
    return { count: initialCount, text: "hi" };
  };

  const reducer = (state, action) => {
    switch (action.type) {
      case "INCREMENT":
        return { ...state, count: state.count + 1 };
      case "SET_TEXT":
        return { ...state, text: action.text || state.text };
      default:
        return state;
    }
  };

  function Counter() {
    const [state, dispatch] = useReducer(reducer, 0, init);

    return (
      <div>
        Count: {state.count}
        <button onClick={() => dispatch({ type: "INCREMENT" })}>
          Increment
        </button>
      </div>
    );
  }
  ```

  위 코드에서 `init` 함수는 `useReducer`의 세 번째 인자로 전달되어, 초기 상태를 설정하는 데 사용됩니다. 이 방법은 초기화가 복잡하거나 성능이 중요한 상황에서 매우 유용합니다.

#### **`useReducer`와 `useState`의 비교**

- **`useState`:** 간단한 상태 관리에 적합하며, 사용하기 쉽고 직관적입니다. 내부적으로는 `useReducer`를 기반으로 구현되어 있습니다.
- **`useReducer`:** 복잡한 상태 관리에 적합하며, 상태 업데이트 로직을 컴포넌트 외부로 분리할 수 있습니다. 이를 통해 코드의 가독성과 유지보수성을 높일 수 있습니다.

### **`useState`와 `useReducer`의 내부 구현 비교**

React는 `useState`를 내부적으로 `useReducer`를 사용하여 구현합니다. `useReducer`의 동작을 간단히 살펴보면, 상태 업데이트 로직이 함수형으로 정의되고, 액션에 따라 상태를 업데이트합니다.

```jsx
const useState = (initialState) => {
  const [state, dispatch] = useReducer((prevState, action) => {
    return typeof action === "function" ? action(prevState) : action;
  }, initialState);

  return [state, dispatch];
};
```

이러한 구조는 `useState`가 `useReducer`의 단순화된 버전임을 보여줍니다. `useState`는 주로 단순한 상태를 다루기 위한 것이고, `useReducer`는 복잡한 상태 업데이트 로직을 처리하기 위해 설계되었습니다.

### **결론**

- **전역 상태 관리의 어려움:** React는 컴포넌트가 독립적으로 상태를 관리하도록 설계되어 있으며, 전역 상태는 적절한 도구와 전략이 없으면 복잡성을 초래할 수 있습니다.
- **`useState`와 베일아웃:** 값이 변경되지 않을 때 리렌더링을 방지하는 최적화로, 성능 개선에 중요한 역할을 합니다.
- **`useReducer`의 지연 초기화:** 복잡한 상태 관리에서 성능을 최적화할 수 있는 방법이며, 상태 초기화 비용을 줄이는 데 유용합니다.
- **상태 관리의 선택:** 애플리케이션의 복잡도에 따라 `useState`와 `useReducer`를 적절히 선택하고, 필요할 경우 전역 상태 관리 도구를 도입하여 상태 관리의 일관성과 성능을 유지하는 것이 중요합니다.

이러한 개념들을 통해 React에서 상태 관리의 본질과 각각의 훅의 역할을 이해하고, 프로젝트에 맞는 최적의 상태 관리 방법을 선택하는 데 도움이 될 것입니다.
