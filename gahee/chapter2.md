# 지역 상태와 전역 상태 사용하기

끌어올리기 패턴(lifting-up) 패턴 : 지역 상태를 비롯해 컴포넌트 트리에서 정보를 상위 컴포넌트로 전달하는 기법

## 지역 상태를 다룰 때

자바스크립트의 함수
끌어올리기 패턴(lifting-up) 패턴 : 지역 상태를 비롯해 컴포넌트 트리에서 정보를 상위 컴포넌트로 전달하는 기법

1. 순수 함수
   : 인수에만 의존하며 동일한 결과를 반환하는 함수

```js
const add = (n) => n + 1;
```

2. 비순수 함수
   : 상태는 인수의 값을 말하며 상태에 의존하는 함수는 순수하지 않게 된다.
   : 리액트 컴포넌트내에서 사용하는 함수가 상태를 사용하게 되면 더이상 순수하게 되지 않지만, 다른 컴포넌트에는 영향을 미치지 않으므로 이를 **_"억제됨"_** 이라 부른다.

```js
let base = 1;

const addBase = (n) => n + base;
```

위와 같이 **_싱글톤_** 으로 사용하는 경우 코드의 재 사용성이 떨어지기 때문에 **_컨테이너 객체_** 를 만드는 것이 더 모듈화된 접근 방식이다.

```js
const createContainer = () => {
  let base = 1;
  const addBase = (n) => n + base;
  conts changeBase = (b) => { base = b; }
   return {addBase, changeBase}
}

const {addBasem changeBase } = createContainer();
```

각 컨테이너를 다른 컨테이너에 영향을 주지 않고 사용할 수 있다 (재사용성)

## 리액트 컴포넌트와 props

### 지역 상태에 대한 useState 이해하기

```jsx
const AddBase = ({ number }) => {
  const [base, changeBase] = useState(1);
  return <div>{number + base}</div>;
};
```

위 함수는 인수에 포함되지 않은 base에 의존하기 때문에 순수하지 않다.

### 지역 상태의 한계

만약 위 AddBase 컴포넌트에서 base를 외부에서 변경 하고 싶다면, 지역성을 사용하는 것이 적절하지 않다.
이럴땐 전역 변수가 필요 하다.

! 전역 상태를 과하게 사용하면, 컴포넌트의 동작을 예측하기 어렵기 때문에 과하게 사용해서는 안된다.

## 지역 상태를 효과적으로 사용하는 방법

### 상태 끌어올리기

```jsx
const Component1 = () => {
  const [count, setCount] = useState(0);
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>카운트 증가</button>
    </div>
  );
};

const Component2 = () => {
  const [count, setCount] = useState(0);
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>카운트 증가</button>
    </div>
  );
};
```

위 두 컴포넌트는 서로 영향을 끼치지 못한다. 만약 두 컴포넌트의 상태를 공유하고 하나의 공유된 카운터로 작동하게 하려면 아래와 같이 사용해야 한다.

```jsx
const Component1 = ({ count, setCount }) => {
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>카운트 증가</button>
    </div>
  );
};

const Component2 = ({ count, setCount }) => {
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>카운트 증가</button>
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

count의 상태는 parent에서 단 한번만 정의하기 때문에 자식 컴포넌트에 공유 된다.

! 단, 이 패턴은 지역 상태를 사용하는 대부분의 상황에서 작동하지만 성능 문제가 있을 수 있다.
이 상태를 상위 컴포넌트로 전달할 경우 Parent는 모든 자식 Component를 리렌더링 할 수 있다. (일부 상황에서 문제가 될수 있음)

### 내용 끌어올리기

상위 컴포넌트의 상태에 의존하지 않기

```jsx
const AdditionalInfo = () => {
  return <div>Something information</div>;
};
const Component1 = ({ count, setCount }) => {
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>카운트 증가</button>
    </div>
  );
};

const Component2 = ({ count, setCount }) => {
  return (
    <div>
      {count}
      <AdditionalInfo />
      <button onClick={() => setCount((c) => c + 1)}>카운트 증가</button>
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

위와 같다면 영향을 받지 않는 AdditionalInfo컴포넌트도 다시 렌더링 된다.
이런걸 막기 위해 상위 컴포넌트로 끌어올릴 수 있다.

```jsx
const AdditionalInfo = () => {
  return <div>Something information</div>;
};
const Component1 = ({ count, setCount }) => {
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>카운트 증가</button>
    </div>
  );
};

const Component2 = ({ count, setCount, additionalInfo }) => {
  return (
    <div>
      {count}
      {additionalInfo}
      <button onClick={() => setCount((c) => c + 1)}>카운트 증가</button>
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
  return;
  <Parent additionalInfo={<AdditionalInfo />}></Parent>;
};
```

또 다른 방법으로는

```jsx
const AdditionalInfo = () => {
  return <div>Something information</div>;
};
const Component1 = ({ count, setCount }) => {
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>카운트 증가</button>
    </div>
  );
};

const Component2 = ({ count, setCount, children }) => {
  return (
    <div>
      {count}
      {additionalInfo}
      <button onClick={() => setCount((c) => c + 1)}>카운트 증가</button>
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
  return;
  <Parent additionalInfo={<AdditionalInfo />}></Parent>;
};
```

코딩 스타일에 다르면 원하는 방식을 사용 하자

## 전역 상태 사용하기

### 전역 상태란?

이 책에서는 지역 상태가 아님을 의미 함. (개념적으로 속한 곳이 어디냐에 따라 그 의미가 달라질 수 있기 때문)

### 언제 전역 상태를 사용할까?

- prop을 전달하는 것이 적절하지 않을때
- 이미 리액트 외부에 상태가 있을떄

#### prop을 전달하는 것이 적절하지 않을때

- 컴포넌트 트리에서 서로 멀리 떨어져 있는 두 컴포넌트 사이에는 prop을 전달하는 것이 적절하지 않다.
- props을 전달한다면, props Driliing이 발생하며 상태와 관련 없는 컴포넌트까지 리렌더링이 일어난다.

```jsx
const Component1 = () => {
  const [count, setCount] = useGloblCountState();
  return (
    <div>
      {count}{" "}
      <button onClick={() => setCount((c) => c + 1)}>카운트 증가</button>
    </div>
  );
};
const Parent = () => {
  return <Component1 />;
};

const GrandParent = () => {
  return <Parent />;
};

const Root = () => {
  return (
    <>
      <GrandParent />
    </>
  );
};
```

#### 이미 리액트 외부에 상태가 있을때,

리액트 없이 획득한 사용자 인증 정보가 있을때는 전역 상태가 외부에 존재해야 한다.

```jsx
const globalState = {
  authInfo: { name: "React" },
};

const Component1 = () => {
  const { authInfo } = useGlobalState();

  return <div>{authInfo.name}</div>;
};
```
