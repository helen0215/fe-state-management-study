> [전체 코드](https://github.com/Apress/frameworkless-front-end-development/tree/master/Chapter07)

# Controller 코드
> 컨트롤러에서 전역 상태를 관리한다. 
1. registry에 렌더링을 위해 컴포넌트 View를 등록합니다.
2. View와 registry.renderRoot는 새로운 Element 를 리턴합니다.
3. 실제 렌더링은 render 함수 내 applyDiff 함수를 호출하여  그려진 Element와 새로 그려질 Element 를 비교하여 변경되었다면 Root Element에 반영합니다.
4. View 들은 그려질때 Controller에 있는 Events를 받아 사용자 인터렉션이 일어났을때 해당 event를 호출합니다.
5. 이벤트 발생 시 전역 상태를 변경하고 render 함수를 호출해 View를 업데이트 합니다.

[index.js](./src/00.%20base/index.js)
<pre>
...
<b>const state = {
  todos: [],
  currentFilter: 'All'
}</b>

const events = {
  addItem: text => {
    <b>state.todos.push({
      text,
      completed: false
    })</b>
    render()
  },
  ...
}

const render = () => {
  window.requestAnimationFrame(() => {
    const main = document.querySelector('#root')

    const newMain = registry.renderRoot(
      main,
      <b>state,</b>
      events)

    applyDiff(document.body, main, newMain)
  })
}
...
</pre>

# 모델-뷰-컨트롤러
> 상태를 분리된 외부 모델에서 관리한다. <br/>
상태 변경은 모델에서 제공하는 메소드를 통해 변경할 수 있다.
렌더링을 위한 현재 상태를 모델에게 요청한다.
모델에서 getState로 상태를 재공할 때 상태를 깊은 복사한 후 Object.freeze로 제공한다.

[index.js](./src/01.%20model/index.js)
<pre>
...
<b>import modelFactory from './model.js'</b>
...

<b>const model = modelFactory()</b>

const events = {
  addItem: text => {
    model.addItem(text)
    render(<b>model.getState()</b>)
  },
  ...
}

<b>const render = (state) => {</b>
  window.requestAnimationFrame(() => {
    ...
  })
}

<b>render(model.getState())</b>
</pre>

[model.js](./src/01.%20model/model.js)
<pre>
const cloneDeep = x => {
  return <b>JSON.parse(JSON.stringify(x))</b>
}

...

export default (initalState = INITIAL_STATE) => {
  const state = cloneDeep(initalState)

  <b>const getState = () => {
    return Object.freeze(cloneDeep(state))
  }</b>

  const addItem = text => {
    if (!text) {
      return
    }

    state.todos.push({
      text,
      completed: false
    })
  }
  ...

  return {
    addItem,
    getState,
    ...
  }
}
</pre>


<img width="583" alt="image" src="https://user-images.githubusercontent.com/60346043/186093210-e22f9bdb-910f-4790-b897-19bb1f11017b.png">


## 옵저버블 모델
> 이벤트 발생시 최첨단 수동 render() 메소드 호출 -> 누군가 render 메소드 호출을 하지 않으면 오류가 발생하기 쉽다. 이를 observer pattern을 기반으로 개선해 보자.

[model.js](./src/02.%20observable/model.js)
<pre>
...
const INITIAL_STATE = {
  todos: [],
  currentFilter: 'All'
}

export default (initalState = INITIAL_STATE) => {
  const state = cloneDeep(initalState)
  <b>let listeners = []</b>

  <b>const addChangeListener = listener => {
    listeners.push(listener)

    listener(freeze(state))

    return () => {
      listeners = listeners.filter(l => l !== listener)
    }
  }</b>

  <b>const invokeListeners = () => {
    const data = freeze(state)
    listeners.forEach(l => l(data))
  }</b>

  const addItem = text => {
    if (!text) {
      return
    }

    state.todos.push({
      text,
      completed: false
    })

    <b>invokeListeners()</b>
  }

  ...

  return {
    ...
    addItem,
    <b>addChangeListener</b>
  }
}
</pre>

테스트 코드를 통해 바뀐 부분에 대해서 좀 더 살펴보자. [model.test.js](./src/02.%20observable/model.test.js)
> Model 객체를 통해 상태를 얻는 방법은 리스터 콜백 추가라는 것을 알 수 있다.


컨트롤러 코드가 더 간단해 진 것을 볼 수 있다 [index.js](./src/02.%20observable/index.js)
<pre>
...
const {
  <b>addChangeListener,
  ...events</b>
} = model

const render = (state) => {
  window.requestAnimationFrame(() => {
    const main = document.querySelector('#root')

    const newMain = registry.renderRoot(
      main,
      state,
      events)

    applyDiff(document.body, main, newMain)
  })
}

<b>addChangeListener(render)</b>

</pre>
옵저버블 모델은 모델의 공개 인터페이스를 수정하지 않고 addChangeListener를 통해 상태가 변경되었을때 새로운 기능을 추가할 수 있다.


# 반응형 프로그래밍
> 반응형 패러다임은 애플리케이션이 모델 변경, HTTP 요청, 사용자 동작, 탐색 등과 같은 이벤트를 방출할 수 있는 옵저버블로 동작하도록 구현하는 것을 의미한다. 

## 반응형 모델
옵저버블을 생성할 수 있는 쉬운 방법이 필요하다. 이를 통해 도메인 로직에만 집중하고 이벤트 방출은 옵저버블 라이브러리로 떠넘겨 보자.

옵저버블 팩토리를 기반으로 하는 모델<br/>
더 이상  상태가 변할때 invokeListeners를 호출하지 않는다.<br/>

[model.js](./src/03.%20reactive/1.%20reactive%20model/model.js)
<pre>
<b>import observableFactory from './observable.js'</b>

...

export default (initalState = INITIAL_STATE) => {
  const state = cloneDeep(initalState)

  const addItem = text => {
    if (!text) {
      return
    }

    state.todos.push({
      text,
      completed: false
    })
  }
  ...

  const model = {
    addItem,
    updateItem,
    deleteItem,
    toggleItemCompleted,
    completeAll,
    clearCompleted,
    changeFilter
  }

  <b>return observableFactory(model, () => state)</b>
}
</pre>

옵저버블 팩토리 [obserbable.js](./src/03.%20reactive/1.%20reactive%20model/observable.js)
<pre>
const cloneDeep = x => {
  return JSON.parse(JSON.stringify(x))
}

const freeze = state => Object.freeze(cloneDeep(state))

export default (model, <b>stateGetter</b>) => {
  let listeners = []

  const addChangeListener = cb => {
    listeners.push(cb)
    cb(freeze(<b>stateGetter()</b>))
    return () => {
      listeners = listeners
        .filter(element => element !== cb)
    }
  }

  const invokeListeners = () => {
    const data = freeze(<b>stateGetter()</b>)
    listeners.forEach(l => l(data))
  }

  const wrapAction = originalAction => {
    return <b>(...args) => {
      const value = originalAction(...args)
      invokeListeners()
      return value
    }</b>
  }

  <b>const baseProxy = {
    addChangeListener
  }</b>

  return Object
    .keys(model)
    .filter(key => {
      return typeof model[key] === 'function'
    })
    .reduce((proxy, key) => {
      const action = model[key]
      return {
        ...proxy,
        [key]: wrapAction(action)
      }
    }, baseProxy)
}
</pre>
Model 객체 메소드와 동일한 이름의 프록시 메소드를 만들고 모델의 메서드가 호출될때 이 프록시 메소드가 호출되게 된다. <br/>
프록시 메서드에서는 모델의 메서드를 호출하고 모델에서 제공하는 Getter 함수를 통해 상태를 가져온 다음 등록된 리스너들을 호출한다. 

## 네이티브 프록시
> 자바스크립트 Proxy 객체를 통해 Proxy를 생성한다. 이는 앞서 구현한 옵저버블 팩토리에서 모델의 메서드를 래핑한것 보다 훨신 쉽게 래핑할 수 있다.

기본 프록시 객체 사용법
<pre>
const base = {
  foo: 'bar'
}

const handler = {
  get: (target, name) => {
    console.log(`Getting ${name}`)
    return target[name]
  },
  set: (target, name, value) => {
    console.log(`Setting ${name} to ${value}`)
    target[name] = value
    <b>return true</b>
  }
}

const proxy = new Proxy(base, handler)

proxy.foo = 'baz'
console.log(`Logging ${proxy.foo}`)
</pre>

프록시 옵저버블 팩토리 [observable.js](./src/03.%20reactive/2.%20native%20proxy/observable.js)
```javascript
const cloneDeep = x => {
  return JSON.parse(JSON.stringify(x))
}

const freeze = state => Object.freeze(cloneDeep(state))

export default (initialState) => {
  let listeners = []

  const proxy = new Proxy(cloneDeep(initialState), {
    set: (target, name, value) => {
      target[name] = value
      listeners.forEach(l => l(freeze(proxy)))
      return true
    }
  })

  proxy.addChangeListener = cb => {
    listeners.push(cb)
    cb(freeze(proxy))
    return () => {
      listeners = listeners.filter(l => l !== cb)
    }
  }

  return proxy
}
```

프록시 옵저버블 팩토리를 사용하는 [model.js](./src/03.%20reactive/2.%20native%20proxy/model.js)
<pre>
...

export default (initialState = INITIAL_STATE) => {
  const state = observableFactory(initialState)

  const addItem = text => {
    if (!text) {
      return
    }

    <b>state.todos = [...state.todos, {
      text,
      completed: false
    }]</b>
  }

  const updateItem = (index, text) => {
    if (!text) {
      return
    }

    if (index < 0) {
      return
    }

    if (!state.todos[index]) {
      return
    }

    <b>state.todos = state.todos.map((todo, i) => {
      if (i === index) {
        todo.text = text
      }
      return todo
    })</b>
  }

  ...

  return {
    addChangeListener: state.addChangeListener,
    addItem,
    updateItem,
    ...
  }
}
</pre>
반응형 모델에서는 todos 배열의 push를 호출하거나 요소를 대체했지만 프록시 기반에서는 Proxy 객체 사용시 set 트랩을 호출하려면 속성을 덮어써야 하기때문에 todos 배열을 매번 덮어쓰는 형태로 변경되었다.

# 이벤트 버스
> 이벤트 버스 패턴을 사용해 애플리케이션의 상태를 관리하는 방법을 알아본다. 이벤트 버스는 이벤트 주도 아키텍쳐(EDA, Event-Driven Architecture)를 구현하는 하나의 방법이다. <br/>
이벤트 버스 패턴의 기본 개념은 애플리케이션을 구성하는 '노드'들을 연결하는 단일 객체가 모든 이벤트를 처리한다는 것이다. 이벤트가 발생하고 이벤트 버스로 전달되면 이벤트 버스는 이를 처리한 후 처리 결과를 모든 노드로 전송한다. 

이벤트는 이벤트 처리를 위한 이벤트 타입과 정보를 담고있는 페이로드로 정의된다.
```javascript
const event = {
  type: 'ITEM_ADDED',
  payload: 'Buy Milk'
}
```

모델에서 이전상태와 이벤트를 받아 새로운 상태를 변환하도록 구현한다.

<img width="455" alt="image" src="https://user-images.githubusercontent.com/60346043/186335797-3eb8cb37-7b10-4dc4-9edd-61380a926900.png">

<img width="494" alt="image" src="https://user-images.githubusercontent.com/60346043/186336458-c8310c8e-2223-4810-be14-9c53bf8eb5e6.png">

이벤트 버스 구현 [eventBus.js](./src/04.%20event%20bus/eventBus.js)
<pre>
const cloneDeep = x => {
  return JSON.parse(JSON.stringify(x))
}

const freeze = state => Object.freeze(cloneDeep(state))

export default (model) => {
  let listeners = []
  <b>let state = model()</b>

  const subscribe = listener => {
    listeners.push(listener)

    return () => {
      listeners = listeners.filter(l => l !== listener)
    }
  }

  const invokeSubscribers = () => {
    const data = freeze(state)
    listeners.forEach(l => l(data))
  }

  const dispatch = event => {
    <b>const newState = model(state, event)</b>

    if (!newState) {
      throw new Error('model should always return a value')
    }

    if (newState === state) {
      return
    }

    <b>state = newState</b>

    invokeSubscribers()
  }
  return {
    subscribe,
    dispatch,
    getState: () => freeze(state)
  }
}
</pre>

모델은 입력으로 이전 상태와 이벤트를 받아 새로운 상태를 반환한다.<br/>
모델의 또달은 중요한 특성은 '순수 함수' 라는 것이다.<br/>
내부 동작을 잘 이해하기 위해 테스트 스위트도 참고해보자. [test](./src/04.%20event%20bus/eventBus.test.js)

이벤트 버스 아키텍쳐를 적용한 모델 [model.js](./src/04.%20event%20bus/model.js)
<pre>
const cloneDeep = x => {
  return JSON.parse(JSON.stringify(x))
}

const INITIAL_STATE = {
  todos: [],
  currentFilter: 'All'
}

<b>const addItem = (state, event) => {
  const text = event.payload
  if (!text) {
    return state
  }

  return {
    ...state,
    todos: [...state.todos, {
      text,
      completed: false
    }]
  }
}</b>

const updateItem = (state, event) => {
  const { text, index } = event.payload
  if (!text) {
    return state
  }

  if (index < 0) {
    return state
  }

  if (!state.todos[index]) {
    return state
  }

  return {
    ...state,
    todos: state.todos.map((todo, i) => {
      if (i === index) {
        todo.text = text
      }
      return todo
    })
  }
}

...

<b>const methods = {
  ITEM_ADDED: addItem,
  ITEM_UPDATED: updateItem,
  ...
}</b>

export default (initalState = INITIAL_STATE) => {
  return (prevState, event) => {
    if (!prevState) {
      return cloneDeep(initalState)
    }

    const currentModifier = methods[event.type]

    <b>if (!currentModifier) {
      return prevState
    }</b>

    return currentModifier(prevState, event)
  }
}
</pre>

모델을 작은 서브모듈로 분리 [model.js](./src/04.%20event%20bus/model_submodule.js)
> 모델은 두개의 서브 모듈을 가지며 첫번째 서브모델은 todo를 관리하고 두번째 서브모듈은 필터를 관리한다. 메인 모델 함수는 서브모델의 결과를 하나의 state로 병합한다.
<pre>
<b>
import todosModifers from './todos.js'
import filterModifers from './filter.js'
</b>

const cloneDeep = x => {
  return JSON.parse(JSON.stringify(x))
}

const INITIAL_STATE = {
  todos: [],
  currentFilter: 'All'
}

export default (initalState = INITIAL_STATE) => {
  return (prevState, event) => {
    if (!event) {
      return cloneDeep(initalState)
    }

    const {
      todos,
      currentFilter
    } = prevState

    const newTodos = todosModifers(todos, event)
    const newCurrentFilter = filterModifers(currentFilter, event)

    if (newTodos === todos && newCurrentFilter === currentFilter) {
      return prevState
    }

    return {
      todos: newTodos,
      currentFilter: newCurrentFilter
    }
  }
}
</pre>

이벤트 버스를 기반으로 하는 컨트롤러 [index.js](./src/04.%20event%20bus/index.js)
<pre>
...

const modifiers = modelFactory()
<b>const eventBus = eventBusFactory(modifiers)</b>

const render = (state) => {
  window.requestAnimationFrame(() => {
    const main = document.querySelector('#root')

    const newMain = registry.renderRoot(
      main,
      state,
      <b>eventBus.dispatch</b>)

    applyDiff(document.body, main, newMain)
  })
}

eventBus.subscribe(render)

render(eventBus.getState())
</pre>
이전과 다르게 렌더링 함수에 이벤트를 제공하지 않고 dispatch 메서드를 사용한다.

뷰에서는 이벤트 버스의 dispatch로 이벤트를 보낸다.
<pre>
// eventCreators
const EVENT_TYPES = Object.freeze({
  ITEM_ADDED: 'ITEM_ADDED',
  ITEM_UPDATED: 'ITEM_UPDATED',
  ..
})

function eventCreators () {
  addItem: text => ({
    type: EVENT_TYPES.ITEM_ADDED,
    payload: text
  }),
  updateItem: (index, text) => ({
    type: EVENT_TYPES.ITEM_UPDATED,
    payload: {
      text,
      index
    }
  }),
  ...
}


// view

let template
...

const getTemplate = () => {
  if (!template) {
    template = document.getElementById('todo-app')
  }

  return template
    .content
    .firstElementChild
    .cloneNode(true)
}

const addEvents = (targetElement, dispatch) => {
  targetElement
    .querySelector('.new-todo')
    .addEventListener('keypress', e => {
      if (e.key === 'Enter') {
        <b>dispatch(eventCreators.addItem(e.target.value))</b>
        e.target.value = ''
      }
    })
  ...
}
...
</pre>
여기까지 하고나서 개인적으로는 Redux와 굉장히 비슷하다는걸 느꼈다.

# 비교
|             | 단순성        | 일관성       | 확장성        |
| ----------- | ----------- | ----------- | ----------- |
| MVC         | O           | X           | X           |
| 반응형        | X           | O           | -           |
| 이벤트 버스    | -           | X           | O           |

## MVC
MVC는 구현하기 단순하며 관심사를 분리할 수 있다.<br/>
다만 MVC 엄걱한 패턴이 아니다. 그렇다 보보니 같은 MVC 패턴을 적용했더라도 약간씩 다른 버전의 MVC가 구현된 것을 쉽게 찾아볼 수 있다. <br/> 
MVC로 효과적으로 작업하기 위해서는 팀의 MVC 규칙을 정의해야하며 확장할 때마다 문제가 발생할 수 있고 애플리케이션이 커질수록 일관성이 없어져 코드를 읽기 힘들어질 수도 있다.


## 반응형 프로그래밍
반응형 프로그래밍의 기본 아이디어는 애플리케이션이 옵저버블 하다는 것이다.<br/> 
상태를 관리해야하는 이벤트가 발생할 때 동일한 객체로 작업하기 때문에 뛰어난 일관성을 보장해준다.<br/>
하지만 모든 옵저버블을 래핑해야하고 제공하는 추상화를 변경해야 하는 등 문제가 생겼을때 확장성에 큰 문제가 될 수 있다.

## 이벤트 버스
이벤트 버스는 "모든 상태 변경은 이벤트에 의해 생성된다."는 엄격한 규칙을 기반으로 한다.<br/>
엄격한 규칙 덕분에 다른 아키텍처에 비해 애플리케이션의 복잡성이 크기에 비례하도록 유지할 수 있다.<br/>
이벤트 버스의 가장 큰 문제점은 다변성(verbosicy)이다.<br/>
모든 상태 업데이트에 대해 이벤트를 생성하고, 버스를 통해 이벤트를 발생하고, 상태를 업데이트한 모델을 작성한 다음 새 상태를 리스너에 전송해야 한다. 이 패턴의 다변성 때문에 애플리케이션의 모든 상태를 이 패턴으로 관리하지 않고 더 작거나 간단한 도메인 관리를 위해 개발자는 다른 상태관리 전략을 함께 사용할 수 있기때문에 결과적으로 일관성이 떨어지게된다.

# 결론
~애플리케이션이 커지면 일관성과 복잡성은 증가하고 확장성, 단순성은 떨어진다~<br/>
우리가 사용하고자 하는 상태 관리 전략을 결정하기 위해 도메인/관심사의 분리, 도메인/관리해야하는 상태에 대한 정의와 관리 방법을 명확히 할 필요가 있다.<br/>
예를들어 모든 상태가 반응형일 필요가 있는지, 또는 전역으로 상태변화 이벤트 전파를 위해 어떤 방법을 사용할 것인지(이벤트 버스) 라는 고민을 해볼 수 있을 것 같다.
