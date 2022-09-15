# SWR이란

💡 stale: 신선하지 않은, revalidate: 재검증
<br />
<br />

SWR이 뭔지 알아보기 앞서 캐시에 대해 간단하게 짚고 넘어가면, **캐시**는 빠른 응답 또는 비용을 절약하기 위해 사용하는 일종의 임시 저장 데이터입니다. 

간단하게 무언가를 찾고자할 때 캐시에 없으면 `Cache Miss` 로 표현한 뒤 **cache** 에 저장해두고, 그 후에 같은 걸 찾고자할 때 캐시에 있으면 `Cache Hit` 으로 표현한 뒤 그대로 가져다 사용할 수 있어 속도 비용의 이점을 가지고 있습니다.

이 과정에서 **Stale Cache Handling** 이라는 개념이 나옵니다.

캐시는 빠른 응답을 위한 임시 데이터이기 때문에 원래의 데이터에 종속적입니다. 때문에 원본 데이터가 바뀌게 되면 이전에 생성된 캐시는 더 이상 쓸모가 없어집니다.

오히려 오래된 캐시가 남아있으면 잘못된 데이터를 제공하게 됩니다. 따라서 데이터를 캐싱하는 것만큼 유효하지 않은 캐시를 지우는 최적화 과정이 중요하게 됩니다.

## Cach Invalidation

일명 **캐시 무효화**로 무언가를 찾고자하는 요청에 대해 캐시가 사용되어야 할지 아닐지를 결정하고 항상 최신의 데이터를 반환하도록 하는 일련의 과정을 뜻합니다.

크게 2가지 방법이 있습니다.

1. 캐시의 일부를 수정하는 방식으로, 항상 최신의 데이터를 반환하도록 하기 위해 Cache Miss 가 일어났을 때, 임의로 설정한 교체정책 (LRU, FIFO 등등) 에 따라 데이터를 교체해주는 방법.
2. 일부를 찾기 힘드니 그냥 통으로 지워버리고 다시 캐싱하는 방법

## SWR

nextjs 를 만든 vercel 이라는 회사에서 data fetching 을 위해 만들어낸 리액트 훅 라이브러리입니다.

SWR 은 stale-while-revalidate 의 약자인데, stale 은 신선하지 않은.. 이미 있었던(?) 의 뜻이고, revalidate 는 재검증.. 의 뜻으로 이해할 수 있습니다.

SWR은 cache 에서 이미 있었던 신선하지 않은 데이터를 찾아 반환해주는 stale,, 요청을 보내 실 데이터를 검증해보는 revalidate, 그리고 최종적으로 캐시 데이터를 다시 최신화해주는 방식을 가지고 있는데 이러한 캐시 관리 방식으로 속도, 정확성, 안정성 등의 장점들을 가지고 있습니다.

<aside>
💡 - 빠르고 가볍고 재사용 가능한 데이터 가져오기
- 전송 및 프로토콜에 구애받지 않음
- 기본 제공 캐시 및 요청 중복제거
- 실시간 경험
등등..

</aside>

## Quick Start

```tsx
// 참고: https://swr.vercel.app/ko/examples/basic
const fetcher = (url) => fetch(url).then((res) => res.json());
const fetcher = async (url) => {
	const response = await axios.get(url);

	return response.data;
}

export default function App() {
  const { data, error, mutate } = useSWR(
    "https://api.github.com/repos/vercel/swr/get",
    fetcher
  );

  if (error) return "An error has occurred.";
  if (!data) return "Loading...";
  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.description}</p>
      <strong>👁 {data.subscribers_count}</strong>{" "}
      <strong>✨ {data.stargazers_count}</strong>{" "}
      <strong>🍴 {data.forks_count}</strong>
    </div>
  );
}
```

```tsx
const fetcher  = async(url) => return await axios.get(url).then({data}=> data)

const { data, error, isValidating, mutate } = useSWR(key, fetcher, options)
```

- key : 요청을 위한 고유한 값
- fetcher : 요청 후의 응답값을 다루기 위한 함수를 반환하는 프라미스
- mutate : 캐시 업데이트하는 함수, mutate(key, data, options) 방식으로 사용할 수 있고, 같은 key 값으로 SWR 캐싱을 하고 있는 subscriber 에게 변경됨을 브로드캐스팅 한다. (보통 로그아웃 시 사용)
- data : fetcher에 의해 반환된 data
- error: fetcher에서 던진 오류, 성공시 undefined 반환
- isValidating : 만약 요청이 있거나 로딩 중인 경우에 반환

서버로의 get, post 요청에 대한 응답을 가지고 상태로써 관리하는 방법을 편하게 사용할 수 있습니다.

## 동작원리

- useSWR
    - config 에서 cache 를 받아올때, initCache 를 통해 mutate, setter, subscribe 함수들을 묶어 SWRGlobalState 에 저장한다.
    - createCacheHelper 를 통해 cache 의 setter, getter, subscribe 함수들을 받을 수 있다. (cache를 상태화)
        
        ⇒ 해당 cache 값을 변경하면 바라보고있는 subscriber 들에게 값 변경이 전파가 되게 된다.
        
    - 리액트 권장사항대로 외부 상태값인 cache를 useSyncExternalStore 로 묶어 cached로 둔다.
    - revalidate 를 통해 api 요청을 통한 응답을 받아오게 되면 [finalState.data](http://finalState.data) 에 newdata 를 넣어주게 되고
    - 마지막으로 finishRequestAndUpdateState 를 호출 후 setCache(finalState) 를 통해 cache 값을 새롭게 설정해준다.
    - 그 후 get 접근자(Object.defineProperty 의 getter와 같음)로 정의된 get data() {}, 를 통해 외부에서 useSWR 의 data를 바라보고 있을 경우 알려주게 된다.

```tsx
export const useSWRHandler = <Data = any, Error = any>(
  _key: Key,
  fetcher: Fetcher<Data> | null,
  config: typeof defaultConfig & SWRConfiguration<Data, Error>
) => {
  const {
    cache,
    compare,
    suspense,
    fallbackData,
    revalidateOnMount,
    refreshInterval,
    refreshWhenHidden,
    refreshWhenOffline,
    keepPreviousData
  } = config

	...

  // `key` is the identifier of the SWR `data` state, `keyInfo` holds extra
  // states such as `error` and `isValidating` inside,
  // all of them are derived from `_key`.
  // `fnArg` is the argument/arguments parsed from the key, which will be passed
  // to the fetcher.
  const [key, fnArg] = serialize(_key)
  // Refs to keep the key and config.
  const keyRef = useRef(key)
  const fetcherRef = useRef(fetcher)
  const configRef = useRef(config)
  
  const [getCache, setCache, subscribeCache] = createCacheHelper<
    Data,
    State<Data, any> & {
      // The original key arguments.
      _k?: Key
    }
  >(cache, key)

  // 리액트에서 직접 관리되지 않는 상태를 참조할 때, 콜백 큐 지연 정도에 따라 값이 꼬일수 있어(tearing) 
  // 랜더링 시점에 값을 강제로 동기 참조하도록 useSyncExternalStore 사용함을 권장
  // Get the current state that SWR should return.
  const cached = useSyncExternalStore(
    useCallback(
      (callback: () => void) =>
        subscribeCache(
          key,
          (prev: State<Data, any>, current: State<Data, any>) => {
            if (!isEqual(prev, current)) callback()
          }
        ),
      // eslint-disable-next-line react-hooks/exhaustive-deps
      [cache, key]
    ),
    getSnapshot,
    getSnapshot
  )

  const cachedData = cached.data
  const fallback = isUndefined(fallbackData)
    ? config.fallback[key]
    : fallbackData
  const data = isUndefined(cachedData) ? fallback : cachedData
  const error = cached.error

	// Use a ref to store previously returned data. Use the initial data as its initial value.
  const laggyDataRef = useRef(data)

	// keepPreviousData = 키가 변경되었지만 데이터가 준비되지 않은 경우 이전 결과를 유지하는 변수
  const returnedData = keepPreviousData
    ? isUndefined(cachedData)
      ? laggyDataRef.current
      : cachedData
    : data

	...

  const revalidate = useCallback(
    async (revalidateOpts?: RevalidatorOptions): Promise<boolean> => {
      const currentFetcher = fetcherRef.current
			...
			// Start fetching. Change the `isValidating` state, update the cache.
      const initialState: State<Data, Error> = { isValidating: true }
      // It is in the `isLoading` state, if and only if there is no cached data.
      // This bypasses fallback data and laggy data.
      if (isUndefined(getCache().data)) {
        initialState.isLoading = true
      }
      try {
        if (shouldStartNewRequest) {
          setCache(initialState)
          // If no cache is being rendered currently (it shows a blank page),
          // we trigger the loading slow event.
          if (config.loadingTimeout && isUndefined(getCache().data)) {
            setTimeout(() => {
              if (loading && callbackSafeguard()) {
                getConfig().onLoadingSlow(key, config)
              }
            }, config.loadingTimeout)
          }

          // Start the request and save the timestamp.
          // Key must be truthy if entering here.
          FETCH[key] = [
            currentFetcher(fnArg as DefinitelyTruthy<Key>),
            getTimestamp()
          ]
        }

        // Wait until the ongoing request is done. Deduplication is also
        // considered here.
        ;[newData, startAt] = FETCH[key]
        newData = await newData

	...

	const cacheData = getCache().data

        // Since the compare fn could be custom fn
        // cacheData might be different from newData even when compare fn returns True
        finalState.data = compare(cacheData, newData) ? cacheData : newData

	...
	
	} catch(e) {...}

	finishRequestAndUpdateState();
     }
  }
	
  ...

  const finishRequestAndUpdateState = () => {
    setCache(finalState)
  }
	
  ...

  return {
    mutate: boundMutate,
    get data() {
      stateDependencies.data = true
      return returnedData
    },
    get error() {
      stateDependencies.error = true
      return error
    },
    get isValidating() {
      stateDependencies.isValidating = true
      return isValidating
    },
    get isLoading() {
      stateDependencies.isLoading = true
      return isLoading
    }
  } as SWRResponse<Data, Error>
}

```

- Polling
    - options 로 refreshInterval 값을 넘기게 되면 자동으로 데이터를 갱신하게끔 설정할 수 있습니다.
    - 요것도 결국 execute 함수의 revalidate 를 호출함으로써 서버의 데이터를 가져와 setCache 해주는 방식입니다.

```tsx
// Polling
useIsomorphicLayoutEffect(() => {
  let timer: any

  function next() {
    // Use the passed interval
    // ...or invoke the function with the updated data to get the interval
    const interval = isFunction(refreshInterval)
      ? refreshInterval(data)
      : refreshInterval

    // We only start the next interval if `refreshInterval` is not 0, and:
    // - `force` is true, which is the start of polling
    // - or `timer` is not 0, which means the effect wasn't canceled
    if (interval && timer !== -1) {
      timer = setTimeout(execute, interval)
    }
  }

  function execute() {
    // Check if it's OK to execute:
    // Only revalidate when the page is visible, online, and not errored.
    if (
      !getCache().error &&
      (refreshWhenHidden || getConfig().isVisible()) &&
      (refreshWhenOffline || getConfig().isOnline())
    ) {
      revalidate(WITH_DEDUPE).then(next)
    } else {
      // Schedule the next interval to check again.
      next()
    }
  }

  next()

  return () => {
    if (timer) {
      clearTimeout(timer)
      timer = -1
    }
  }
}, [refreshInterval, refreshWhenHidden, refreshWhenOffline, key])
```
