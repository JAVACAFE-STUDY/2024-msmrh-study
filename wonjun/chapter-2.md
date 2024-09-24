<h1>지역 상태와 <br> 전역 상태 사용하기</h1>

- React Component는 Tree 구조로 구성된다.

## 자바스크립트 함수의 작동 원리

- 순수 함수 : Argument에만 의존하며, 동일한 Argument를 받으면 언제나 동일한 값을 반환한다.

  ```tsx
  const plusOne = (n: number) => n + 1;

  console.log(plusOne(1)); // => 2
  console.log(plusOne(1)); // => 2
  console.log(plusOne(1)); // => 2
  ```

- 비순수 함수 : 외부 요인이 바뀔 경우 동일한 Argument를 받아도 다른 값을 반환할 수 있는 함수.

  ```tsx
  const plusNow = (n: number) => n + +new Date();

  // 현재 timestamp = 0
  console.log(plusOne(1)); // => 1
  // 현재 timestamp = 100
  console.log(plusOne(1)); // => 101
  // 현재 timestamp = 200
  console.log(plusOne(1)); // => 201
  ```

- singleton : 특정 클래스의 인스턴스가 오직 하나만 생성되도록 보장하는 패턴

  ```tsx
  const Singleton = (function () {
    let instance;

    function createInstance() {
      const object = new Object("I am the instance");
      return object;
    }

    return {
      getInstance: function () {
        if (!instance) {
          instance = createInstance();
        }
        return instance;
      },
    };
  })();

  const instance1 = Singleton.getInstance(); // 여기서 instance가 생성된다
  const instance2 = Singleton.getInstance(); // 위에서 생성된 instance를 가져온다

  console.log(instance1 === instance2); // true
  ```

- contained : 상태가 Component 외부에서 사용될 때, 해당 상태가 다른 컴포넌트에 영향을 미치지 않는 특성
- 이 함수는 순수할까?
  ```tsx
  function addBase({ number }) {
    const [base, setBase] = useState(1);
    return <div>{number + base}</div>;
  }
  ```
  - 엄밀히 말하자면 순수하지 않다.
  - 하지만 useState는 변경되지 않는 한 같은 base를 반환하기에 멱등성을 가진다.
  - `setBase` 는 함수 선언 범위 내에서만 사용할 수 있기 때문에 억제되었고, 바깥에서 변경하는것이 불가능하다. → 지역 상태를 사용한다 == 지역성을 보장한다.

## 지역 상태를 효과적으로 사용하는 방법

### 상태 끌어올리기

- 2개의 카운트의 상태를 공유하려면? → 컴포넌트 2개를 합친 부모 컴포넌트에서 상태를 관리한다.

  ```tsx
  function Root() {
    const [count, setCount] = useState(0);

    return (
      <>
        <Component1 count={count} setCount={setCount} />
        <Component2 count={count} setCount={setCount} />
      </>
    );
  }

  function Component1({
    count,
    setCount,
  }: {
    count: number;
    setCount: (n: number) => void;
  }) {
    return (
      <>
        {count}
        <button onClick={setCount(count + 1)}>add</button>
      </>
    );
  }

  function Component2({
    count,
    setCount,
  }: {
    count: number;
    setCount: (n: number) => void;
  }) {
    return (
      <>
        {count}
        <button onClick={setCount(count + 1)}>add</button>
      </>
    );
  }
  ```

### 하지만 퍼포먼스 이쓔가 있다.

- count 가 변경되면 Component1 / Component2 가 모두 리렌더링 된다.
- 만약 여기서 다른 컴포넌트가 추가된다면 그 컴포넌트까지 같이 리렌더링 된다.

### 이를 해결하려면…

- 아효 모르겠다 React Memo 써도 되긴 하는데 요기서는 additionalInfo를 따로 보내면 된다고 하네…

## 전역 상태 사용하기

- 전역 상태 : 상태가 하나의 컴포넌트에 속하지 않고 여러 컴포넌트에서 사용될 수 있는 상태

### 언제 써야 하는 가

1. prop을 전달하기에 컴포넌트가 너무 멀리 있을 때
   - 물론 prop을 계속 상위로 올리고 올려서 쓸수도 있지만
   - 유지보수성이 떨어진다. → prop drilling
2. 이미 React 외부에 상태가 있을 때
   - React 외부에서 가져온 값이 있을 경우 (로컬스토리지 라던가…)
