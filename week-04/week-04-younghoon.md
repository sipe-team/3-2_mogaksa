# 4주차 활동일지

원래는 후보자들 끼리의 매칭을 ai를통해 추천해주는 로직을 검증하려 했....

## 목표 

처음 생각했던 프로젝트 개발이 규모가 커짐에 따라 프로덕션 배포가 나가야 했고 처음 실험적인 서비스 구성의 목표를 버리고 
값싸고 안정성있는 인프라구성에 집중 했습니다.

## 당면한 문제점

### 서버의 메모리 부족으로 인한 시스템 다운

저번주에 메모리의 최소 용량을 256MB로 줄여서 해결했지만 어플리케이션 자체가 요구하는 메모리가 상당히 많아( 약 400MB 이상;;)

롤링 배포시 컨테이너가 2개가 뜨면 프리티어로 구성한 1GB ram을 가진 노드가 감당을 못해서 어플리케이션이 동작하지 않는 황당한 상황발생...

### 해결방안

- 과금이 조금 되더라도 fargate로 옮기기로 하였습니다. 그렇다면 프리티어로 커버안되는 과금은 어떻게할 것인가? 

- credit 헌팅을 통해 수급할 예정입니다. 마침 설문조사하면 $100를 주는 이벤트를 통해서 팀원들마다 설문에 참여했고 크레딧 보상을 대기중입니다.

## 이후 주어진 태스크

- kotlin과 layered architecture로 작성된 backend-core에 SQS 비동기 통신 인프라 레이어 코드 작업 

- 클라이언트들의 이미지를 저장할 이미지스토리지로 S3를 활용 serverless인 lambda를 활용해 클라이언트사이드에서 s3 url이 노출되지 않게 presigned url 을 generate하는 로직을 구현할 예정

- prompt 엔지니어링을 통해 ai가 scoreing 할 수 있는 기준을 잡고 정해진 대답(json data형식 등)을 수행하도록 작업해야함..
