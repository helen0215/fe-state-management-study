# 상태관리란?
> 상태관리란 클라이언트 애플리케이션의 효과적인 데이터 관리 방법을 "상태관리"라고 한다.

# 배경
과거의 웹 사이트는 그저 URL에 해당하는 리소스를 BE에서 정적인 페이지를 응답을 받아 브라우저에 보여주는 것에서<br/>
사용자와의 상호작용에 따라 다시 페이지를 로드할 필요없이 동적으로 페이지를 구성하고<br/>
필요하면 BE에 데이터를 요청, 응답하는 형태로 발전해왔다.

BE와 FE의 역할이 구분되면서 사용자에게 보여주고 데이터를 요청하기 위해 FE에서 데이터 관리가 필요하게 됐다.<br/> 
우리는 이것을 "상태관리" 라고 말하며<br/> 
현재는 이 "상태관리"로 인한 페이지 렌더링 최적화, UI처리를 위해 FE를 위한 서버를 두는 형태로도 발전하고 있다.

"상태관리"가 Angular, React, Vue 같은 FE 프레임워크들이 인기를 끌면서 생긴 새로운 것은 아니다.<br/>
사용자에게 보여주어야 하는 상태와 내부적으로 관리해야하는 상태(요청 실패 등)상태는 이전에도 있었고<br/>
MVC, MVP도 Model과 View를 두어 상태를 관리하는 대표적인 디자인 패턴이며<br/>
이런 양방향 상태관리를 개선하고자 우리가 잘 알고 있는 MVVM, Flux 패턴이 등장했다.<br/>
~(하지만 애플리케이션이 커지면 문제는 생기기 마련이고, 그래서 개발자로 오래 먹고 살수 있게 되었다.)~

# 개인적으로 생각하는 상태관리의 문제점
> 1. 코드의 비대
> 2. 남용과 부작용 (렌더링과 관련없는 상태들로 인한 커진 렌더링 비용)

MVC가 자유로운 패턴이며 애플리케이션이 커짐에 따라 양방향에서 오는 잦은 상태의 업데이트를<br/> 
뷰와 동기화 하여 관리하기 어려우니 MVC보다 엄격한 단방향 패턴이 탄생했고 MVVM, Flux 패턴과<br/> 
이를 기반으로 프레임워크들이 만들어졌다고 생각한다.
단방향 패턴도 애플리케이션이 커짐에 따라 props drilling 문제가 발생!<br/> 
contextAPI, eventbus, 전역상태를 잘 관리하기 위해 상태관리 라이브러리를 사용한다고 생각한다.

상태관리 라이브러리가 프레임워크와 유기적으로 동작하여 주는 편리함도 있다.<br/> 
하지만 렌더링 최적화 문제는 우리를 늘 따라다니며<br/> 
간단한 애플리케이션에도 상태관리를 넣어 실제 다루는 데이터 보다 더 많은 코드를 작성해야하며<br/> 
비대해지는 상태관리 코드는 상태를 추가할때마다 부담을 준다.

렌더링 최적화나 상태관리의 분리를 문제를 해결하기 위해 FE를 위한 서버를 두는 형태도 있다.<br/>
하지만 서버를 위한 상태, CSR를 위한 상태, 서버 캐싱을 다루기 위한 상태를 관리해아하는 끔찍한 혼종이 나오기도 한다.<br/>
~문제를 해결하면 새로운 문제가 나온다. 역시 FE 개발자로 오래 먹고 살수 있게 되었다.~

우리가 관리하는 상태는 변경되면 화면 렌더링에 영향을 미친다.<br/>
하지만 그렇지 않은 경우도 있다.<br/>
사용자가 화면에서 텍스트를 입력하거나 버튼을 누르면 화면 상 변경은 일어나지 않은 채 상태를 변경하거나 서버에 요청을 보내야한다.<br/>
상태관리 라이브러리는 상태가 변하면 화면을 다시 렌더링하기 위한 목적으로 만들어졌는데<br/>
화면 렌더링외에도 관리하고 있는 상태(데이터)들이 섞여있어 더 혼란함을 주는것은 아닐까?

# 이 스터디가 끝나고 질문해보면 좋을 것
> 우리가 알고 있는 문제들은<br/>
상태관리 라이브러리가 필요없는데 사용한다던가, 상태관리를 하지 않아도 되는 데이터를 상태관리에 넣어 관리하는 등<br/>
상태관리 라이브러리를 잘 알지 못하고 남용해서 생겨난 문제가 있을 것 이라고 생각한다. ~(내얘기..)~<br/>
이번 스터디를 통해 상태관리를 잘 이해하고 우리가 운영중인 서비스에 남용된 상태관리를 걷어내는 기회가 된다면 좋겠다.

* 현재 운영중인 프로젝트에서 개선할 점이 있는가? 있다면 무엇인가?
* 화면 렌더링을 위한 상태와 아닌 상태를 분리하는게 좋은가?
* 상태관리 라이브러리 선택할 때 어떤 점(어떤 문제를 회피하기 위해)을 고려해야 하는가? 

# 같이 읽어보기
[전역 상태 관리에 대한 단상 (stale-while-revalidate)](https://jbee.io/react/thinking-about-global-state/)<br/>
[On state management and why I stopped using it](https://dev.to/beggars/on-state-management-and-why-i-stopped-using-it-4di)<br/>
