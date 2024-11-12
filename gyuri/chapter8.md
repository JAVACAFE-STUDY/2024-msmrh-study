# 사용 사례 시나리오 2: Jotai

컴포넌트 상태를 사용하는 상태 관리 라이브러리입니다. (Zustand는 모듈 상태 사용)

## Jotai 이해하기

### 구문 단순성

Jotai는 React의 useState와 매우 유사한 API를 제공합니다. useAtom(countAtom)은 useState()와 같은 튜플인 [count, setCount]를 반환하기 때문에 기존 React 코드를 쉽게 마이그레이션할 수 있습니다.

### 동적 아톰 생성

Jotai는 WeakMap을 사용하여 동적으로 아톰을 생성하고 관리합니다. 이는 메모리 관리에 효율적이며, 더 이상 참조되지 않는 아톰은 자동으로 가비지 컬렉션됩니다.

## 렌더링 최적화

Jotai는 두 가지 주요 접근 방식을 제공합니다:

1. 선택자 접근 방식 (하향식/top-down)

   - 큰 상태 객체에서 필요한 부분만 선택하여 사용
   - 불필요한 리렌더링 방지
   - 성능 최적화에 유용

2. 상향식 접근법 (bottom-up)
   - 작은 아톰들을 먼저 만들고 이를 결합하여 더 큰 아톰 생성
   - 더 세밀한 상태 관리 가능
   - 재사용성이 높음

## Jotai가 아톰 값을 저장하는 방식 이해하기

### 아톰속 아톰들(Atoms-in-Atom)

- 아톰은 다른 아톰을 참조할 수 있음
- 중첩된 상태 구조 생성 가능
- 복잡한 상태 로직을 모듈화하고 재사용하기 용이
- 파생된 상태(derived state) 쉽게 생성 가능

## Jotai의 다양한 기능 사용하기

- 아톰의 write 함수 정의하기
- 액션 아톰 사용하기
- 아톰의 onMount 옵션 이해하기
- jotai/utils 번들 소개하기

주요 특징:

- 타입스크립트 지원이 우수
- React Suspense 완벽 지원
- 비동기 아톰 지원
- 미들웨어 시스템 제공