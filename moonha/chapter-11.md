## Zustand 와 Redux의 차이점

- 공통점
    - 단방향 데이터 흐름을 기반
        - `action`을 실행하여 상태를 변경하고, 상태가 변경되면 새로운 상태가 전파됨
        - 디스패치와 전파를 분리하여 데이터의 흐름을 단순화하고 전체 시스템을 더 예측 가능하게 함
- 차이점
    - 상태를 갱신하는 방법에 차이
        - `Redux`
            - 리듀서에 기반
                - `리듀서`
                    - 이전 상태와 `action` 객체를 받아 새로운 상태를 반환하는 순수 함수
                - 예측 가능성이 더 높음
        - `Zustand`
            - 반드시 리듀서를 사용해 상태를 갱신할 필요는 없음

### Redux와 Zustand를 사용한 예제

- `Redux`
    - 스토어 생성
        
        ```tsx
        import { configureStore } from "@reduxjs/toolkit";
        import counterReducer from "../features/counter/counterSlice";
        
        export const store = configureStore({
          reducer: {
            counter: counterReducer,
          },
        });
        ```
        
    - `Slice` 정의
        
        ```tsx
        const initialState = {
          value: 0,
        };
        
        export const counterSlice = createSlice({
          name: "counter",
          initialState,
          reducers: {
            increment: (state) => {
              state.value += 1;
            },
            decrement: (state) => {
              state.value -= 1;
            },
            incrementByAmount: (state, action: PayloadAction<number>) => {
              state.value += action.payload;
            },
          },
        });
        
        export const { increment, decrement, incrementByAmount } = counterSlice.actions;
        export default counterSlice.reducer;
        ```
        
        - Slice는 리듀서와 액션을 모두 포함함
    - `store` 사용
        
        ```tsx
        import { useSelector, useDispatch } from "react-redux";
        import { decrement, increment } from "./counterSlice";
        
        export function Counter() {
          const count = useSelector(
            (state: { counter: { value: number } }) => state.counter.value
          );
          
          const dispatch = useDispatch();
          
          return (
            <div>
              <button onClick={() => dispatch(increment())}>Increment</button>
              <span>{count}</span>
              <button onClick={() => dispatch(decrement())}>Decrement</button>
            </div>
          );
        }
        ```
        
        - `useSelector` 를 통해 스토어의 상태를 가져옴
        - `useDispatch` 를 통해 액션을 디스패치
        - **스토어를 직접 사용하지 않음**
            - `useSelector` 훅은 컨텍스트에서 스토어를 가져옴
    - `App` 컴포넌트 정의
        
        ```tsx
        import { Provider } from "react-redux";
        import { store } from "./app/store";
        import { Counter } from "./features/counter/Counter";
        
        const App = () => (
          <Provider store={store}>
            <div>
              <Counter />
              <Counter />
            </div>
          </Provider>
        );
        ```
        
        - 생성한 `store` 변수를 `Provider` 컴포넌트로 전달하고, 이를 통해 `useSelector` 훅이 `store`에 접근할 수 있게 됨
        
- `Zustand`
    - `store` 정의
        
        ```tsx
        import create from "zustand";
        
        type State = {
          counter: {
            value: number;
          };
          counterActions: {
            increment: () => void;
            decrement: () => void;
            incrementByAmount: (amount: number) => void;
          };
        };
        
        export const useStore = create<State>((set) => ({
          counter: { value: 0 },
          counterActions: {
            increment: () =>
              set((state) => ({ counter: { value: state.counter.value + 1 } })),
            decrement: () =>
              set((state) => ({ counter: { value: state.counter.value - 1 } })),
            incrementByAmount: (amount: number) =>
              set((state) => ({ counter: { value: state.counter.value + amount } })),
          },
        }));
        ```
        
        - 리듀서 로직은 액션의 함수 내부에서 구현됨
    - `store` 사용
        
        ```tsx
        import { useStore } from "./store";
        
        export function Counter() {
          const count = useStore((state) => state.counter.value);
        
          const { increment, decrement } = useStore((state) => state.counterActions);
        
          return (
            <div>
              <div>
                <button onClick={increment}>Increment</button>
                <span>{count}</span>
                <button onClick={decrement}>Decrement</button>
              </div>
            </div>
          );
        }
        ```
        
        - `useStore` 훅을 사용해 상태와 상태를 갱신하는 액션을 가져옴
            - **`useStore`은 직접 `store` 파일에 접근해서 가져옴**
    - `App` 컴포넌트 정의
        
        ```tsx
        import { Counter } from "./Counter";
        
        const App = () => (
          <div>
            <Counter />
            <Counter />
          </div>
        );
        ```
        
        - 컨텍스트를 사용하지 않았기 떄문에 공급자 컴포넌트가 필요하지 않음

### Redux와 Zustand 예제 비교

- 디렉터리 구조
    - `Redux`
        - `features` 디렉터리 구조를 제안
        - 대규모 애플리케이션에 유용한 패턴
    - `Zustand`
        - 구조에 대한 의견을 제시하지 않음
        - 파일과 디렉터리를 어떻게 구성할지는 개발자의 몫
- `Immer` 사용 여부
    - `Redux`
        - 최신 `Redux`는 기본적으로 Immer를 사용
    - `Zustand`
        - `Immer`를 사용하지 않음
- 상태 전파
    - `Redux`
        - 컨텍스트를 사용
    - `Zustand`
        - 모듈 임포트를 사용
        - 컨텍스트를 선택적으로 지원
- 데이터 흐름
    - `Redux`
        - 단방향 데이터 흐름을 기반
        - 상태를 갱신하려면 액션을 디스패치해야 함
        - 유지보수성과 확장성에 도움이 됨
    - `Zustand`
        - 데이터 흐름 측면에 대한 의견을 제시하지 않음
        - 단방향 데이터 흐름을 사용할 수 있지만 라이브러리 지원이 없음
- 정리
    - `Redux`
        - 상태를 어떻게 관리할지에 대해 더 많은 의견을 제시 (라이브러리에 이미 정해진 룰이 있음)
    - `Zustand`
        - 적극적인 의견을 제시하지 않음

## Jotai와 Recoil을 사용하는 시점

### Recoil과 Jotai 예제

- `Recoil`
    - 상태 생성
        
        ```tsx
        const textState = atom({
          key: "textState",
          default: "",
        });
        ```
        
        - `key` 문자열과 `default` 값 필요
    - 상태 사용
        
        ```tsx
        const TextInput = () => {
          const [text, setText] = useRecoilState(textState);
          
          return (
            <div>
              <input
                type="text"
                value={text}
                onChange={(event) => {
                  setText(event.target.value);
                }}
              />
              <br />
              Echo: {text}
            </div>
          );
        };
        ```
        
        - `useRecoilState` 훅 사용
    - 파생 상태 생성
        
        ```tsx
        const charCountState = selector({
          key: "charCountState",
          get: ({ get }) => get(textState).length,
        });
        ```
        
        - `selector` 함수 사용
        - `key` 문자열과 파생 값을 반환하는 `get` 함수가 필요
    - 파생 상태 사용
        
        ```tsx
        const CharacterCount = () => {
          const count = useRecoilValue(charCountState);
          
          return <>Character Count: {count}</>;
        };
        ```
        
        - 상태의 값 부분만 반환하는 `useRecoilValue` 훅 사용
    - `App` 컴포넌트 정의
        
        ```tsx
        const App = () => (
          <RecoilRoot>
            <CharacterCounter />
          </RecoilRoot>
        );
        ```
        
        - 상태 값을 저장하는 `RecoilRoot` 컴포넌트를 사용

- `Jotai`
    - 아톰 생성
        
        ```tsx
        const textAtom = atom("");
        ```
        
        - `key` 문자열이 불필요함
        - 변수 이름 뒤에 `State` 대신 `Atom`을 접미사로 사용하는 것이 관례
    - 아톰 사용
        
        ```tsx
        const TextInput = () => {
          const [text, setText] = useAtom(textAtom);
          
          return (
            <div>
              <input
                type="text"
                value={text}
                onChange={(event) => {
                  setText(event.target.value);
                }}
              />
              <br />
              Echo: {text}
            </div>
          );
        };
        ```
        
        - `useAtom` 훅 사용
    - 파생 아톰 생성
        
        ```tsx
        const charCountAtom = atom((get) => get(textAtom).length);
        ```
        
        - `atom` 함수 사용
    - 파생 아톰 사용
        
        ```tsx
        const CharacterCount = () => {
          const [count] = useAtom(charCountAtom);
          
          return <>Character Count: {count}</>;
        };
        ```
        
        - `useAtom` 훅 사용
        
        (→  Recoil과 마찬가지로 값만 반환하는 `useAtomValue` 훅 있음)
        
    - `App` 컴포넌트 정의
        
        ```tsx
        const App = () => (
          <>
            <CharacterCounter />
          </>
        );
        ```
        
        - `Provider` 컴포넌트 필요 없음

### Recoil과 Jotai 예제 비교

- `key` 문자열의 존재 여부
    - `Jotai`
        - `key` 문자열이 불필요함
        - `WeakMap`을 활용하고 아톰 객체의 참조에 의존함
    - `Recoil`
        - 객체 참조에 의존하지 않는 `key` 문자열을 기반으로 함
        - **key 문자열의 장점은 직렬화가 가능하다는 것**
            - `localStorage`, `sessionStorage` 혹은 그와 유사한 기능을 이용해 상태를 지속적으로 보관하는 것을 쉽게 구현할 수 있음
- 통합된 atom 함수
    - `Jotai`
        - `atom` 함수가 `Recoil`의 `atom`과 `selector` 두 가지 모두를 대체함
- 공급자 여부
    - `Jotai`
        - Provider 컴포넌트를 생략할 수 있게 해주는 공급자 제거 모드(provider-less mode)

## Valtio와 MobX 사용하기

- 유사점
    - 변경 가능한 상태를 기반으로 함
- 차이점
    - 렌더링 최적화 방법
        - `Valtio`
            - 훅을 사용
        - `MobX`
            - `HOC`를 사용

### MobX, Vatio 관련 예제

- `MobX`
    - 타이머 인스턴스 정의
        
        ```tsx
        import { makeAutoObservable } from "mobx";
        import { observer } from "mobx-react";
        
        class Timer {
          secondsPassed = 0;
          
          constructor() {
            makeAutoObservable(this);
          }
          
          increase() {
            this.secondsPassed += 1;
          }
          
          reset() {
            this.secondsPassed = 0;
          }
        }
        
        const myTimer = new Timer();
        ```
        
        - `makeAutoObservable` 훅은 `myTimer` 인스턴스를 관찰 가능한 객체로 만드는 데 사용됨
    - 컴포넌트 정의
        
        ```tsx
        const TimerView = observer(({ timer }: { timer: Timer }) => (
          <button onClick={() => timer.reset()}>
            Seconds passed: {timer.secondsPassed}
          </button>
        ));
        ```
        
        - `observer` 함수
            - 고차 컴포넌트
            - 렌더링 함수에서 `timer.secondsPassed`가 사용됐다는 것을 알고 `timer.secondsPassed`가 변경될 때마다 리렌더링을 발생시킴
    - `App` 컴포넌트 정의
        
        ```tsx
        const App = () => (
          <>
            <TimerView timer={myTimer} />
          </>
        );
        ```
        

- `Valtio`
    - 타이머 인스턴스 정의
        
        ```tsx
        const myTimer = proxy({
          secondsPassed: 0,
          
          increase: () => {
            myTimer.secondsPassed += 1;
          },
          
          reset: () => {
            myTimer.secondsPassed = 0;
          },
        });
        ```
        
    - 컴포넌트 정의
        
        ```tsx
        const TimerView = ({ timer }: { timer: typeof myTimer }) => {
          const snap = useSnapshot(timer);
        
          return (
            <button onClick={() => timer.reset()}>
              Seconds passed: {snap.secondsPassed}
            </button>
          );
        };
        ```
        
        - `useSnapshot`
            - 렌더링 함수에서 상태가 어떻게 사용되는지 알고 사용된 부분이 변경될 때 리렌더링되게 하는 훅
    - `App` 컴포넌트 정의
        
        ```tsx
        const App = () => (
          <>
            <TimerView timer={myTimer} />
          </>
        );
        ```
        

### MobX와 Valtio 예제 비교

- 갱신 방식
    - `MobX`
        - 클래스 기반
    - `Valtio`
        - 객체 기반
        - 특정 스타일을 강요하지 않음
        - 상태 객체에서 함수를 분리하는 스타일도 허용
            
            ```tsx
            const timer = proxy({ secondsPassed: 0 })
            
            export const increase = () => {
              myTimer.secondsPassed += 1;
            };
              
            export const reset = () => {
              myTimer.secondsPassed = 0;
            };
            
            export const useSecondsPasses = () => useSnapshot(timer).secondsPassed;
            ```
            
            - 코드 분할과 최소화, 불필요한 코드 제거가 가능
                - 결과적으로 번들 크기 최적화를 기대
- 렌더링 최적화 방식
    - `MobX`
        - 옵저버 방식
            - 더 예측 가능성이 높음
    - `Valtio`
        - 훅 방식
            - 동시성 렌더링에 더 친화적

## Zustand, Jotai, Valtio 비교하기

- 유사점
    - `Poimandres`에서 제공됨
        - 라이브러리를 제공하는 개발자 집단
    - 작은 API를 제공하는 공통 철학을 가짐
        - 개발자의 필요에 따라 API를 조합할 수 있게 하려고 노력함
- 차이점
    - 상태가 어디에 위치하는가?
        - 모듈 상태
            - 모듈 수준에서 생성되는 상태
            - 리액트에 속하지 않는 상태
            - `Zustand`, `Valtio`
        - 컴포넌트 상태
            - 리액트 컴포넌트 생명 주기에서 생성되고 리액트에 의해 제어되는 상태
            - `Jotai`
    - 상태 갱신 스타일이 무엇인가?
        - 불변 상태 모델
            - `Zustand`
            - 객체 참조를 비교해서 변경 사항이 있는지 파악할 수 있음 → 규모가 크고 중첩된 객체의 성능 향상에 도움이 됨
            - 리액트가 불변 모델에 기반하기 때문에 호환성이 좋음
        - 변경 가능한 상태 모델
            - `Valtio`