# 목차
1. [기본 사용법](#기본-사용법)
2. [내부 구조](#내부-구조)
3. [반응성](#반응성)

# 기본 사용법

## ****Store 정의****

1. Option Stores (Option API 와 유사한 방법)

```tsx
// https://pinia.vuejs.org/core-concepts/
export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0, name: 'Eduardo' }),
  getters: {
    doubleCount: (state) => state.count * 2,
  },
  actions: {
    increment() {
      this.count++
    },
  },
})
```

1. Setup Stores (Composition API 와 유사한 방법)

```tsx
// https://pinia.vuejs.org/core-concepts/
export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  const name = ref('Eduardo')
  const doubleCount = computed(() => count.value * 2)
  function increment() {
    count.value++
  }

  return { count, name, doubleCount, increment }
})
```

- `ref()` ⇒ `state`
- `computed()` ⇒ `getters`
- `function()` ⇒ `actions`
- 1번 방법보다 더 유연하고, store 내부에 watchers 를 만들 수도 있다.

## Store 사용

- setup() 내부에서 useXxxStore() 를 호출했을 때 비로소 Store 가 생성된다. ( setup() 을 사용하지 않더라도 pinia 를 사용할 수 있는 방법을 제공하고 있다. )
- 원하는 만큼 Store 를 각각 다른 파일에 정의할 수 있다.
- 일단 Store 가 생성되고 나면, state, getters, actions 에 접근할 수 있다.
- Store 는 reactive 로 감싸진 객체이다. ( .value 로 접근할 필요 없고, props 처럼 쓰면 된다. props 와 마찬가지로 구조분해할당하면 state 의 반응성을 잃게 된다. )
- 구조분해할당을 하려면 storeToRefs() 를 사용하면 된다.

```tsx
// https://pinia.vuejs.org/core-concepts/
import { storeToRefs } from 'pinia'

export default defineComponent({
  setup() {
    const store = useCounterStore()
    // `name` and `doubleCount` are reactive refs
    // This will also create refs for properties added by plugins
    // but skip any action or non reactive (non ref/reactive) property
    const { name, doubleCount } = storeToRefs(store)
    // the increment action can be just extracted
    const { increment } = store

    return {
      name,
      doubleCount,
      increment,
    }
  },
})
```

# 내부 구조

## Pinia

Pinia 를 사용하기 위해서 createPinia() 를 통해 rootStore 를 생성한다.

```tsx
// https://github.com/vuejs/pinia/blob/v2/packages/pinia/src/rootStore.ts
export interface Pinia {
  install: (app: App) => void

  /**
   * root state
   */
  state: Ref<Record<string, StateTree>>

  /**
   * Effect scope the pinia is attached to
   *
   * @internal
   */
  _e: EffectScope

  /**
   * Registry of stores used by this pinia.
   *
   * @internal
   */
  _s: Map<string, StoreGeneric>

  ...
}
```

## defineStore

defineStore 를 실행했을 때, id 값으로 생성된 Store 가 있다면 Pinia rootStore 에서 가져오고, 아니라면 내부적으로 위의 ‘Store 정의' 에서 1번, 2번 방법에 맞게 createXxxStore 함수를 호출한다.

```tsx
// https://github.com/vuejs/pinia/blob/v2/packages/pinia/src/store.ts
export function defineStore(
  // TODO: add proper types from above
  idOrOptions: any,
  setup?: any,
  setupOptions?: any
): StoreDefinition {

  if (!pinia._s.has(id)) {
    // creating the store registers it in `pinia._s`
    if (isSetupStore) {
      createSetupStore(id, setup, options, pinia)
    } else {
      createOptionsStore(id, options as any, pinia)
    }
  }

  const store: StoreGeneric = pinia._s.get(id)!

  return store as any
}
```

## createOptionStore

‘OptionStore’ 방법으로 Store 를 생성하게 되면 아래 로직을 타게 된다.

```tsx
function createOptionsStore<
  Id extends string,
  S extends StateTree,
  G extends _GettersTree<S>,
  A extends _ActionsTree
>(
  id: Id,
  options: DefineStoreOptions<Id, S, G, A>,
  pinia: Pinia,
  hot?: boolean
): Store<Id, S, G, A> {

  const { state, actions, getters } = options

  const initialState: StateTree | undefined = pinia.state.value[id]

  let store: Store<Id, S, G, A>

	// 'Store 정의' 2번 방법의 2번째 파라미터 함수와 같은 꼴을 만들어준다.
  function setup() {
    if (!initialState && (!__DEV__ || !hot)) {
      /* istanbul ignore if */
      if (isVue2) {
        set(pinia.state.value, id, state ? state() : {})
      } else {
        pinia.state.value[id] = state ? state() : {}
      }
    }

    // avoid creating a state in pinia.state.value
    const localState =
      __DEV__ && hot
        ? // use ref() to unwrap refs inside state TODO: check if this is still necessary
          toRefs(ref(state ? state() : {}).value)
        : toRefs(pinia.state.value[id])

    return assign(
      localState,
      actions,
      Object.keys(getters || {}).reduce((computedGetters, name) => {
        if (__DEV__ && name in localState) {
          console.warn(
            `[🍍]: A getter cannot have the same name as another state property. Rename one of them. Found with "${name}" in store "${id}".`
          )
        }

        computedGetters[name] = markRaw(
          computed(() => {
            setActivePinia(pinia)
            // it was created just before
            const store = pinia._s.get(id)!

            // allow cross using stores
            /* istanbul ignore next */
            if (isVue2 && !store._r) return

            // @ts-expect-error
            // return getters![name].call(context, context)
            // TODO: avoid reading the getter while assigning with a global variable
            return getters![name].call(store, store)
          })
        )
        return computedGetters
      }, {} as Record<string, ComputedRef>)
    )
  }

  store = createSetupStore(id, setup, options, pinia, hot, true)

  store.$reset = function $reset() {
    const newState = state ? state() : {}
    // we use a patch to group all changes into one single subscription
    this.$patch(($state) => {
      assign($state, newState)
    })
  }

  return store as any
}
```

## createSetupStore

SetupStore 방법으로 Store 를 생성하게 되었을 때 아래 로직을 타게 되고, OptionStore 방법을 거쳐온 경우도 아래의 로직을 타게 된다.

해당 함수에서 pinia plugin 이나 subscription 등의 옵션들을 처리하게 된다.

```tsx
function createSetupStore<
  Id extends string,
  SS,
  S extends StateTree,
  G extends Record<string, _Method>,
  A extends _ActionsTree
>(
  $id: Id,
  setup: () => SS,
  options:
    | DefineSetupStoreOptions<Id, S, G, A>
    | DefineStoreOptions<Id, S, G, A> = {},
  pinia: Pinia,
  hot?: boolean,
  isOptionsStore?: boolean
): Store<Id, S, G, A> {
  ...

  // Store 를 reactive 로 감싸준다.
  const store: Store<Id, S, G, A> = reactive(
    assign(
      __DEV__ && IS_CLIENT
        ? // devtools custom properties
          {
            _customProperties: markRaw(new Set<string>()),
            _hmrPayload,
          }
        : {},
      partialStore
      // must be added later
      // setupStore
    )
  ) as unknown as Store<Id, S, G, A>

  // rootStore 에 해당 Store 를 저장한다. 
  pinia._s.set($id, store)

  const setupStore = pinia._e.run(() => {
    // effectScope 생성하여 SetupStore 의 2번째 인자 혹은 OptionStore 에서 넘겨준 setup 함수를 실행한다.
    scope = effectScope()
    return scope.run(() => setup())
  })!

  ...

  // add the state, getters, and action properties
  if (isVue2) {
    Object.keys(setupStore).forEach((key) => {
      set(
        store,
        key,
        // @ts-expect-error: valid key indexing
        setupStore[key]
      )
    })
  } else {
    assign(store, setupStore)
    // allows retrieving reactive objects with `storeToRefs()`. Must be called after assigning to the reactive object.
    // Make `storeToRefs()` work with `reactive()` #799
    assign(toRaw(store), setupStore)
  }
}
```

# 반응성

## reactive

reactive(object) 를 실행하는 것은 observe(object) 를 실행하는 것과 같다.

observe() 를 실행하면 Vue.observable 을 통해 반응형 객체를 반환한다.

```tsx
// https://github.com/vuejs/composition-api/blob/main/src/reactivity/reactive.ts
export function reactive<T extends object>(obj: T): UnwrapRef<T> {
  if (!isObject(obj)) {
    if (__DEV__) {
      warn('"reactive()" must be called on an object.')
    }
    return obj
  }

  if (
    !(isPlainObject(obj) || isArray(obj)) ||
    isRaw(obj) ||
    !Object.isExtensible(obj)
  ) {
    return obj as any
  }

  // Vue.observable 
  // https://v2.vuejs.org/v2/api/#Vue-observable
  // https://github.com/vuejs/vue/tree/main/src/core/observer
  const observed = observe(obj)
  // reactive property 에 접근했을 때, ref 와 다르게 .value 를 사용하지 않아도 값을 사용할 수 있도록 한다.
  setupAccessControl(observed)
  return observed as UnwrapRef<T>
}
```

아래의 코드는 Vue.observable 을 통해 반응형 객체를 반환하는 코드는 [https://github.com/vuejs/vue/blob/main/src/core/observer/index.ts#L131](https://github.com/vuejs/vue/blob/main/src/core/observer/index.ts#L131) 를 참고하면 된다.

## ref

reactive 객체를 proxy 함수로 감싸서 반환한다.

```tsx
// https://github.com/vuejs/composition-api/blob/main/src/utils/utils.ts
export function proxy(
  target: any,
  key: string,
  { get, set }: { get?: Function; set?: Function }
) {
  Object.defineProperty(target, key, {
    enumerable: true,
    configurable: true,
    get: get || noopFn,
    set: set || noopFn,
  })
}

// https://github.com/vuejs/composition-api/blob/main/src/reactivity/ref.ts
export class RefImpl<T> implements Ref<T> {
  readonly [_refBrand]!: true
  public value!: T
  constructor({ get, set }: RefOption<T>) {
    proxy(this, 'value', {
      get,
      set,
    })
  }
}

export function createRef<T>(
  options: RefOption<T>,
  isReadonly = false,
  isComputed = false
): RefImpl<T> {
  // reactive 객체로부터 값을 꺼내오고 설정하는 options 객체를 proxy 로 감싸준다.
  const r = new RefImpl<T>(options)

  // add effect to differentiate refs from computed
  if (isComputed) (r as ComputedRef<T>).effect = true

  // ref 객체에 .value 이외의 property 를 만들 수 없다. 
  const sealed = Object.seal(r)

  if (isReadonly) readonlySet.set(sealed, true)

  return sealed
}

export function ref(raw?: unknown) {
  if (isRef(raw)) {
    return raw
  }

  const value = reactive({ [RefKey]: raw })
  return createRef({
    get: () => value[RefKey] as any,
    set: (v) => ((value[RefKey] as any) = v),
  })
}
```

## computed

watcher 를 통해 반응성이 부여된 getter 와 setter (사용자가 정의한 경우) 를 createRef 함수로 감싸 computed 객체를 생성한다.

```tsx
// https://github.com/vuejs/composition-api/blob/main/src/apis/computed.ts
export function computed<T>(
  getterOrOptions: ComputedGetter<T> | WritableComputedOptions<T>
): ComputedRef<T> | WritableComputedRef<T> {

  const vm = getCurrentScopeVM()
  let getter: ComputedGetter<T>
  let setter: ComputedSetter<T> | undefined

  if (isFunction(getterOrOptions)) {
    getter = getterOrOptions
  } else {
    getter = getterOrOptions.get
    setter = getterOrOptions.set
  }

  let computedSetter
  let computedGetter

  if (vm && !vm.$isServer) {
    const { Watcher, Dep } = getVueInternalClasses()
    let watcher: any
    computedGetter = () => {
      if (!watcher) {
        watcher = new Watcher(vm, getter, noopFn, { lazy: true })
      }
      if (watcher.dirty) {
        watcher.evaluate()
      }
      if (Dep.target) {
        watcher.depend()
      }
      return watcher.value
    }

    computedSetter = (v: T) => {
      if (__DEV__ && !setter) {
        warn('Write operation failed: computed value is readonly.', vm!)
        return
      }

      if (setter) {
        setter(v)
      }
    }
  } else {
    // fallback
    ...
    })

  return createRef<T>(
    {
      get: computedGetter,
      set: computedSetter,
    },
    !setter,
    true
  ) as WritableComputedRef<T> | ComputedRef<T>
}
```
