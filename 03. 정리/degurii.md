## 느낀점

어떤 상태관리 라이브러리를 쓸 지가 중요한 게 아니라, 어떤 문제를 해결하기 위해 나온 건지에 초점을 맞춰야겠다고 생각했습니다.

- 리덕스: prop drilling 문제를 해결하고, 상태를 한 곳에서 관리하여 디버깅을 용이하게 하기 위함
- 레코일: 리덕스는 다 좋은데, 보일러 플레이트가 너무 많다. 더 가볍게 쓸 수 있을까?
- react swr, react query: 데이터 페칭과 동기화에서 오는 불편함을 해결, 프론트엔드 상태의 개념을 로컬 상태, 서버 상태로 분리 
- pinia: 이번에 처음 들어봤는데, 결국 vuex의 불편함을 해결하기 위해 도입됨

이렇게 살펴보면 라이브러리들은 기존 도구들의 한계점과 불편함을 해결하기 위한 방향으로 발전하는 것 같습니다.

이런 관점과 테크닉을 우리의 상황에 맞게 적용하는 것이 더 중요지 않을까.. 하는 생각입니다.
