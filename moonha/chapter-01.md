# Chapter-01. 리액트 훅을 이용한 마이크로 상태 관리

## 마이크로 상태 관리

- 상태
    - UI를 나타내는 모든 데이터
    - 시간이 지남에 따라 변할 수 있음
    - 리액트는 상태와 함께 렌더링할 컴포넌트를 처리함
- 훅이 나오기 전
    - 중앙 집중형 상태 관리 라이브러리가 일반적이었음
- 훅이 등장
    - 폼, 서버 캐시, 네비게이션 등 특정 목적에 따라 각각의 해결책을 제공할 수 있게 됨
- 마이크로 상태 관리
    - 목적 지향적인 방법으로 처리할 수 없는 범용적인 상태 관리도 필요
    - 가벼워야 하며, 요구사항에 따라 적절한 방법을 선택할 수 있어야 함
    - 상태 읽기, 상태 갱신, 상태 기반 렌더링의 기본적인 기능이 필요함
    - 리렌더링 최적화, 다른 시스템과의 상호 작용, 비동기 지원, 파생 상태, 간단한 문법 등 추가적인 기능이 필요할 수도 있음

## 리액트 훅 사용하기

- `useState`
    - 지역 상태를 생성하는 기본적인 함수
    - 로직을 캡슐화하고 재사용 가능
- `useReducer`
    - 마찬가지로 지역 상태 생성
    - useState를 대체하는 용도로 사용
- `useEffect`
    - 리액트 렌더링 프로세스 바깥에서 로직을 실행할 수 있음
    - 전역 상태를 다루기 위한 상태 관리 라이브러리를 개발할 때 필요
        - 리액트 컴포넌트 생명 주기와 함께 작동하는 기능을 구현할 수 있기 때문
- 리액트 훅은 UI 컴포넌트에서 로직을 추출할 수 있음
    - 렌더링 부분을 제외하고 useCounter 라는 훅으로 로직을 추출 가능
        - 컴포넌트를 수정하지 않고도 기능을 추가할 수 있음

### 데이터 불러오기를 위한 서스펜스와 동시성 렌더링

- 서스펜스
    - 비동기 처리에 대한 걱정 없이 컴포넌트를 코딩할 수 있는 방법
- 동시성 렌더링
    - 렌더링 프로세스를 청크라는 단위로 분할해서 중앙 처리 장치가 장시간 차단되는 것을 방지하는 방법
- 리액트 훅 사용시 주의 사항
    - 기존 `state` 객체나 `ref` 객체를 직접 변경해서는 안 됨
    - 리액트 훅 함수와 컴포넌트는 여러 번 호출될 수 있으므로, 여러 번 호출되더라도 일관되게 동작할 수 있게 충분히 ‘순수’해야 함

## 전역 상태 탐구하기

- 지역 상태
    - `useState`와 같이 컴포넌트 트리 내에서 사용되는 상태
- 전역 상태
    - 애플리케이션 내 서로 멀리 떨어져 있는 여러 컴포넌트에서 사용하는 상태
    - 싱글턴일 필요가 없다는 점을 명확히 하기 위해 ‘공유 상태’라 부르기도 함
- 리액트는 컴포넌트 모델에 기반함
    - 컴포넌트 모델에서는 ‘지역성’이 중요하며 서로 격리돼야 재사용이 가능함

## useState 사용하기

### 값으로 상태 갱신하기

- 베일아웃
    - 리렌더링을 발생시키지 않는 것

```jsx
const [count, setCount] = useState(0)

() => setCount(1) // 베일아웃 되어 리렌더링이 되지 않음
() => setCount(count + 1) // 빠르게 클릭해도 한 번만 증가
```

### 함수로 상태 갱신하기

```jsx
const [count, setCount] = useState(0)

() => setCount(c => c + 1) // 클릭한 횟수만큼 값이 갱신됨
```

### 지연 초기화

- 첫 번째 렌더링에서만 평가되는 초기화 함수를 받을 수 있음
    
    ```jsx
    const init = () => 0;
    const [count, setCount] = useState(init);
    ```
    
    - useState가 호출되기 전까지 init 함수는 평가되지 않고 지연 평가됨

## useReducer 사용하기

- 장점
    - 훅 외부에서 리듀서 함수를 정의하면 코드를 분리 가능
    - 순수함수이기 때문에 테스트에 용이함

### 베일아웃

```jsx
const reducer = (state, action) => {
  switch (action.type) {
	  ....
	  case 'SET_TEXT': 
	    if (!action.text) {
	      // 베일아웃
	      return state;
	    }
  
  }
}
```

### 원시 값

- 객체가 아닌 원시값에 대해서도 작동함
    
    ```jsx
    const reducer = (count, delta) => {
    	if (delta < 0) {
    		throw new Error( ~
    	}
    	if (delta > 10) {
    		return count
    	}
    	if (delta < 100) {
    		return count + delta + 10
    	}
    	return count + delta
    }
    ```
    

### 지연 초기화(init)

- 세 번째 매개변수를 통해 지연 초기화 가능
    
    ```jsx
    const init = (count) => ({ count, text: 'hi' });
    
    const reducer = ...
    
    const [state, dispatch] = useReducer(reducer, 0, init)
    ```
    
    - `init` 함수는 컴포넌트가 마운트될 때 한 번만 호출되므로 무거운 연산을 포함 가능
    - `useState`와 달리 `useReducer`의 두 번째 인수인 `initialArg`를 받음

## useState와 useReducer의 유사점과 차이점

## useReducer를 이용한 useState 구현

- `useState`는 `useReducer`로 구현되어 있음
    
    ```jsx
    const useState = (initialState) => {
    	const [state, dispatch] = useReducer(
    		(prev, action) => 
    			typeof action === 'function' ? action(prev) : action, initialState	
    	);
    	return [state, dispatch];		
    }
    ```
    
    ```jsx
    const reducer = (prev, action) => 
    	typeof action === 'function' ? action(prev) : action;
    
    const useState = initialState => 
    	useReducer(reducer, initialState);
    ```
    

### useState를 이용한 useReducer 구현

- 반대로 `useReducer`를 `useState`로 거의 구현 가능
    
    ```jsx
    const useReducer = (reducer, initialState) => {
    	const [state, setState] = useState(initialState);
    	const dispatch = (action) => 
    		setState(prev => reducer(prev, action));
    	return [state, dispatch];	
    }
    ```
    
- 지연 초기화, `useCallback`을 사용해 안정적인 디스패치 함수 구현
    
    ```jsx
    const useReducer = (reducer, initialArg, init) => {
    	const [state, setState] = useState(
    		init ? () => init(initialArg) : initialArg,
    	);
    	const dispatch = useCallback(
    		(action) => setState(prev => reducer(prev, action)),
    		[reducer]
    	);
    	return [state, dispatch];
    }
    ```
    

### 초기화 함수 사용하기

- 차이점
    - `useReducer`만 `reducer`와 `init`을 훅이나 컴포넌트 외부에 정의할 수 있음
        - withUseReducer
            
            ```jsx
            const init = ~
            const reducer = ~
            
            const Component = ({initialCount}) => {
            	const [state, dispatch] = useReducer(reducer, initialCount, init);
            }
            ```
            
        - withUseState
            
            ```jsx
            const Component = ({initialCount}) => {
            	const [state, setState] = useState(() => init(initialCount));
            	const dispatch = (delta) => setState((prev) => reducer(prev, delta));
            }
            ```
            

### 인라인 리듀서

- 일반적으로 사용하지 않음