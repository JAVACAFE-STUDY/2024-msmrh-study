# Chapter-03. 리액트 컨텍스트를 이용한 컴포넌트 상태 공유

- 컨텍스트(Context)
    - props를 대신해서 컴포넌트 간에 데이터를 전달하는 것이 가능
    - 컴포넌트 상태와 결합하면 전역 상태를 제공할 수 있음
    - **전역 상태를 위해 설계된 것은 아님**
        
        → 종종 나오는 논쟁(?)거리, 상태 관리 도구는 아님
        
    - 상태가 갱신될 때 모든 컨텍스트 소비자(consumer)가 리렌더링되므로 불필요한 렌더링이 발생할 수 있음
        - 여러 조각으로 나누어 사용하는 것이 권장됨

## useState와 useContext 탐구하기

### useContext 없이 useState 사용하기

- 상태 끌어올리기(lifting state up) 패턴을 통해 상태를 부모 컴포넌트로 올리고,
    - 프로퍼티 내려꽂기(props drilling)
        - 부모 컴포넌트에서 자식 컴포넌트로 props를 전달하는 작업
- 애플리케이션 규모가 커진다면 트리 아래로 props를 전달하는 것이 적절하지 않을 수 있음, Parent 컴포넌트가 count 상태에 대해 알 필요가 없으므로 상태를 제거하는 편이 합리적일 수 있음

### 정적 값을 이용해 useContext 사용하기

- 컨텍스트를 사용하면 props를 사용하지 않고도 부모 컴포넌트에서 트리 아래에 있는 자식 컴포넌트로 값을 전달하는 것이 가능
- 공급자(provider)
    - 값을 제공하는 역할
    - 중첩될 수 있음
    - 소비자 컴포넌트(`useContext`가 있는 컴포넌트)는 컴포넌트 트리 중에서 가장 가까운 공급자를 선택해 컨텍스트 값을 가져옴

```jsx
const ColorContext = createContext('black')
```

- **기본값은 컴포넌트 트리에 컨텍스트 공급자가 없으면 사용됨**
    - **Provider로 감싸여지지 않은 소비자 컴포넌트의 경우 기본값 ‘black’이 사용됨**
    
    → ‘프로바이더 적용 === 해당 범위에 내에서 컨텍스트를 사용하겠다’는 의미라고 생각해서 기본값을 사용 안 하는 편 → 책 뒤에 나옴
    
- 소비자 컴포넌트

```jsx
const Component = () => {
	const color = useContext(ColorContext);
	
	return <div style={{ color }}>Hello {color}</div>;
}
```

```jsx
<ColorContext.Provider value="skublue">
	<Component />
</ColorContext.Provider>
```

### useContext와 함게 useState 사용하기

```jsx
const CountStateContext = createContext({ 
  count: 0,
  setCount: () => {}
});
```

- 기본값은 타입을 유추하는데 도움이 됨
- 대부분의 경우 기본값이 그다지 유용하지 않기 때문에 정적인 값 대신 상태가 필요
    - 기본값을 사용하는 것은 의도하지 않은 것이므로 에러가 발생할 수도 있음

→ 더 명확하고 구체적인 타입 정의를 위해 명시적으로 제네릭으로 타입을 선언하는 것을 선호하는 편

→ 위에 언급한 것처럼 provider로 감싸지 않았는데 기본값이 출력되는 건 사용 의도에 맞지 않다고 생각

→ null, undefined 등을 기본값으로 설정하고 타입을 선언해주는 편

- 소비자 컴포넌트는 가장 가까운 공급자로부터 컨텍스트 값을 가져와서 사용함

## 컨텍스트 이해하기

- 컨텍스트 공급자가 새로운 컨텍스트 값을 갖게 되면 모든 컨텍스트 소비자는 새로운 값을 받고 리렌더링됨
    - 공급자의 값이 모든 소비자에게 전파된다는 것을 의미

### 컨텍스트 전파의 작동 방식

- 컨텍스트 공급자가 새로운 컨텍스트 값을 받으면 모든 컨텍스트 소비자 컴포넌트가 리렌더링됨
    - 리렌더링 이유?
        - 부모 컴포넌트가 리렌더링 되어서
        - 컨텍스트가 변경되어서
- 컨텍스트의 값이 변경되지 않았는데도 리렌더링이 발생하는 문제를 방지하려면 ‘내용 끌어올리기’ 또는 `memo`를 사용
    - `memo`를 사용하더라도 컨텍스트가 변경되어서 발생하는 리렌더링을 막지는 못함
        - 컴포넌트가 일관된 컨텍스트 값을 가지기 위해서 불가피한 일

### 컨텍스트에 객체를 사용할 때의 한계점

- 객체에는 여러 가지 값을 포함할 수 있으며, 컨텍스트 소비자는 모든 값을 사용하지 않을 수 있음
- 하나의 값만 사용하더라도 객체 내의 다른 값이 변경된 경우 불필요한 리렌더링이 발생함

## 전역 상태를 위한 컨텍스트 만들기

### 작은 상태 조각 만들기

- 전역 상태를 여러 조각으로 나누기

```jsx
const App = () => {
	<Counter1Provider>
		<Counter2Provider>
			<Parent />
		</Counter2Provider>
	</Counter1Provider>
}
```

- `Counter1`, `Counter2` 컴포넌트는 각각 `counter1`, `counter2`가 변경될 때만 리렌더링됨

### useReducer로 하나의 상태를 만들고 여러 개의 컨텍스트로 전파하기

- 단일 상태를 만들고 여러 컨텍스트를 사용해 상태 조각을 배포
    - 상태를 갱신하는 함수를 배포하는 것은 별도의 컨텍스트로 해야 함

```tsx
type Action = { type: 'INC1' } | { type: "INC2" };

const Count1Context = createContext<number>(0);
const Count2Context = createContext<number>(0);
const DispatchContext = createContext<Dispatch<Action>>(
	() => {}
);
```

```tsx
const Counter1 = () = {
	const count1 = useContext(Count1Context);
	const dispatch = useContext(DispatchContext);
	...
}
```

```tsx
const Provider = ({ children } : { children: ReactNode }) => {
	const [state, dispatch] = useReducer(
		(
			prev: { count1: number; count2; number },
			action: Action
		) => {
			if (action.type === "INC1") {
				return { ...prev, count1: prev.count1 + 1 };
			}
			if (action.type === "INC2") {
				return { ...prev, count2: prev.count2 + 1 }
			}
			throw new Error("no matching action")
		},
		{
		  count1: 0,
		  count2: 0,
		}
	);
	
	return (
		<DispatchContext.Provider value={dispatch}>
			<Count1Context.Provider value={state.count1}>
				<Count2Context.Provider value={state.count2}>
					{children}
				</Count2Context.Provider>
			</Count1Context.Provider>
		</DispatchContext.Provider>	
	);	
};
```

- 불필요한 리렌더링이 발생하지 않음
- 단일 상태를 사용할 때의 장점
    - 단일 상태에서는 하나의 액션으로 여러 조각을 갱신할 수 있음

## 컨텍스트 사용을 위한 모범 사례

### 사용자 정의 훅과 공급자 컴포넌트 만들기

- 컨텍스트의 기본값을 `null`로 지정
    - 기본값을 사용할 수 없고 공급자가 항상 필요함

```tsx
type CountContextType = [number, Dispatch<SetStateAction<number>>]

const Count1Context = createContext<CountContextType | null>(null);
```

```tsx
export const Count1Provider = ({ children } : { children: ReactNode }) => {
	<Count1Context.Provider value={useState(0)}>
		{children}
	</Count1Context.Provider>
}
```

```tsx
export const useCount1 = () => {
	const value = useContext(Count1Context);
	if (value === null) throw new Error("Provider missing");
	return value;
}
```

- `Count1` 컴포넌트가 `useCount1` 훅에 숨겨진 컨텍스트에 대해서는 알지 못함
- 일반적으로 각 컴포넌트에 대해 `contexts/count1.jsx` 와 같은 파일을 별도로 두고 `useCount1`과 같은 사용자 정의 훅 및 `Count1Provider`와 같은 컴포넌트만 export 함
    - `Count1Context`는 `export` 하지 않음

### 사용자 정의 훅이 있는 팩토리 패턴

- createStateContext
    - 초깃값을 받아 상태를 반환하는 `useValue` 사용자 정의 훅을 사용
    - 상태를 가져오는 사용자 정의 훅과 공급자 컴포넌트 튜플을 반환
    - 공급자 컴포넌트에서는 `useValue`에 전달되는 선택적인 `initialValue`를 받는 새로운 기능을 제공함
        - 생성 시 초깃값을 정의하는 대신 런타임에 상태의 초깃값을 설정할 수 있음
    
    ```tsx
    const createStateContext = (
    	useValue: (init) => State,
    ) => {
    	const StateContext = createContext(null);
    	
    	const StateProvider = ({
    		initialValue,
    		children,
    	}) => (
    		<StateContext.Provider value={useValue(initialValue)}>
    			{children}
    		</StateContext.Provider>
    	);;
    	
    	const useContextState = () => {
    		const value = useContext(StateContext);
    		
    		if (value === null) throw new Error("Provider missing");
    		
    		return value;
    	};
    	
    	return [StateProvider, useContextState] as const;
    };
    ```
    

- 사용 예
    
    ```tsx
    const useNumberState = (init) => useState(init || 0);
    
    const [Counter1Provider, useCount1] = createStateContext(useNumberState);
    ```
    

- useState 대신 useReducer를 이용한 사용자 정의 훅
    
    ```tsx
    const useMyState = () => useReducer({}, (prev, action) => {
    	if (action.type === 'SET_FOO') {
    		return { ...prev, foo: action.foo };
    	}
    	// ...
    };
    ```
    

- 더 복잡한 훅
    
    ```tsx
    const useMyState = (initialState = { count1: 0, count2: 0 }) => {
    	const [state, setState] = useState(initialState);
    	
    	useEffect(() => {
    		console.log('updated', state);
    	});
    	
    	const inc1 = useCallback(() => {
    		setState((prev) => ({
    			...prev,
    			count1: prev.count1 + 1
    		}));
    	}, []);
    	
    	const inc2 = useCallback(() => {
    		setState((prev) => ({
    			...prev,
    			count2: prev.count2 + 1
    		}));
    	}, []);
    	
    	return [state, { inc1, inc2 }];
    }
    ```
    

- 타입
    
    ```tsx
    const createStateContext = <Value, State>(
    	useValue: (init?: Value) => State,
    ) => {
    	const StateContext = createContext<State | null>(null);
    	
    	const StateProvider = ({
    		initialValue,
    		children,
    	}: {
    		initialValue?: Value;
    		children?: ReactNode;
    	}) => (
    		<StateContext.Provider value={useValue(initialValue)}>
    			{children}
    		</StateContext.Provider>
    	);;
    	
    	const useContextState = () => {
    		const value = useContext(StateContext);
    		
    		if (value === null) throw new Error("Provider missing");
    		
    		return value;
    	};
    	
    	return [StateProvider, useContextState] as const;
    };
    ```
    

### reduceRight를 이용한 공급자 중첩 방지

```tsx
const App = () => {
	const providers = [
		[Count1Provider, { initialValue: 10 }],
		[Count2Provider, { initialValue: 20 }],
		[Count3Provider, { initialValue: 30 }],
		[Count4Provider, { initialValue: 40 }],
		[Count5Provider, { initialValue: 50 }],
	] as const;
	
	return providers.reduceRight(
		(children, [Component, props]) => 
			createElement(Component, props, children),
		<Parent />,
	);	
};
```