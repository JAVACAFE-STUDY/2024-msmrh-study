# 사용 사례 시나리오 3 : Valtio

Zustand, Jotai와 다르게 변경 가능한 갱신 모델 (mutating update model)

리액트와 통합하기 위해 proxy를 사용해 변경 불가능한 snapshot을 가져온다.

# Valtio 핵심 개념 정리

## 1. 기본 특징

- 변경 가능한 갱신 모델(mutating update model) 사용
- 리액트와의 통합을 위해 proxy와 snapshot 활용
- 두 가지 상태 업데이트 모델 제공:
  - 프락시 객체 (변경 가능)
  - 스냅숏 (불변)

## 2. 주요 기술적 특징

- ES6 프락시를 활용한 변경 감지
- 스냅숏은 필요한 경우에만 생성되어 메모리 최적화
- 중첩된 객체와 배열 완벽 지원
- 자동 리렌더링 최적화

## 3. 사용 방법

```typescript
import { proxy, useSnapshot } from "valtio";

// 상태 생성
const state = proxy({ count: 0 });

// 컴포넌트에서 사용
const Component = () => {
  const snap = useSnapshot(state);
  return <div>{snap.count}</div>;
};

// 상태 수정
state.count++; // 직접 수정 가능
```

## 4. 장단점

### 장점

- 네이티브 자바스크립트처럼 직관적인 상태 수정
- 간결한 코드 작성 가능
- 자동화된 리렌더링 최적화

### 단점

- 내부 동작의 예측 가능성이 낮음
- 디버깅이 어려울 수 있음
- 불변/가변 상태 모델 혼용으로 인한 혼란 가능성

## 5. 성능 최적화

useSnapshot()은 컴포넌트가 실제로 사용하는 속성만 추적합니다
중첩된 객체의 경우에도 변경된 부분만 새로운 참조가 생성되어 효율적입니다

```typescript
const state = proxy({
  user: {
    name: "Kim",
    settings: { theme: "dark" },
  },
});

// settings만 변경되면 user.name은 동일한 참조 유지
state.user.settings.theme = "light";
```

### 참고자료

https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/assign

https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze

https://github.com/pmndrs/valtio/blob/e65fd89fb089f810274bedb028fa49f63f98600e/src/vanilla.ts#L62
