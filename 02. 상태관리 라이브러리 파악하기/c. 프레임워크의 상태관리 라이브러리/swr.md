# SWR이란

생성일: 2022년 8월 9일 오후 12:48

💡 `(stale: 신선하지 않은, revalidate: 재검증)`
<br />
<br />

SWR이 뭔지 알아보기 앞서 캐시에 대해 간단하게 짚고 넘어가면 **캐시**는 빠른 응답 또는 비용을 절약하기 위해 사용하는 일종의 임시 저장 데이터입니다. 

간단하게 무언가를 찾고자할 때 캐시에 없으면 `Cache Miss` 로 표현한 뒤 **cache** 에 저장해두고, 그 후에 같은 걸 찾고자할 때 캐시에 있으면 `Cache Hit` 으로 표현하고, 그대로 가져다 사용할 수 있어 속도 비용의 이점을 가지고 있습니다.

이 과정에서 Stale Cache Handling 이라는 개념이 나옵니다.

캐시는 빠른 응답을 위한 임시 데이터이기 때문에 원래의 데이터에 종속적입니다. 때문에 원본 데이터가 바뀌게 되면 이전에 생성된 캐시는 더 이상 쓸모가 없어집니다.
오히려 오래된 캐시가 남아있으면 잘못된 데이터를 제공하게 됩니다. 따라서 데이터를 캐싱하는 것만큼 유효하지 않은 캐시를 지우는 최적화 과정이 중요하게 됩니다.

## Cach Invalidation

일명 **캐시 무효화**로 무언가를 찾고자하는 요청에 대해 캐시가 사용되어야 할지 아닐지를 결정하고 항상 최신의 데이터를 반환하도록 하는 일련의 과정을 뜻합니다.

크게 2가지 방법이 있는데,

1. 캐시의 일부를 수정하는 방식으로, 항상 최신의 데이터를 반환하도록 하기 위해 Cache Miss 가 일어났을 때, 임의로 설정한 교체정책 (LRU, FIFO 등등) 에 따라 데이터를 교체해주는 방법.
2. 일부를 찾기 힘드니 그냥 통으로 지워버리고 다시 캐싱하는 방법

## SWR

data fetching 을 위해 만들어진 리액트 훅 라이브러리입니다.

SWR 은 stale-while-revalidate 의 약자인데, stale 은 신선하지 않은.. 이미 있었던(?) 의 뜻이고, revalidate 는 재검증.. 의 뜻으로 이해할 수 있습니다.

SWR은 cache 에서 이미 있었던 신선하지 않은 데이터를 찾아 반환해주는 stale,, 요청을 보내 실 데이터를 검증해보는 revalidate, 그리고 최종적으로 캐시 데이터를 다시 최신화해주는 방식을 가지고 있는데 이러한 방식으로 속도, 정확성, 안정성 등의 장점들을 가지고 있습니다.

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
import React from "react";
import useSWR from "swr";

const fetcher = (url) => fetch(url).then((res) => res.json());

export default function App() {
  const { data, error } = useSWR(
    "https://api.github.com/repos/vercel/swr",
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
