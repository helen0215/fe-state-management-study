# Recoil

2020년 5월 14일에 온라인으로 진행된 React Europe 2020 에서 처음으로 소개 되었으며, Facebook에서 만든 React용 상태 관리 라이브러리
https://github.com/facebookexperimental/Recoil
> A state management library for React <br/>
> 리액트를 위한 상태 관리 라이브러리

<br/>

## 기존 상태 관리 라이브러리들
Redux, Mobx
- 많은 보일러플레이트와 코드가 많이 필요함
- 비동기 처리시 다른 라이브러리를 사용해야함
- react 라이브러리가 아니라 내부 스케줄러에 접근할 수 없음

Context Api
- 부분적인 상태 변경이 어려워 반복적인고 복잡한 업데이트에 비효율적
- Provider 하위의 모든 Consumer들은 Provider 속성이 변경될 때마다 모두 랜더링 됨
```javascript
const App = () => {
  const Context = createContext(0);
  const [state, setState] = useState(0);

  return (
    <Context.Provider value={state}>
      <button onClick={() => setState((prev) => prev + 1)}>+</button>
      <PassingComp />
    </Context.Provider>
  );
}

const PassingComp = () => {
  // state를 사용하지 않지만 state의 상태가 바뀔때마다 리렌더링 됩니다.

  return <DestinationComp />
}

const DestinationComp = () => {
  const state = useContext(Context);

  return <div>{state}</div>
}
```

## Recoil
### 기본 개념
- Context Api를 기반으로 구현 => 업데이트 된 state 부분만 랜더링
- 훅으로 구현되어 함수형 컴포넌트에서만 사용 가능
- 진입 장벽이 매우 낮고, 단순함


1) RecoilRoot <br/>
context api를 기반으로 하기 때문에 context api와 마찬가지로 recoil을 사용할 컴포넌트를 RecoilRoot로 감싸져야한다.

2) atom
- 상태의 단위 (로컬 컴포넌트 상태 대신 사용 가능)
- 업데이트와 구독이 가능
- 업데이트되면 각각의 구독된 컴포넌트는 새로운 값을 반영하여 다시 렌더링
- 동일한 atom이 여러 컴포넌트에서 사용되는 경우 모든 컴포넌트는 상태를 공유
- 전역적으로 고유한 키를 가짐
- 컴포넌트 상태처럼 기본값을 가짐
```javascript
const fontSizeState = atom({
  key: 'fontSizeState',
  default: 14
});
```

3) selector
- atoms나 다른 selector를 입력으로 받아들이는 순수함수
- 다른 atom에 의존하는 동적인 데이터를 만들 수 있게 함
- Redux의 reselect와 MobX의 @computed처럼 동작하는 "get" 함수를 가지고 있음
- atom을 업데이트 할 수 있는 set 함수를 옵션으로 받을 수 있음
```javascript
const fontSizeLabelState = selector({
  key: 'fontSizeLabelState',
  get: ({get}) => {
    const fontSize = get(fontSizeState);
    const unit = 'px';

    return `${fontSize}${unit}`;
  },
});
```
3. useRecoilState
- useState와 유사
- 컴포넌트간에 공유될 수 있는 차이
```javascript
const [fontSize, getFontSize] = useRecoilState(fontSizeState);
```
useRecoilValue, useSetRecoilState etc.

<br/>

### 구현 원리
### Q. 리랜더링이 왜 안발생할까?
```javascript
// Recoil_RecoilRoot.js

const defaultStore = Object.freeze({
  storeID: getNextStoreID(),
  getState: notInAContext,
  replaceState: notInAContext,
  getGraph: notInAContext,
  subscribeToTransactions: notInAContext,
  addTransactionMetadata: notInAContext,
});

const AppContext = React.createContext({ current: defaultStore });
const useStoreRef = () => useContext(AppContext);

const RecoilRoot = () => {
  storeRef = ... // 스토어 생성

  return (
    <AppContext.Provider value={storeRef}>
      {children}
    </AppContext.Provider>
  );
}
```
A. 내려주고 있는 스토어가 React의 state가 아니라 스토어를 ref형태로 내려주고 있어서 리렌더링이 안되고 있는 것
<br/>
<br/>
### Q. atom을 setState 했을 때, atom의 value를 사용하고 있는 컴포넌트들이 어떻게 리렌더링 되는걸까?

Recoil 스토어에서 상태를 가져올 때 사용하는 useRecoilValue훅의 구현
```javascript
// component
const data = useRecoilValue(fontSizeState);

console.log(data); // 14
```

```javascript
// Recoil_Hooks.js

function useRecoilValue(recoilValue) {
  const storeRef = useStoreRef();
  const loadable = useRecoilValueLoadable(recoilValue);

  // React Suspense를 활용하여 비동기 처리해주는 함수
  return handleLoadable(loadable, recoilValue, storeRef); 
}
```

```javascript
// Recoil_Hooks.js

function useRecoilValueLoadable(recoilValue) {
  // 여러가지 reactMode에 대한 useRecoilValueLoadable 훅 구현체
  return {
    CONCURRENT_LEGACY: useRecoilValueLoadable_CONCURRENT_LEGACY,
    SYNC_EXTERNAL_STORE: useRecoilValueLoadable_SYNC_EXTERNAL_STORE,
    MUTABLE_SOURCE: useRecoilValueLoadable_MUTABLE_SOURCE,
    LEGACY: useRecoilValueLoadable_LEGACY,
  }[reactMode().mode](recoilValue);
}
```

```javascript
// Recoil_Hooks.js

function useRecoilValueLoadable_LEGACY(recoilValue) {
  const [, forceUpdate] = useState([]);

  useEffect(() => {
    const subscription = subscribeToRecoilValue(
      _state => {
        const newLoadable = getLoadable();
        forceUpdate(newLoadable);
      },
      ...,
    );  
  }, [...])

  return loadable
}
```
A. useRecoilValue를 사용하는 컴포넌트들은
- subscribeToRecoilValue 함수를 통해 인자로 들어오는 recoilValue(= 우리가 넣어주는 atom/selector)에 대해 subscribe 하게 됨
-  subscribeToRecoilValue에 등록된 콜백이 Recoil 런타임에서 정해진 시점에 호출될 때 useState의 상태가 바뀜에 따라 해당 컴포넌트만 리렌더링 되는 것

<br/>

### Q. subscribeToRecoilValue는 어느 시점에 호출되나요?
```javascript
// Recoil_Hooks.js

function useSetRecoilState(recoilState) {
  const storeRef = useStoreRef();

  return useCallback(
    (newValueOrUpdater) => {
      setRecoilValue(storeRef.current, recoilState, newValueOrUpdater);
    },
    [storeRef, recoilState],
  );
}
```
```javascript
// Recoil_RecoilValueInterface.js

function setRecoilValue(store, recoilValue, valueOrUpdater) {
  queueOrPerformStateUpdate(store, {
    type: 'set',
    recoilValue,
    valueOrUpdater,
  });
}
```
```javascript
// Recoil_RecoilValueInterface.js

function queueOrPerformStateUpdate(store, action) {
  if (batchStack.length) { // recoil의 배치 처리 여부
    const actionsByStore = batchStack[batchStack.length - 1];
    let actions = actionsByStore.get(store);
    if (!actions) {
      actionsByStore.set(store, (actions = []));
    }
    actions.push(action);
  } else {
    applyActionsToStore(store, [action]);
  }
}
```
배치 처리 마지막 단계에도 applyActionsToStore 함수가 똑같이 실행됨

```javascript
// Recoil_RecoilValueInterface.js

function applyActionsToStore(store, actions) {  
  // 상태를 newState로 변경
  store.replaceState(state => {
    const newState = copyTreeState(state);

    for (const action of actions) {
      applyAction(store, newState, action);
    }

    return newState;
  });
}
```
```javascript
// Recoil_RecoilRoot.js

const replaceState = replacer => {
  const nextTree = storeStateRef.current.nextTree
  let replaced;
  
  replaced = replacer(nextTree);

  storeStateRef.current.nextTree = replaced;
  
  if (reactMode().early) { // true
    notifyComponents(storeRef.current, storeStateRef.current, replaced);
  }
  
  notifyBatcherOfChange.current();
};
```
notifyComponents가 호출되면 subscribeToRecoilValue이 호출되고, 결과적으로 useRecoilValue 훅에 있던 forceUpdate가 호출됨에 따라 상태와 연관된 컴포넌트만 리렌더링 되게 됨

## 결론
결과적으로, useRecoilValue 훅을 사용하는 컴포넌트는 자동적으로 Recoil recoilState를 subscribe 하게 되고, 이 subscribe 과정에서 useRecoilValue 훅에 있는 useState의 setState(= forceUpdate)를 넘겨줌

그리고 subscribe 하고 있는 recoilState가 set 되는 시점에 적절하게 forceUpdate 함수가 실행되게되면서 subscribe하고 있는 컴포넌트만 리렌더링이 발생

![recoil-store](https://user-images.githubusercontent.com/42891424/190305499-3bd60d0c-b182-415e-9281-e0e396175a22.png)

