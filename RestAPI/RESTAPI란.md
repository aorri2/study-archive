#RestApi 
> 본 글은 그런 REST API로 괜찮은가 라는 유튜브 영상을 참고하여 작성한 글입니다.

개인 프로젝트나, 회사에서 프로젝트를 진행하면서, HTTP를 이용한 API를 작성하게 됩니다.  
이때마다 REST API 라는 단어를 마주쳤으나 과연 내가 작성하고 있는 API가 REST API가 맞는가? 라는 의문이 들었습니다. 그렇기에 REST API에 대한 오해를 풀고, 좀더 깊은 이해를 하기 위해 해당 글을 작성하게 되었습니다.

먼저 저는 REST API는 URI를 통해 Resource를 표현하면 된다고 생각했습니다.

-   `/members/delete/1`

그리고 자원에 대한 행위는 HTTP Method를 통해 표현하면 된다고 생각했습니다.

-   `DELETE /members/1`

REST API를 정의한 로이 필딩에 의하면 위와 같이 작성된 API는 REST API라고 말할 수 없다고 합니다. 그렇다면 REST API에서는 추가적으로 어떤 조건들을 만족해야 하는지 알아보겠습니다.

### Server-Client 구조

자원을 갖고 있는 쪽이 Server이고, 해당 자원을 요청하는 쪽이 Client라고 합니다.  
이렇게 함으로써, client는 server에서 어떤 일을 수행하더라도 내부 작업을 알 필요 없이, 요청한 정보에 대한 결과만 받으면 되기에, 각각의 클라이언트와 서버가 독립적으로 확장될 수 있습니다.

### Stateless(무상태)

HTTP 프로토콜 기반에서 작동하는 REST API도 HTTP 프로토콜과 마찬가지로 무상태여야 합니다.  
그렇기에 클라이언트에서 서버로 하는 각 요청에는 그 요청에 필요한 모든 정보가 포함되어야 합니다.

### Cacheable

요청에 대한 응답 내의 데이터에 대한 요청은 캐시가 가능한지 불가능한지 명시해야 합니다.

-   HTTP Header에 `cache-control` 헤더를 이용해 표시할 수 있습니다.

### Uniform Interface

Uniform Interface는 URI로 지정된 리소스에 대한 조작을 통일하고 한정된 인터페이스로 수행하는 아키텍쳐 스타일 입니다.

**Uniform Interface의 4가지 제약 조건**

1.  Resource Based
2.  Manipulation Of Resources Through Representations
3.  Self-Descriptive Message
4.  Hypermedia As The Engine Of Application State

대부분의 사람들이 1번과 2번에 만 관심을 가졌을 것 이다. 따라서 3번과 4번을 중점적으로 설명하겠습니다.

#### Self-Descriptive Message

메시지가 스스로 자신에 대한 설명을 할 수 있어야 한다.  
즉, API 문서가 REST API 응답 본문에 존재해야 한다는 말입니다.

**그렇다면 왜 이런 방식을 채택해야할까요?**

이 방식을 사용하게 된다면, 서버 내부의 변경으로 인해, Response Data가 변경되었을 때, 클라이언트에서는 해당 API 문서를 통해 어떤 데이터가 바뀐건지 알 수 있게 됩니다.

#### Hypermedia As The Engine Of Application State

Hypermedia(링크) 를 통해 애플리케이션의 상태 전이가 가능해야 한다.  
또한 Hypermedia에 자기 자신에 대한 정보가 담겨야 한다.

예를 들어 다음과 같이 게시글을 조회하는 URI가 있다고 가정하겠습니다.  
`GET https://simgee.com/article`  
위 URI를 통해 해당 글을 조회한 사용자는 다음 행동으로 어떤 행동을 할 수 있을까요?

-   다음 게시물 조회
-   댓글 달기
-   게시물을 내 피드에 저장

위와 같은 행동들이 바로 상태 전이가 가능한 것들입니다. 이런 내용들을 응답 본문에 넣어줘야 한다는 말입니다.

따라서 Hypermedia(링크)를 통해서 상태 전이가 가능한 것들을 넣어줄 수 있습니다.

```
{ 
"data": { 
"id": 1000, 
"name": "게시글 1", 
"content": "HAL JSON을 이용한 예시 JSON", 
"self": "http://localhost:8080/api/article/1000", // 현재 api 주소 
"profile": "http://localhost:8080/docs#query-article", // 해당 api의 문서 
"next": "http://localhost:8080/api/article/1001", // 다음 article을 조회하는 URI "comment": "http://localhost:8080/api/article/comment", // article의 댓글 달기 
"save": "http://localhost:8080/api/feed/article/1000", // article을 내 피드로 저장 }, }
```

이렇게 사용하게 되면

1.  API 버전을 명시하지 않아도 됩니다. `/api/v1/...` 과 같이 말입니다.
2.  링크 정보를 동적으로 바꿀 수 있습니다.
3.  링크를 통해서 쉽게 상태 전이가 가능합니다.

위와 같은 방식으로 데이터를 클라이언트에게 보내준다면, 클라이언트는 해당 링크를 참조하는 방식으로 쉽게 탐색할 수 있게 됩니다.

이외에도 HAL JSON을 이용해 HATEOAS를 더 완벽하게 구현할 수 있습니다.  
관련한 내용은 참고 링크를 참고해주세요

### 마무리

위 제약을 모두 지킨 REST API는 API 대한 정보가 모두 들어가 있게되어 서버측에서 API가 변경되고 수정됨에 따라 Client도 자연스럽게 같이 수정될 수 있는 장점이 있는것 입니다. 하지만 이런 장점이 있는 반면에 REST API의 반환 값에 전송될 데이터의 크기가 증가해 서버간 통신 비용이 증가한다는 단점이 있을 수도 있으니, 각각의 장단점을 잘 고려해 적용하는게 좋을 것 같다는 생각입니다.

이 뿐만아니라 [Martin Fowler가 제안한 Richardson 성숙도 모델](https://martinfowler.com/articles/richardsonMaturityModel.html)과 같은 글에서는 REST API를 단계적으로 나눠놓기도 했으니, 참고해보시기 바랍니다.

## 참고

[https://meetup.nhncloud.com/posts/92](https://meetup.nhncloud.com/posts/92)  
[https://www.youtube.com/watch?v=RP\_f5dMoHFc](https://www.youtube.com/watch?v=RP_f5dMoHFc)  
[https://martinfowler.com/articles/richardsonMaturityModel.html](https://martinfowler.com/articles/richardsonMaturityModel.html)