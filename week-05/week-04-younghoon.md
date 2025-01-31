
# 5주차 활동일지 및 마무리

서비스 규모가 너무 커졌고.. 나는 감당할 수 없는 상황이 되었다...

그래서 돈이라도 벌어왔다..

## 목표 

커진 규모에 맞게 주어진 태스크들이 늘어나면서 AI를 '실험' 하려했던 나는 AI를 실제 비즈니스에 '적용' 해야되는 책임감을 떠안았다..
1. AI sdk를 활용해 사용하는 labmda에 SQS를 붙혔으니 SQS를 호출하는 backend 로직을 내가 담당해보자
2. 클라이언트의 이미지를 저장할 이미지스토리지를 생각못했다. S3로 만들되 s3 url이 노출되지 않게 lambda에서 presigned url을 생성해주는 로직을 만들자.
3. prompt 엔지니어링을 통해서 스코어링 기준과 input/output을 backend 서버가 알아들을 수 있게 고정시키자!
4. 서비스 안정싱이 최우선!
5. 크레딧 헌팅 

## 결과

### 서버의 안정성

1. fargate활용 - 0.25 cpu, 512MB RAM 배포 170초, 과금됨 뒤지게느림 서버 죽은적은 없음..
2. node활용 - 1 cpu, 1 GB RAM, 배포 30초, 과금안됨 프리티어커버되는 선에서 t4g.micro로 생성(1코어 1GB램) 롤링 배포시 메모리 수시로 터짐
3. fargate로 회귀 - 1 cpu, 1GM RAM, 배포 70초 과금됨 하지만... 크레딧으로 해결해볼까?

결국 크레딧 헌팅으로 $225 의 크레딧을 얻었고 현재 과금상황을 보아선 한 6개월은 커버될 것으로 보인다..

![image](https://github.com/user-attachments/assets/c750fcde-8e53-48cb-903e-1ee0b13503b8)

### SQS호출 로직 작성

사이프톤에서 우재선생님에게 배운 layered architecture를 적용해보기위해 코드 전체구조는 아니지만 infrastruture layer의 SQS Provider 작성과 member 도메인에서 호출하는 것 까지 작성해보는중
infrastructure layer에 어디에 위치시켜야하는지?
interface로 작성해야할지? 구현체는 어디에?
머리가 아프다


![image](https://github.com/user-attachments/assets/91777a32-4e9d-47a5-a969-adeebcff44a7)

일단은 Invitatin 이라는 도메인 밑에 infrastructure layer 에다가 event infra를 따로 만들고 주고받을 클라이언트 정보의 dto와 함께 provider를 만들어놓았다..


### s3 and presigned url gerneration

이건 뭐 간단했다.

lambda 에서 boto3 이용해 s3 sdk 사용하면 아주 간단하다.
```
        presigned_url = s3_client.generate_presigned_url(
            'put_object',
            Params={
                'Bucket': bucket_name,
                'Key': filename
            },
            ExpiresIn=expiration,
            HttpMethod='PUT'
        )
        
        # 생성된 presigned_url을 API 응답으로 반환
        return {
            "statusCode": 200,
            "body": json.dumps({
                "presigned_url": presigned_url
            })
        }
```
generate_presigned_url 메소드 사용하고 결과값을 return 해주면 끝
코드는 참 쉽다

하지만 S3와 LAMBDA 모두 '권한설정'이 골치아프다. 

### AI가 쉽지만은 않더라..

AI를 너무 만만하게 보았다. prompt에서 대화 서술형으로 깔짝대는 것과 서버가 sdk로 호출해 원하는 답을 나오게 만드는 법 그리고 그 대답이 얼마나 정확성이 높은지 스코어링하는 것 
모두 힘들다.. 죽을 것 같다..

prompt 엔지니어링이라는게 무엇인지 이 과정을 통해서 제대로 배웠다. 

'그거 그냥 말만 잘하면 되는거아니야?'

'응 아니야.'

예를 들어 이런식이다.

```
System instructor: You are a professional matchmaker who make lover man and woman

Input Example : 

{
  "gender" : "male"
  "age" : int
  "name" : string
  "idealType":{
    "ageRange": {"min": int, "max": int}
    "mbti: str
    ...
  }
 ....
}

If I give you Input like this you have to answer output like this

Output Example :
{
  "gender" : "female"
  "age" : int
  ....
}

If I giva you input data which has "gender" value "male". you have to answer "female" data which is best 

```
AI에게 너가 무슨일을 해야할 지 알려주고

어떤 input을 줄것이며 어떤 output을 내야할 지 알려줘야한다

이게 참 쉽지않다 

하지만 bedrock 에서 제공하는 knowledge base를 사용하면 

bedrock sdk에서 프롬프트 엔지니어링을 위한 사전 input form을 정의 할 수 있다.!


```
'promptTemplate': {
  'textPromptTemplate': 'string'
}
```

여기에 써주면 된다..

## 결론

주섬주섬으로 시작했을 때의 나의 목표는 누군가가 원하는 비즈니스에 AI 특히나 LLM 을 RAG형식을 통해 값싸고 간단하게 적용해서 결과물을 확인해보는 것이었다.
cufit으로 바뀌면서 프로젝트는 규모가 커지고 나는 안정적인 인프라와 repo구성, secret 변수처리, CICD 파이프라인 등을 담당하게 되었다..
작은 규모의 프로젝트라고 만만히 봤다가 서버가 자주터져서 참 골치아프고 팀원들한테 미안했다.. 

아직도 디버깅하려면 내가 직접 aws에 로그인해서 봐줘야한다. 에러날 때마다 프론트 - 백엔드 - 나 셋이서 뭉쳐서 디버깅한다.. 참으로.. 힘든 삶이다..
프로덕션 환경의 서버에서 가시성을 높이려면 그것도 다 '돈'인데...

cufit을 이끄는 팀장님에게 ai 관련 로직은 MVP(최소기능배포) 까지는 아니니까 살살해달라고 무릎꿇고 빌었다... 아직 대답은 못들었다.


동아리 시작하고 나서 내인생에서 가장 열심히 살았던 2024 후반기 였다
덕분에 발표도 많이하고 강의도 하고 좋은 친구들도 만들고 레퍼럴도 받고 행복한 삶이었다..

But it's not over yet.....


