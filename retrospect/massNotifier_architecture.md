# 대용량 알림 처리 서버 개발 회고록 1: 대용량 처리를 위한 이벤트 스트리밍 아키텍쳐 설계

### 개요 (TL;DR)
#### 개발한 것: 대형 알림 처리 마이크로 서비스 (TPS 10K ~ 1M)
- 메세지 큐와 이벤트 스트리밍 구조를 활용한 대용량 트래픽 처리 서비스 개발 경험
---
### 배경
- 현재 개발하고 있는 '대용량 알림 서비스'는 다음과 같은 조건으로 개발되고 있다.
    1. 초당 적게는 만 건, 많게는 백만 건의 요청을 감당할 수 있는 서비스.    
    2. 기술 교체가 쉬운 서비스 (MVP는 Redis, RabbitMQ로 개발하지만, 나중에는 Apache Kafka로 교체하기 쉽도록)

### 구조 설계: 메세지큐를 활용한 이벤트 스트리밍 아키텍쳐

- 1번과 같은 경우는, 이벤트 스트리밍 구조와 Fire & Forget전략을 취하여 해결을 하였다.
    - 클라이언트로부터 요청이 오는대로 일단 메세지큐에 쌓아두고, 클라이언트에게는 "이벤트가 형성되었다"라는 의미의 `201 CREATED` 응답을 준다.
    - 메세지 큐에서 해당 요청이 consume이 된 후에 처리가 오래 걸리는 알림 작업을 한다.
    - 클라이언트는 어떤 경우에도 빠른 응답을 받을 수 있을 것이다. 
    - 나중에 모니터링 api를 제공해줌으로 결과를 확인할 수 있게 할 예정이다.
- 2번과 같은 경우는, Core Manager와 Core Provider를 모듈화하여 해결을 하였다.
    - 나중에 기술이 바뀌어도, Core Provider만 바꾸면 되는 형식으로 만들어, 각 모듈을 분리하고 의존도를 낮추었다.

- 그리고 다음과 같은 추가적인 전략도 취했다.
    1. Stash & Flush 전략: 알림의 내용이 너무 긴 경우, 이를 '이벤트'라는 단위로 나누어 처리를 한 뒤, 나중에 한번에 다시 직렬화 (Serialization)을 하여 전송을 한다.
        - Queue는 순서가 보장되기에, 여러개의 이벤트로 이루어진 알림의 경우, 일단 이벤트들을 저장하고 (Stash), '마침'을 알리는 이벤트가 오면 다시 불러와서 (Flush) 직렬화후 처리하는 전략을 취했다.

---
### 기술 선택
- 회사의 프레임워크에 맞춰 백엔드 설계는 `expressJS`로 했다. NestJS만 사용하다가 오랜만에 `expressJS`를 사용하여서, 보일러플레이팅부터 시간이 조금 오래 걸렸다.
- 메세지큐는 일단 `Redis` Pub/Sub으로 구현을 하고, 이후 `Apache Kafka`로 기술 이전 시 문제가 없도록 모듈화하여 의존도를 낮춘다.
- 캐싱, Rate Limit 관리용 db역시 `Redis`를 사용한다.
- `ajv`로 요청에 대한 validation을 진행한다.
- Linter/Formatter로는 최근에 알고 사용하기 시작한 `Biome`을 사용하였다. lint, format이 모두 되기도 하고, 기존에 eslint와 prettier간 충돌을 해결하는 것이 너무나도 피곤한 일이었기에, `Biome`의 존재는 나에게 있어서 좋은 옵션이었다. ~~제발 제 2의 Rome이 되지 않기를~~
- Dev 환경은 `nodemon`으로, Production 환경은 `pm2`로 서버를 돌리고 `docker`로 containerize한다.
- 테스트는 `Jest`를 사용할 것이지만, TBD가 아니기 때문에 코어를 제외하고는 테스트 코드를 짜지는 않을 생각이다.
- Package Manager로는 `pnpm`을 사용해보고 싶어서 도입을 해보았다.