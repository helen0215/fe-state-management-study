# 본격적인(?) 상태관리의 시작
## Flux
<img width=1024 src=https://facebook.github.io/flux/img/overview/flux-simple-f8-diagram-explained-1300w.png>

**배경**
* We originally set out to deal correctly with derived data: for example, we wanted to show an unread count for message threads while another view showed a list of threads, with the unread ones highlighted. This was difficult to handle with MVC — marking a single thread as read would update the thread model, and then also need to update the unread count model.
  * => MVC 로 기능을 구현하는데 어려웠다!

**해결**
* 데이터를 단방향으로 흐르도록 구조를 고안했다.

**장점**
* This structure allows us to reason easily about our application in a way that is reminiscent of functional reactive programming, or more specifically data-flow programming or flow-based programming, where data flows through the application in a single direction — there are no two-way bindings.
  * => 이 구조를 사용하면 우리 앱을 functional reactive programming, 구체적으로 data-flow programming 의 방식으로 추론하기 쉬워진다.
* Application state is maintained only in the stores, allowing the different parts of the application to remain highly decoupled.
  * => 상태를 스토어에서만 관리해서, 앱의 다른 부분들과의 커플링을 낮게 유지할 수 있다.

**출처**
* https://facebook.github.io/flux/docs/in-depth-overview

## Redux
<img width=1024 src=https://miro.medium.com/max/1400/1*THgvhJmeqo6rsR98qAnD0A.jpeg>

**배경**
* SPA 앱이 더 복잡해지면서 많은 상태를 관리하게 되었는데, 항상 변하는 상태를 관리하는 것은 어렵고 결국에는 상태를 언제, 왜, 어떻게 업데이트해야하는지 모르는 지경에 이르고 말았다.

**해결**
* [Flux](https://facebook.github.io/flux), [CQRS](https://martinfowler.com/bliki/CQRS.html), [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html) 을 따라, Redux는 상태 변화가 일어나는 시점에 제약을 두어 상태 변화를 예측 가능하게 만들고자 시도했다.

**장점**
* 단일 스토어이고, 상태는 불변을 보장하고, 상태를 갱신하는 역할을 리듀서가 가져갔기 때문에, 디버깅이 용이하고, 개발 중에 상태를 저장하고, 상태 undo/redo 를 구현하기 쉽다.

**출처**
* [https://ko.redux.js.org/understanding/thinking-in-redux/motivation](https://ko.redux.js.org/understanding/thinking-in-redux/motivation)

## Context + Hook

**배경**
- 앱 안의 여러 컴포넌트들에게 props 를 건네주는 일은 번거로울 수 있다.

**해결**
- context 를 사용해 하위 컴포넌트 트리 내에서는 전역적으로 상태를 공유한다.

**장점**
- 하위 다양한 레벨의 컴포넌트들에게 손쉽게 데이터를 전달할 수 있다.

**출처**
- [https://ko.reactjs.org/docs/context.html](https://ko.reactjs.org/docs/context.html)

## SWR

**배경**
- 보통 최상위 레벨 컴포넌트에서 가져온 모든 데이터를 유지하고 트리 아래의 모든 자식 컴포넌트의 props로 추가해야 합니다. 페이지에 더 많은 데이터 의존성을 추가한다면 코드는 점점 유지하기가 힘들어집니다.
- [Context](https://reactjs.org/docs/context.html)를 사용하여 props 전달을 피할 수 있습니다만 동적 콘텐츠 문제가 여전히 존재합니다: 페이지 콘텐츠 내 컴포넌트들은 동적일 수 있으며, 최상위 레벨 컴포넌트는 그 자식 컴포넌트가 필요로하는 데이터가 무엇인지 알 수 없을 수도 있습니다.

**해결**
- SWR 키를 사용하여 중복 제거, 캐시, 공유되므로, 단 한 번의 요청만 API로 전송한다.

**장점**
- 불필요한 네트워크 요청을 줄여준다.
- 서버 응답 처리에 대해 정의된 useSWR 훅을 컴포넌트에서 재사용할 수 있다.
- 엄청 다양한 옵션을 제공한다.

**출처**
- [https://swr.vercel.app/ko](https://swr.vercel.app/ko)


## 참고

- [https://medium.com/razroo/the-history-of-javascript-state-management-in-2019-161491d588ed](https://medium.com/razroo/the-history-of-javascript-state-management-in-2019-161491d588ed)
- [https://leerob.io/blog/react-state-management](https://leerob.io/blog/react-state-management)
- [https://velog.io/@soryeongk/SWRBasic](https://velog.io/@soryeongk/SWRBasic)
