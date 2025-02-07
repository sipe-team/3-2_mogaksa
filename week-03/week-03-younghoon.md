# 3주차 활동일지
본래 Bedrock의 Knowledgebase 로 AI 로직을 검증 및 테스트 하려 했지만 데이터 부족으로 4주차 활동인 Lambda 개발을 했습니다.

AI 서버리스 lambda 로직 중 후보자를 등록하는 로직인 regist 부분을 개발했습니다.

## 서버의 메모리 부족으로 인한 timeout 발생

중간에 프로젝트 개발팀에게 서버가 timeout으로 상태가 안좋다는 이야기를 들었습니다.

메모리 최소치를 512MB로 잡았는데 프리티어로 커버되는 t4g.micro 타입을 사용하기에 전체 RAM 할달량이 1G였고 OS가 사용하는 부분을 제외하면 여유분은 800MB 정도였습니다.

그치만 back api 서버의 컨테이너 최소치가 512MB이어서 rolling 방식으로 배포될 때에 무중단을 위해 잠시 2개의 컨테이너가 할당되어야 하는데 리소스가 부족해서 배포가 안되고 기존의 컨테이너 마저 메모리가 부족해 timeout 현상이 일어났습니다.

그러하여 메모리의 최소치를 256MB로 잡았으며, fargate에서 노드방식으로 변경함에 따라 서버가 뜨는 시간은 늘었지만 서버가 실행되는 시간은 여전히 30초이상이 걸리기에 로드밸런서 에서 timeout 제한을
100초로 늘렸습니다.

## ai증강을 위한 후보자 데이터 저장 로직 구현

코드의 기본구조는 잡아놓았으며 SQS로 메시지를 송신하면 송신하자마자 이벤트가 발생해 lambda가 실행되는 서버리스 및 비동기 방식으로 구현되었습니다.
registPerson 이라는 eventType으로 실행되게 해놓았으며 body로 들어온 data를 json과 metadata 형식으로 AI의 knowledgeBase 소스 스토리지인 S3에 저장되는 방식으로 구현하였습니다.
