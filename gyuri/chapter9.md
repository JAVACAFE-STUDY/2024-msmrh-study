# 사용 사례 시나리오 3 : Valtio

Zustand, Jotai와 다르게 변경 가능한 갱신 모델 (mutating update model)

리액트와 통합하기 위해 proxy를 사용해 변경 불가능한 snapshot을 가져온다.

## 또 다른 모듈 상태 라이브러리인 Valtio 살펴보기

### 1. store를 생성하여 상태를 불변으로 갱신하기

불변성 테스트
https://github.com/pmndrs/valtio/blob/main/tests/snapshot.test.ts

Valtio에서 Object를 불변하게 생성하는 부분
https://github.com/pmndrs/valtio/blob/e65fd89fb089f810274bedb028fa49f63f98600e/src/vanilla.ts#L62

실제로는 `Object.preventExtensions(obj);` 가 사용되고있음
https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/preventExtensions

- 내부적으로 Objext.assign()으로 동작

```js
moduleState = object.assign({}, moduleState, {
  count: noduleState.count + 1,
});
```

### 2. Proxy 객체를 사용하여 변경 하기

#### Proxy?

https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy

```markdown
Proxy 객체의 주요 내용을 다음과 같이 요약해드리겠습니다:

### 1. Proxy의 기본 개념

- Proxy는 객체의 기본 작업(속성 접근, 수정 등)을 가로채고 재정의할 수 있게 해주는 객체입니다
- 주로 속성 접근 로깅, 유효성 검사, 형식 지정, 삭제 등에 사용됩니다

### 2. Proxy 생성 방법

const proxy = new Proxy(target, handler);

- `target`: 프록시할 원본 객체
- `handler`: 작업을 가로채고 재정의하는 방법을 정의하는 객체

### 3. 주요 handler 메서드들

- `get`: 속성 읽기 작업을 가로챕니다
- `set`: 속성 쓰기 작업을 가로챕니다
- `has`: in 연산자의 작업을 가로챕니다
- `deleteProperty`: delete 연산자의 작업을 가로챕니다

### 4. 주요 특징과 제한사항

- 프라이빗 속성(`#`)에는 직접 접근할 수 없습니다
- 내부 슬롯이 있는 객체(예: Map)는 특별한 처리가 필요합니다
- `this` 바인딩에 주의해야 합니다

### 5. 주요 활용 사례

- 객체 속성 접근 검증
- 기본값 제공
- 로깅
- 읽기 전용 뷰 생성
- 속성 포맷팅
- DOM 요소 조작

### 6. 장점

- 객체의 기본 동작을 커스터마이즈할 수 있습니다
- 기존 코드를 수정하지 않고도 새로운 기능을 추가할 수 있습니다
- 객체의 가상 속성을 만들 수 있습니다

이러한 기능들은 객체 조작을 더욱 유연하게 만들어주며, 특히 유효성 검사나 로깅과 같은 횡단 관심사를 처리하는 데 매우 유용합니다.
```

Proxy 객체 만들기
첫 번째 인수 : 객체
두 번째 인수 : 핸들러를 담는 컬렌션 객체 (set 핸들러)

```jsx
const proxyObject = new Proxy(
  {
    count: 0,
    text: "hello",
  },
  {
    set: (target, prop, value) => {
      console.log("start setting", prop);
      target[prop] = value;
      console.log("end setting", prop);
    },
  }
);
```

- Node REPL
  - REPL은 Read-Eval(evaluation)-Print Loop의 약어로 사용자가 특정 코드를 입력하면 그 코드를 평가하고 코드의 실행결과를 출력해주는 것을 반복해주는 환경

## 프락시를 활용한 변경 감지 및 불변 상태 생성하기

Valtio는 프락시를 사용해 변경 가능한 객쳬에서 변경 불가능한 객체 "스냅숏(snapshot)"를 생성.

Object.freeze()
https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze

```js
const state = proxy({
  obj1: { c: 0 },
  obj2: { c: 0 },
});

const snap1 = snapshot(state);

++state.obj1.c;

const snap2 = snapshot(state);
```

이 경우 snap1은 `obj1: { c: 0 }, obj2: { c: 0 },`
snap2는 `obj1: { c: 1 }, obj2: { c: 0 },`

snap1과 snap1는 서로 다른 참조를 가지기 때문에 `snap1 !== snap2`
`snap1.obj2 === snap2.obj2`

Object.freeze vs Object.seal
https://ui.toast.com/posts/ko_20220420

## 프락시를 활용한 리렌더링 최적화

## 작은 애플리케이션 만들어보기

## 이 접근 방식의 장단점

Valtio에는 두 가지 상태 업데이트 모델이 있다.
불변 갱신
변경 가능한 갱신
리액트는 불변 갱신이기 때문에 proxy, snapshot으로 맞춰준다.

리액트는 불변 상태를 중심으로 만들어졌기 때문에 Valtio와 상태를 명확하게 분리해야한다.

변경 가능 갱신의 가장 큰 장점 : 네이티브 자바스크립트 함수를 사용할 수 있다

프락시 기반 렌더링 최적화의 단점 : 예측 가능성이 떨어진다. 동작을 디버깅하기 어렵다. 이 경우 개발자는 더 명시적인 선택자 기반 훅을 선호할 수도 있다.
