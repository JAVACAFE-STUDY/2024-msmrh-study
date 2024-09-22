# 자바스크립트 함수의 작동법

자바스크립트 함수는 크게 순수 함수와 비순수 함수로 나눌 수 있다.

리액트 컴포넌트는 자바스크립트 함수이기 때문에 순수할 수 있지만, 컴포넌트 내에서 상태를 사용할 경우 순수하지 않게 됨

그러나 상태가 컴포넌트 내에서만 사용된다면 다른 컴포넌트에 영향을 미치지 않는다. ⇒ 억제됨(contained) 특성

## 순수 함수

오직 인수에만 의존하여 같은 인수에 대해 항상 같은 값을 변환하는 함수

```jsx
const addOne = (n) => n + 1;
```

## 비순수 함수

인수 외부의 값(상태)에 의존하는 함수

```jsx
let base = 1;

const addBase = (n) => n + base;
```

외부에서 함수 작동 방식을 변경할 수 있다는 장점이 있으나, 해당 함수가 외부 변수에 의존한다는 사실을 모르고 다른 곳에 사용될 수 있다는 단점이 있음

위와 같이 base가 싱글턴인 경우보다는, 재사용성을 높일 수 있게 컨테이너 객체를 만드는 것이 더 모듈화된 접근 방식이다.

```jsx
const createContainer = () => {
  let base = 1;
  const addBase = (n) => n + base;
  const changeBase = (b) => {
    base = b;
  };
  return { addBase, changeBase };
};

const { addBase, changeBase } = createContainer();
```

이 경우 각 컨테이너를 다른 컨테이너에 영향을 주지 않고 사용할 수 있음

이때 컨테이너의 addBase는 순수한 함수는 아니지만, changeBase를 통해 base를 변경하지 않는 이상 항상 동일한 결과를 반환하므로 **멱등성(idempotent)**이 있다.

<aside>
💡 멱등성 : 연산을 여러 번 적용하더라도 결과가 달라지지 않는 성질, 연산을 여러 번 반복하여도 한 번만 수행된 것과 같은 성질
</aside>

# 리액트 컴포넌트와 props

리액트는 상태를 사용자 인터페이스(UI)로 변환하는 함수

즉 리액트 컴포넌트는 자바스크립트 함수이며 그것의 인수를 props라고 한다.

```jsx
const AddOne = ({ number }) => {
  return <div>{number + 1}</div>;
};
```

이 컴포넌트는 이전 절의 addOne 함수와 똑같이 작동하는 순수 함수다.

차이점은 인수가 props 객체이고 반환값이 JSX 형식일 뿐

# 지역 상태에 대한 useState 이해하기

```jsx
const AddBase = ({ number }) => {
  const [base, changeBase] = useState(1);
  return <div>{number + base}</div>;
};
```

이 함수는 인수에 포함되지 않은 base에 의존하기 때문에 엄밀히 말해 순수하지는 않지만, 변경되지 않는 한 base를 반환하므로 멱등성을 지닌다.

AddBase 함수는 changeBase를 함수 선언 범위 내에서만 사용할 수 있기 때문에 억제된 상태이다.

⇒ useState를 이렇게 사용하는 것은 **지역 상태를 사용하는 것**에 해당하며, 컴포넌트는 외부의 어떤 것에도 영향을 미치지 않기 때문에 지역성을 보장한다.

# 지역 상태의 한계

함수 컴포넌트 외부에서 상태를 변경해야 한다면 전역 상태가 필요하다.

전역 상태는 컴포넌트 외부에서 리액트 컴포넌트의 동작을 제어할 때 유용하게 사용할 수 있지만 컴포넌트 동작을 예측하기 어렵다는 장단점이 있다.

⇒ 지역 상태를 기본으로 사용하고, 전역 상태는 보조 수단으로 사용하는 것이 좋다.

# 지역 상태를 효과적으로 사용하는 방법

지역 상태를 효과적으로 사용하는 몇 가지 패턴이 있다.

- 상위 컴포넌트 트리에서 상태를 정의하는 상태 끌어올리기(lifting state up) 패턴
- 상위 컴포넌트 트리에서 내용을 정의하는 내용 끌어올리기(lifting content up) 패턴

## 상태 끌어올리기(Lifting State Up)

두 컴포넌트의 상태를 공유하고자 할 때, 부모 컴포넌트를 만들고 상태를 상위 컴포넌트로 전달하는 패턴

```jsx
const Component1 = ({ count, setCount }) => {
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>Increment Count</button>
    </div>
  );
};

const Component2 = ({ count, setCount }) => {
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>Increment Count</button>
    </div>
  );
};

const Parent = () => {
  const [count, setCount] = useState(0);
  return (
    <>
      <Component1 count={count} setCount={setCount} />
      <Component2 count={count} setCount={setCount} />
    </>
  );
};
```

count는 여전히 컴포넌트 내에 지역 상태로 존재하며, 자식 컴포넌트는 부모 컴포넌트의 상태를 이용할 수 있다.

이 패턴은 지역 상태를 사용하는 대부분의 상황에서 작동하지만, Parent는 모든 자식 컴포넌트를 포함해 하위 트리 전체를 리렌더링하므로 일부 상황에서 성능 문제를 일으킬 수 있다는 단점이 있다.

## 내용 끌어올리기(Lifting Content Up)

복잡한 컴포넌트 트리라면 상위 컴포넌트의 상태에 의존하지 않는 컴포넌트가 있을 수 있는데, 이 경우 상태 끌어올리기 패턴을 사용하면 해당 상태가 변경될 때마다 불필요한 리렌더링이 일어나므로 성능에 영향을 줄 수 있다.

이를 방지하기 위해 JSX 요소를 상위 컴포넌트로 끌어올려, 상태 변경에 영향을 받지 않는 GrandParent 컴포넌트에 JSX를 배치할 수 있다.

```jsx
const AdditionalInfo = () => {
  return <p>Some information</p>;
};

const Component1 = ({ count, setCount, additionalInfo }) => {
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>Increment Count</button>
      {additionalInfo}
    </div>
  );
};

const Parent = ({ additionalInfo }) => {
  const [count, setCount] = useState(0);
  return (
    <>
      <Component1
        count={count}
        setCount={setCount}
        additionalInfo={additionalInfo}
      />
      <Component2 count={count} setCount={setCount} />
    </>
  );
};

const GrandParent = () => {
  return <Parent additionalInfo={<AdditionalInfo />} />;
};
```

위의 경우 AdditionalInfo 컴포넌트는 count가 변경될 때 리렌더링되지 않는다.

이는 성능뿐만 아니라 컴포넌트 트리 구조를 구성할 때도 고려할 만한 기법이다.

이 기법의 변형으로 children prop을 사용할 수 있다. children은 JSX 형식으로 중첩된 자식 요소를 표현하는 특별한 prop 이름이다.

```jsx
const Component1 = ({ count, setCount, children }) => {
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>Increment Count</button>
      {children}
    </div>
  );
};

const Parent = ({ children }) => {
  const [count, setCount] = useState(0);
  return (
    <>
      <Component1 count={count} setCount={setCount}>
        {children}
      </Component1>
      <Component2 count={count} setCount={setCount} />
    </>
  );
};

const GrandParent = () => {
  return (
    <Parent>
      <AdditionalInfo />
    </Parent>
  );
};
```

개발자들은 각자 선호하는 접근법을 선택하면 된다.

# 전역 상태 사용하기

## 전역 상태란?

개념적으로 하나의 컴포넌트에 속하고 컴포넌트에 의해 캡슐화된 상태를 지역 상태라고 한다.

따라서 상태가 하나의 컴포넌트에만 속하지 않고 여러 컴포넌트에서 사용할 수 있다면 전역 상태라고 한다.

지역 상태와 전역 상태는 명확하게 나누기 힘들다.

대부분의 경우 상태가 개념적으로 속한 곳이 어디인지 생각해 보면 지역 상태인지 전역 상태인지를 파악할 수 있다.

ex) 모든 컴포넌트가 의존하는 애플리케이션 차원의 지역 상태가 있을 경우, 이는 전역 상태이다.

전역 상태는 메모리에 하나의 값으로만 존재하는 싱글턴인 경우와 아닌 경우(공유 상태)가 있다. 공유 상태인 전역 상태는 컴포넌트 트리의 다른 부분에 대해 여러 값을 가질 수 있다.

```jsx
const createContainer = () => {
  let base = 1;
  const addBase = (n) => n + base;
  const changeBase = (b) => {
    base = b;
  };
  return { addBase, changeBase };
};

const container1 = createContainer();
const container2 = createContainer();

container1.changeBase(10);

console.log(container1.addBase(2)); // shows "12"
console.log(container2.addBase(2)); // shows "3"
```

위 예제에서 base는 각 컨테이너에 격리돼 있으므로 컨테이너마다 다른 값을 가질 수 있다.

# 언제 전역 상태를 사용할까?

리액트에서는 다음과 같은 두 가지 상황에서 전역 상태를 사용한다.

1. **prop을 전달하는 것이 적절하지 않을 때**

컴포넌트 트리에서 서로 멀리 떨어져 있는 두 컴포넌트 간에 상태를 공유해야 할 경우, 공통 컴포넌트에 상태를 만든 다음 두 컴포넌트까지 전달하는 방법은 번거롭고 좋지 못한 개발자 경험을 준다. 또한, 상태가 변경되면 중간 컴포넌트가 리렌더링되므로 성능에도 영향을 미친다.

이 경우 전역 상태를 사용하면 중간 컴포넌트가 상태를 전달하지 않아도 된다.

2. **이미 리액트 외부에 상태가 있을 때**

어떤 경우에는 이미 리액트 외부에 전역 상태를 가지고 있다. ex) 애플리케이션에서 리액트 없이 획득한 사용자 인증 정보

이 경우 전역 상태는 리액트 외부에 존재해야 하며, 인증 정보는 전역 상태에 저장될 수 있다.

그러한 예제를 나타내는 코드는 다음과 같다.

```jsx
const globalState = {
  authInfo: { name: "React" },
};

const Component1 = () => {
  // useGlobalState is a pseudo hook
  const { authInfo } = useGlobalState();
  return <div>{authInfo.name}</div>;
};
```

두 번째 경우가 잘 안 와닿아서 Claude를 통해 예제 코드를 받아왔다. 브라우저의 localStorage를 사용하여 사용자 인증 정보를 저장하고 관리하는 경우를 예시로 들어 줬다.

```jsx
// authService.js
// 이 파일은 리액트 컴포넌트가 아닌 일반 JavaScript 모듈입니다.

// localStorage에서 토큰을 가져오거나 저장하는 함수들
export const getToken = () => localStorage.getItem("authToken");
export const setToken = (token) => localStorage.setItem("authToken", token);
export const removeToken = () => localStorage.removeItem("authToken");

// 현재 인증 상태를 확인하는 함수
export const isAuthenticated = () => !!getToken();

// 로그인 함수 (API 호출을 가정)
export const login = async (username, password) => {
  // .. API 호출
  if (data.token) {
    setToken(data.token);
    return true;
  }
  return false;
};

// 로그아웃 함수
export const logout = () => {
  removeToken();
};

// App.js
import React from "react";
import { BrowserRouter as Router, Route, Redirect } from "react-router-dom";
import { isAuthenticated } from "./authService";

const PrivateRoute = ({ component: Component, ...rest }) => (
  <Route
    {...rest}
    render={(props) =>
      isAuthenticated() ? <Component {...props} /> : <Redirect to="/login" />
    }
  />
);
```
