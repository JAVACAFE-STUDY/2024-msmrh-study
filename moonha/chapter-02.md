# Chapter-02. 지역  상태와 전역 상태 사용하기

## 언제 지역 상태를 사용할까?

- 순수 함수
    - 오직 인수에만 의존하며 동일한 인수를 받은 경우 동일한 값을 반환
- 상태
    - 인수 외부의 값을 말하며 상태에 의존하는 함수는 순수하지 않게 됨
- `억제됨(contained)`
    - 컴포넌트 내에서 상태를 사용할 경우 외부의 값에 의존하기 때문에 순수하지 않게 되지만, 상태가 컴포넌트 내에서만 사용된다면 다른 컴포넌트에 영향을 미치지 않음

### 함수와 인수

- 전역 변수에 의존하는 함수
    
    ```jsx
    let base = 1;
    
    const addBase = (n) => n + base;
    ```
    
    - 장점
        - 외부에서 함수 작동 방식을 변경할 수 있음
    - 단점
        - 외부 변수에 의존한다는 사실을 모르고 다른 곳에서 사용될 수 있음
    - 싱글턴(기술적으로 자바스크립트의 전역 변수는 싱글턴이라고 할 수 있음. 한 번 생성된 전역 변수는 window 객체에 등록되며, 이후 참조할 경우 window 객체에 등록된 값을 반환함)인 경우 코드 재사용성이 떨어지기 때문에 선호되는 패턴은 아님
- 컨테이너 객체
    
    ```jsx
    const container = () => {
    	let base = 1;
    	const addBase = (n) => n + base;
    	const changeBase = (b) => { base = b };
    	return { addBase, changeBase };
    };
    
    const { addBase, changeBase } = createContainer();
    ```
    
    - 더 모듈화된 접근 방식
    - 필요한 만큼 컨테이너를 만들 수 있음
    - 컨테이너는 격리돼 있으므로 재사용하기 쉬움, 다른 컨테이너에 영향을 주지 않고 사용 가능

### 리액트 컴포넌트와 props

- 리액트 컴포넌트는 props라는 인수를 받아서 JSX 요소를 반환하는 함수

### 지역 상태에 대한 useState 이해하기

- 인수에 포함되지 않은 값에 의존하기 때문에 엄밀히 말해 순수하지 않음
- useState의 동작을 추측하자면 변경되지 않는 한 동일한 값을 반환하므로 멱등성을 지님
- 함수 선언 범위 내에서만 값을 변경할 수 있기 때문에 억제됐다고 할 수 있음
- 컴포넌트는 억제돼 있고 컴포넌트 외부의 그 어떤 것에도 영향을 미치지 않기 때문에 지역성을 보장함

### 지역 상태의 한계

- 함수 컴포넌트 외부에서 상태를 변경해야 한다면 전역 상태가 필요함
- 전역 변수
    - 함수 외부에서 자바스크립트 함수의 동작을 제어할 때 유용하게 사용 가능
- 전역 상태
    - 장점
        - 컴포넌트 외부에서 리액트 컴포넌트의 동작을 제어할 때 유용하게 사용할 수 있음
    - 단점
        - 컴포넌트 동작을 예측하기 어려움

## 지역 상태를 효과적으로 사용하는 방법

### 상태 끌어올리기(Lifting State Up)

- 두 컴포넌트의 상태를 공유하고 하나의 공유된 상태로 작동하게 만들고 싶다면 부모 컴포넌트를 만들고 상태를 상위 컴포넌트로 전달
- 성능 문제
    - 모든 자식 컴포넌트를 포함해 하위 트리 전체를 리렌더링하기 때문에 성능 문제가 발생할 수 있음

### 내용 끌어올리기(Lifting Content Up)

```jsx
const AdditionalInfo () => {}

const Component1 = ({ count, setCount }) => {
  return (
	  <div>
	    {count}
	    ...
	    <AdditionalInfo />
	  </div>
	);
};

const Parent = () => {
	const [count, setCount] = useState(0);
	return (
		<>
			<Component1 count={count} setCount={setCount} />
			...
		</>
	);
};
```

- 불필요한 리렌더링을 방지하기 위해서 JSX 요소를 상위 컴포넌트로 끌어올리기
    - GrandParent 만들기
        
        ```jsx
        const AdditionalInfo () => {}
        
        const Component1 = ({ count, setCount, additionalInfo }) => {
          return (
        	  <div>
        	    {count}
        	    ...
        	    {additionalInfo}
        	  </div>
        	);
        };
        
        const Parent = ({ additionalInfo }) => {
        	const [count, setCount] = useState(0);
        	return (
        		<>
        			<Component1 count={count} setCount={setCount} additionalInfo={additionalInfo} />
        			...
        		</>
        	);
        };
        
        const GrandParent = () => {
        	return <Parent additionalInfo={<AdditionalInfo />} />;
        };
        ```
        
        - count가 변경되어도 AdditionalInfo 컴포넌트는 리렌더링되지 않음
- Children prop를 사용
    
    ```jsx
    const AdditionalInfo () => {}
    
    const Component1 = ({ count, setCount, children }) => {
      return (
    	  <div>
    	    {count}
    	    ...
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
    			...
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
    
    - children
        - JSX 형식으로 중첩된 자식 요소를 표현하는 특별한 prop 이름

## 전역 상태 사용하기

### 전역 상태란?

- 지역 상태가 아님을 의미
    - 지역 상태
        - 하나의 컴포넌트에 속하고 컴포넌트에 의해 캡슐화된 상태
- 여러 컴포넌트에서 사용할 수 있다면 전역 상태

### 언제 전역 상태를 사용할까?

**prop을 전달하는 것이 적절하지 않을 때**

- 컴포넌트 트리에서 서로 멀리 떨어져 있는 두 컴포넌트 간에 상태를 공유
- 상태가 변경되면 중간 컴포넌트가 리렌더링되어 성능에도 영향을 미칠 수 있음

**이미 리액트 외부에 상태가 있을 때**

- 사용자 인증 정보