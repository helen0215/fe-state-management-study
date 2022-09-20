# Redux 이해하기

## Redux
- 자바스크립트 상태 관리 라이브러리
- 현재까지도 가장 많이 사용되는 라이브러리

![image](https://user-images.githubusercontent.com/26708382/191023749-17897816-85ab-40f5-84f5-795f0a0ded53.png)
![image](https://user-images.githubusercontent.com/26708382/191023766-04372b96-23d7-4885-8571-6e28b67fe624.png)

<br/><br/><br/><br/><br/><br/>

# Redux 도식화<br/>
![redux](https://user-images.githubusercontent.com/26708382/191023705-f1987be3-e44b-48c9-87d2-70a5d510924a.png)
- State : 저장소에 저정할 상태 값. 하나의 Store 내부 객체 트리에 저장된다.
- Action :  상태를 변경하기 위한 객체. 상태를 변경할 수 있는 유일한 방법.
- Reducer : 액션을 보고 상태를 업데이트한다. (이전 상태와 액션을 인자로 받아서 새로운 상태를 반환한다.)
- Dispatching Function : 액션을 스토어에 전달하기 위해서는 dispatch 함수를 통해서 reducer에 전달한다.
- Store : 애플리케이션의 상태와 리듀서를 관리하는 객체


<br/><br/><br/>

# Redux 예제
```javascript
import { createStore } from 'redux'

// reducer
function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state
  }
}

// store
let store = createStore(counter)

// subscribe
store.subscribe(() => console.log(store.getState())))

// 
store.dispatch({ type: 'INCREMENT' }) // 1
store.dispatch({ type: 'INCREMENT' }) // 2
store.dispatch({ type: 'DECREMENT' }) // 1
```

<br/><br/><br/>

# Redux 분석하기
<br/>

## 3가지 원칙
**1. Single source of truth(SSOT)<br/>**
    → 하나의 스토어에서 관리되는 상태<br/><br/>
**2. Read-only state<br/>**
    → 상태는 읽기만 가능<br/><br/>
**3. Changes from pure functions<br/>**
    → 순수 함수를 통해 상태를 변경할수 있음.<br/><br/>
<br/>
-> 리덕스의 상태는 하나의 스토어에서 관리되며 스토어를 통해 읽기만 할수 있고, 리듀서라는 순수 함수를 통해서만 상태를 변경할 수 있다.<br/>
-> 전역 상태 관리와 디버깅이 편리해진다.<br/>

<br/><br/><br/>

## store
- 내부적으로 애플리케이션의 상태와 리듀서를 관리하는 객체
- API<br/>
getState() : 현재 스토어에 있는 상태를 반환<br/>
subscribe(listener) : 상태 변경시 호출할 함수를 등록<br/>
dispatch(action) : 스토어에 등록한 리듀서에 액션 객체를 전달<br/>
replaceReducer(reducer) : 리듀서 변경<br/><br/>

- createStore
```javascript
export default function createStore(reducer, initialState, enhancer) {
  var currentReducer = reducer
  var currentState = initialState
  var currentListeners = []
  var nextListeners = currentListeners
  var isDispatching = false


  function getState() {
    return currentState
  }


  function subscribe(listener) {
   nextListeners.push(listener)

    return function unsubscribe() {
      var index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
    }
  }


  function dispatch(action) {
    // action 유효성 검증
    if (!isPlainObject(action)) {
      throw new Error()
    }
    if (typeof action.type === 'undefined') {
      throw new Error()
    }
    if (isDispatching) {
      throw new Error()
    }

    // 리듀서에서 반환하는 상태를 반영
    try {
      isDispatching = true
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }

    
    // 등록된 리스너를 순서대로 실행
    var listeners = currentListeners = nextListeners
    for (var i = 0; i < listeners.length; i++) {
      listeners[i]()
    }

    return action
  }


  function replaceReducer(nextReducer) {
    currentReducer = nextReducer
    dispatch({ type: ActionTypes.INIT })
  }

  dispatch({ type: ActionTypes.INIT })

  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer
  }
}
```

- enhancer : 고차함수
- applyMiddleware 제공

<br/><br/><br/>

## middleware
- 미들웨어를 통해 dispatch 과정에서 액션이 리듀서에 도달하기 전에 특정 작업을 미리 처리할 수 있다.<br/>
- 액션 검증, 액션 필터링, api 연동, 비동기 처리 등을 처리<br/>
- 특정 작업을 액션이랑 분리하기 때문에 복잡성, 중복 등 유지보수 측면에서 장점<br/><br/>
- Compose
```javascript
compose(f, g, h) = (...args) => f(g(h(...args)))
const compose = (...functions) => (initialVal) 
	=> functions.reduceRight((val, fn) => fn(val), initialVal);
```

<br/>

- middleware
```javascript
// 3중 중첩 함수
// 기본 디스패치 함수(baseDispatch)를 래핑
function middleware({getState, dispatch}}) {
  return function wrapDispatch(next) {
    return function handleAction(action) {
      // do something
      return next(action);
    }
  }
}

const anotherExampleMiddleware = ({getState, dispatch}) => next => action => {
  // Do something in here, when each action is dispatched

  return next(action)
}
```

<br/>

- applyMiddleware
```javascript

// createStore 자체를 감싸는 고차 함수
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, initialState, enhancer) => {
    var store = createStore(reducer, initialState, enhancer)
    var dispatch = store.dispatch
    var chain = []

    // 미들웨어들에게 인자로 전달되는 객체
    // 비동기 작업을 위해 이전 미들웨어 체인으로 돌아가야할 케이스가 있음. 클로저 변수로 전체 미들웨어가 연결된 dispatch가 필요하다.
    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }
    
    /*
    이전 미들웨어 체인을 참조 할 수 없음
    const middlewareAPI = {
        getState: store.getState,
        dispatch
    }
    */
    
    // 미들웨어가 반환하는 체인 함수(wrapDispatch)
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    
    // 체인 함수들을 결합해서 새로운 dispatch 함수
    dispatch = compose(...chain)(store.dispatch)

    //  파라미터로 받았던 기존 스토어와 동일한 Store api와 새로 만들어진 dispatch 함수를 반환한다.
    return {
      ...store,
      dispatch
    }
  }
}
```

<br/><br/><br/>

## reducer
- 이전 상태와 액션을 받아 새로운 상태를 반환하는 순수함수로 정의
- 복잡한 애플리케이션의 모든 상태를 전부 하나의 리듀서로 관리하기는 어려움
- 작은 단위의 리듀서를 만들고 마지막에 combineReducers API를 통해 리듀서를 조합

- combineReducers
```javascript
export default function combineReducers(reducers) {
  var reducerKeys = Object.keys(reducers)
  var finalReducers = {}
  for (var i = 0; i < reducerKeys.length; i++) {
    var key = reducerKeys[i]
    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  var finalReducerKeys = Object.keys(finalReducers)

  var sanityError
  try {
    assertReducerSanity(finalReducers)
  } catch (e) {
    sanityError = e
  }

  return function combination(state = {}, action) {
    if (sanityError) {
      throw sanityError
    }

    if (process.env.NODE_ENV !== 'production') {
      var warningMessage = getUnexpectedStateShapeWarningMessage(state, finalReducers, action)
      if (warningMessage) {
        warning(warningMessage)
      }
    }

    var hasChanged = false
    var nextState = {}
    for (var i = 0; i < finalReducerKeys.length; i++) {
      var key = finalReducerKeys[i]
      var reducer = finalReducers[key]
      var previousStateForKey = state[key]
      var nextStateForKey = reducer(previousStateForKey, action)
      if (typeof nextStateForKey === 'undefined') {
        var errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      nextState[key] = nextStateForKey
      // 상태 변화가 있는 경우에만 새로운 참조를 가진 객체를 반환
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    return hasChanged ? nextState : state
  }
}
```
