# fe-state-management-study
FE 상태관리 스터디!

> 아래 진행 단계의 상세 내용은 스터디 회차를 진행함에 따라 달라질 수 있음
```
1. 상태관리란? 
  a. 상태관리의 등장 배경, 철학 현재까지의 흐름 
  b. 상태관리 라이브러리가 해결하려는 문제와 어떻게 문제를 해결했는지?
2. 상태관리 라이브러리 파악하기 
  a. 그룹별 담당하고 있는 서비스의 상태관리 현황 (비밀)
    - 8/9 카터네가 먼저! ㅋㅋㅋㅋㅋ
    - 8/16 쥬디네!
  a-1. 상태관리에 대한 고민
    - 전역 상태와 지역 상태는 어떻게 구별되어야 하는가? 
    - props drilling 이 나쁜가? 아니라면 어떨때 나쁜가, 어떻게 더 잘 다룰 수 있는가 
    - 기능 하나를 우리가 만든 상태관리 라이브러리로 만들면 어떨까?
  b. javascript로 어떻게 상태를 관리할 수 있는 방법 - helen0215 프레임워크 없는 프론트엔드 개발의 상태관리 예제 코드 (https://github.com/helen0215/frameworkless-front-end-development/tree/master/Chapter07)
  c. 다른 프레임워크의 상태관리 라이브러리 코드 분석 (반응성, 최적화 방법, 사용 패턴 등)
    - 제이미 : pinia 9/6 
    - 케이드 : vuex 9/6
    - 주디 : recoil 9/15
    - 카터 : swr 9/15
    - 케빈 : redux 9/20
3. 9/27 정리: 스터디 내용을 각자 돌아가며 새롭게 알게 된것, 느낀점, 최종적으로 어떤걸 만들고 싶은지 공유
  a. 어떤 문제를 어떤 방법으로 해결 하고 싶은지 (라이브러리, 상태 모으기 등)
     L 예) 상태관리 라이브러리를 만든다면 꼭 들어가야 하는 스펙에 대해서 이야기 (캐싱, 디버깅...)
     L 예) 해결해야하는 문제와 방법, 실제 운영하는 서비스에서의 상태관리는 도메인에 적합하게 만들어져있음.
           -> 현재 프로젝트의 문제들을 다시 가져와서 해결하는 것에 집중 하는 것도 좋을 것 같다

4. 실습 

```
