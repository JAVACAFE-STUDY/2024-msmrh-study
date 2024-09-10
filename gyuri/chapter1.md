# 1. 리액트 훅과 마이크로 상태 관리

## 마이크로 상태 관리 이해하기

- 목적 지향적, 특정 코딩 패턴과 함께 사용
- 특정 목적에 따른 해결책 제공
- 기본적인 상태 관리 기능 포함
- 리액트 상태: UI를 나타내는 모든 데이터
- 배우기 쉬워야함

## 리액트 훅 사용하기

- useState
- useReducer
- useEffect
- 장점:
- UI 컴포넌트에서 로직 추출 가능
- 사용자 정의 훅 (예: useCount) 생성 가능

사용자 정의 훅 이점:

1. 코드 가독성 향상
2. 컴포넌트와 로직 분리
3. 새로운 규칙 추가 용이

> 💡사용자 정의 훅
> https://www.npmjs.com/search?q=react%20hooks > https://github.com/search/?q=reacthooks&type=repositories

### 데이터 불러오기를 위한 서스펜스와 동시성 렌더링

서스펜스 : 비동기 처리(async)에 대한 걱정 없이 컴포넌트를 코딩할수 있는방법
동시성 렌더링 : 렌더링 프로세스를 청크(chunk)라는 단위로 분할해서 중앙 처리 장치 (CPU)가 장시간 차단되는것을 방지하는 방법

주의사항:

- state, ref 객체 직접 변경 금지 (Why? 리렌더링시 예기치 않은 동작이 발생할 수 있다.)
- 훅과 컴포넌트 함수는 순수해야 함 (Why? 여러 번 호출될 수 있기 때문)

> 💡React 18에서는?
> https://github.com/reactwg/react-18/discussions/4

React 18 소개
`startTransition`:

- 비용이 많이 드는 상태 전환 동안 UI의 반응성을 유지할 수 있게 해줍니다.
- 예를 들어, 큰 데이터 세트를 렌더링하는 동안 사용자 입력에 즉시 반응할 수 있습니다.
  `useDeferredValue`:
- 화면의 덜 중요한 부분의 업데이트를 지연시킬 수 있습니다.
- 중요한 콘텐츠를 먼저 업데이트하고, 덜 중요한 부분은 나중에 업데이트할 수 있습니다.
  Streaming SSR with selective hydration
- 페이지의 일부분씩 점진적으로 로드하고 hydrate(JavaScript 이벤트 연결)할 수 있습니다.

## 전역 상태 탐구하기

구현 어려움 (Why? 컴포넌트 모델 때문)
컴포넌트: 격리 및 재사용 필요하기 때문에 전역 상태에 의존하지 않는 것이 좋다.

리액트에서 전역 상태를 구현하는 것은 간단한 작업이 아니다. 그 이유는 리액트가 컴포넌트 모델에 기반하기때문이다. 컴포넌트 모델에서는 지역성(locality)이 중요하며, 이는 컴포넌트가 서로 격리돼야하고 재사용이가능해야한다는것을의미한다

## useState 사용하기

### 값으로 상태 갱신하기

> 💡베일아웃(bailout) : 리렌더링을 발생시키지 않는 것을 의미

- 동일 값: 베일아웃으로 리렌더링 방지
- 객체: 얕은 비교 수행

```tsx
const Component = 0 => {
const [count, setCountj = useState(0);

return (
<div>
{count}
<button onCtick={() => setCount(count + 1))>
Set Count to {count + 1} </button>
</div>
```

버튼을두 번 빠르게클릭해도한 번만증가

### 함수로 상태 갱신하기

- 이전 상태 기반 새 상태 계산 가능

```tsx
// 클릭 횟수 세는 의미에서는 보통 이렇게 함수 상태로 사용
<button onClick={() => setCount((c) => c + 1)}>

Increment Count </button>


// 베일 아웃을 조정할 수 도 있다.
<button
onCtick={() => setCount((c)=>c%2===0? c: c+1)}
>
Increment Count if it makes the result even

</button>
```

### 지연 초기화

- 복잡한 초기 상태 계산에 사용
- 불필요한 재계산 방지
- 초기 로딩 시간 단축

> 💡어떤 경우에 지연 초기화를 사용할 수 있을까?

- 복잡한 데이터 구조 생성, 파일 시스템 작업, 네트워크 요청, DB쿼리 등등

> 💡지연 초기화를 사용하지 않으면 어떤 단점이 있을까?

- 불필요한 재계산:
  - 리렌더링될 때마다 복잡한 계산이 반복됨 -> 성능 저하
- 초기 로딩 시간 증가:
  - 컴포넌트가 처음 마운트될 때 무거운 연산이 즉시 실행됨 -> 초기 로딩 시간 증가

## useReducer 사용하기

### 기본 사용법

- 복잡한 상태 관리에 유용
- reducer 분리 가능
- 순수 함수로 테스트 용이

### 베일아웃

- 동일 state 반환: 베일아웃 발생
- 새 객체 생성: 베일아웃 없음

### 원시값

- 숫자, 문자열 등에 작동
- 외부 복잡 로직 정의 가능

객쳬가 아닌 값, 즉 숫자나 문자열 같은 원시 값에 대해 작동한다
useReducer 에 원시값을사용하는것은 외부에서 복잡한 리듀서 로직을 정의할수 있기 때문에 유용하다.

```tsx
const reducer = (count, delta) => { if (delta K0) {

thrownewError('delta cannotbenegative');

}
if (delta >10) {

// 너무 크다면 무시

return count

I
if (countK100){

// 보너스를 더한다

return count + delta + 10

}
return count + delta

}
```

### 지연 초기화(init)

- 세 번째 매개변수로 초기화 함수 전달

```tsx
const [state, dispatch] = useReducer(reducer, 0, init);
```

## useState와 useReducer의 유사점과 차이점

Preact에서 useState와 useReducer
https://github.com/preactjs/preact/blob/b976caa6790bfce77ee4e4621963a8cc5965c537/hooks/src/index.js#L166

React에서 useState
https://github.com/facebook/react/blob/d160aa0fbb1bd2d00ea8c771c551c9cb5b47f1e9/packages/react/src/ReactHooks.js#L93

### useReducer를 이용한 useState 구현

- useReducer로 useState 구현 가능

### useState를 이용한 useReducer 구현

- useState로 useReducer 거의 대체 가능

### 초기화 함수 사용하기

- useReducer: 외부에서 reducer, init 정의 가능

### 인라인 리듀서 사용하기

- 인라인 리듀서: 외부 변수 의존 가능, 잘 안 씀

```tsx
const useScore = (bonus) =>
  useReducer((prev, delta) => prey + delta + bonus, 0);
```
