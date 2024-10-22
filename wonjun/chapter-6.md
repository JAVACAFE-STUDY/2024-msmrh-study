# 6장 : 전역 상태 관리 라이브러리 소개

## 01. 전역 상태 관리의 문제점

- 리액트의 중심 설계 → 컴포넌트 (모든 것이 재 사용 가능한 것으로 여겨짐)
- 지금까지의 전역 상태는 컴포넌트 외부에 존재함 → 컴포넌트의 의존성이 생겨 피해야 한다. 하지만 편했죠?

### 전역 상태 설계의 문제점

1. 읽는 방법
    - 상태는 여러 값을 가질 수 있고, 컴포넌트는 상태의 일부 값만 필요한 경우가 있다.
    - 필요 없는 상태가 변경되어도 리렌더링이 되기 때문에 이를 해결해야 한다
2. 쓰는 방법
    - 중첩된 객체를 가지는 상태의 경우 전역 변수가 1개일 경우 객체를 `{ ...object }` 의 형태로 스프레딩 하여 변경해야 한다.
    - 솔직히 귀찮다.

### 위에꺼 해결법

1. 번은 다른 챕터에서 다룰 예정이라고 한다 (selector 같은 사용 방법을 예시로 들것 같다)
2. **전역 상태를 직접 변경하지 않고, 상태를 변경하는 함수를 사용해야 할 것이다. (이번 장에서 다룰 예정)**

## 02. 데이터 중심 접근 방식

- 데이터 모델은 싱글턴으로 가질 수 있고, 처리할 데이터가 이미 있는 경우에 사용할 수 있는 방법
- 컴포넌트 정의 → 데이터 연결 순으로 작업하며, 라이브러리 / 서버 등에서 데이터를 변경할 수도 있다.

### 특징

- 상태가 JS 메모리에 있기 때문에 모듈을 사용하는 편이 바람직하다
- 리액트의 생명주기와 연관이 없어 마운트 이전 / 마운트 해제 이후에도 존재할 수 있다.
- 모듈 상태를 생성하고 모듈 상태를 React와 연결하는 API를 제공한다.
- 상태는 모듈을 통해 접근하고 별도의 wrapper 를 사용하여 읽거나 쓴다.

## 03. 컴포넌트 중심 접근 방식

- 데이터 모델이 컴포넌트에 강한 의존성을 가지고 있다.
- 컴포넌트의 생명 주기 내에서 전역 상태가 유지된다 → 상태를 의존하는 컴포넌트가 사라지면 상태도 사라진다
- 팩토리 함수를 통해 전역 상태를 초기화해 리액트가 전역 상태의 생명 주기를 처리하도록 한다.

## 04. 예외

- 모듈 상태는 대체로 싱글턴이지만, 싱글턴이 아닐수도 있다.
- 컴포넌트 상태는 하위 트리에 상태를 제공하기 위해서도 사용된다. (Context 중첩)
- useState 말고도 useRef 훅을 사용해서도 컴포넌트 상태를 구현할 수 있다.

## 05. 리렌더링 최적화

이런 상태가 있을 때 최적화 하는 방법

```tsx
const state = {
  a: 1,
  b: { c: 2, d: 3 },
  e: { f: 4, g: 5 },
};
```

### 셀렉터 함수 (수동 렌더링 최적화)

[https://github.com/reduxjs/reselect](https://github.com/reduxjs/reselect)

- 전체 상태를 받아서 상태의 일부를 반환한다

```tsx
function Component() {
	const val = useSelector((state) => state.b);
}
```

### 속성 접근 감지 (자동 렌더링 최적화)

[https://github.com/dai-shi/react-tracked](https://github.com/dai-shi/react-tracked)

- 특정 값이 참조될 때에만 리렌더링을 발생시킨다.
- 별도의 Proxy 가 필요하다.

```tsx
function Component() {
	const val = useTrackedState();
	
	return <>{val.b.c}</>
}
```

- 이런 경우에는 최적화가 안된다.

```tsx
function Component() {
	// state.a 가 변경될 때마다 리렌더링 된다.
	const val = useTrackedState().a < 10;
	
	return <>{val ? "good" : "not good"}</>
}
```

### 아톰 사용

[https://github.com/pmndrs/jotai](https://github.com/pmndrs/jotai)

- 아톰 : 리렌더링을 발생시키는 최소한의 상태 단위

```tsx
const globalState = {
  a: atom(1),
  b: atom(2),
  e: atom(3),
};

function Component() {
  const val = useAtom(globalState.a);

  return <>{val}</>;
}
```
