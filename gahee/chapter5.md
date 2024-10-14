# 리액트 컨텍스트와 구독을 이용한 컴포넌트 상태 공유

## 모듈 상태의 한계

모듈 상태는 단일한 상태를 전역에서 관리하며, 컴포넌트 트리 내에서 다른 상태를 갖기 어렵습니다. 이를 극복하기 위해 Context와 모듈을 함께 사용하는 방법을 사용할 수 있습니다.

## 컨텍스트 사용이 필요한 시점

컨텍스트는 여러 컴포넌트에서 공통으로 상태를 필요로 할 때 사용합니다. 예를 들어, 테마나 사용자 인증 정보를 여러 컴포넌트에서 공유해야 하는 경우에 유용합니다.

## 컨텍스트와 구독 패턴 사용하기

구독 패턴은 상태 변경을 감지하고 필요한 경우 리렌더링을 트리거할 수 있게 합니다. 아래는 상태를 구독하여 컴포넌트 트리에 전달하는 방법입니다.

1. createStore 함수를 통해 상태 관리 및 구독 로직을 설정합니다. 상태를 업데이트하거나 구독할 수 있습니다.
2. StoreProvider 컴포넌트는 StoreContext.Provider를 사용해 트리 하위에서 상태를 사용할 수 있게 합니다.
3. useSelector 훅은 선택된 상태를 구독하고, 상태가 업데이트되면 컴포넌트가 리렌더링됩니다.


특히, useSelector 구현 방식에서 리덕스와 컨텍스트 구독 패턴을 비교하면 다음과 같습니다.

1. Redux useSelector
리덕스의 useSelector는 리덕스 스토어의 상태를 구독합니다. 아래 코드는 리덕스의 useSelector 예시입니다.

```js
import { useSelector } from 'react-redux';

const MyComponent = () => {
  const count = useSelector((state) => state.counter.value);
  
  return <div>{count}</div>;
};
```
- 리덕스 스토어 구독: 리덕스의 useSelector는 글로벌 스토어에서 상태를 가져오고, 특정 부분만 선택하여 컴포넌트에 전달합니다.
- 상태 비교 최적화: 리덕스는 내부적으로 shallowEqual을 통해 상태의 변경을 감지하고, 상태가 변하지 않으면 리렌더링을 막아 성능을 최적화합니다.
- 미들웨어 지원: 리덕스 미들웨어와 개발자 도구 지원이 있어 더 복잡한 상태 관리를 효율적으로 수행할 수 있습니다.

2. Custom useSelector in Context with Subscription
아래 코드는 컨텍스트와 구독 패턴을 사용하여 구현한 useSelector입니다.

```js
const useSelector = (selector) => {
  const store = useContext(StoreContext);

  return useSubscription({
    getCurrentValue: () => selector(store.getState()),
    subscribe: store.subscribe,
  });
};
```
- 컨텍스트 기반 구독: 이 패턴은 상태를 컨텍스트로 관리하며, 특정 컴포넌트 트리에서 상태를 공유합니다.
- 단순 구독 및 상태 관리: 상태가 변경될 때마다 구독자들에게 알리며, 리덕스처럼 상태 변경을 감지하여 필요한 경우에만 리렌더링을 합니다.
- 간소화된 구조: 리덕스와 달리, 추가적인 미들웨어나 개발자 도구 없이 구독 패턴만으로 상태를 공유하며, 국지적인 상태 관리에 더 적합합니다.





