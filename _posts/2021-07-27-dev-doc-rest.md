---
layout: post
title:  "REST API의 이해"
subtitle:   "What is REST?"
categories: dev
tags: sonarqube install
comments: true
---

# REST API의 이해
#### ___feat. 그런 REST API로 괜찮은가 - DEVIEW 2017___
"Representation State Transfer" are a way of providing interoperability between computer systems on the Internet. 인터넷상의 컴퓨터 시스템간에 상호 운용성을 제공하는 방법.. 즉, 자원을 자원의 표현으로 구분하여 정보를 주고받는 모든것을 의미하며, www과 같은 분산 하이퍼미디어 시스템을 위한 소프트웨어 아키텍처의 한 형식이다. 

- 배경  
HTTP를 명세한 주요 저자 중 한 사람인 [Roy Fielding](https://en.wikipedia.org/wiki/Roy_Fielding)은 현재의 아키텍처가 WEB의 본래 설계의 우수성을 많이 사용하지 못하고 있다고 생각했다. 로이는 ___"WEB의 장점을 최대한 활용하며 어떻게 독립적으로 WEB을 보존하면서 진보시킬 수 있을까?"___ 고민끝에 REST의 기반이 되는 _HTTP Object Model_ 이론을 마이크로소프트 Research에서 발표하게 되면서 시작됐다. 2년 뒤 2000년에 박사과정중이었던 로이는 현재 우리가 알고있는 "Representation State Transfer"를 120Page 분량의 박사논문에 정의하여 발표하게된다.

- 선택  
WEB에 API 개념이 생겨나면서 마이크로소프트는 XML을 이용하여 원격으로 다른 시스템에 메소드를 호출할 수 있는 [프로토콜(XML-RPC)](https://ko.wikipedia.org/wiki/XML-RPC)를 만들고 [SOAP](https://ko.wikipedia.org/wiki/SOAP)(Simple Object Access Protocol)란 이름으로 변경한다. 2000년 SOAP를 이용한 salesforce API가 나타나고 4년후 REST를 이용한 flickr API와 비교가 되며 REST의 단순하고 규칙적고 쉬운 환경에 업계는 서비스의 대부분의 API를 REST로 대체한다. EMC와 IBM, 마이크로소프트 등 글로벌 거대 벤더들이 웹서비스와 웹 2.0 인터페이스를 기반으로 공동 개발한 CMIS(Content Management Interoperability Services: 콘텐츠 관리 상호 운용 서비스) 규격 을 발표하고, 마이크로소프트에서는 우리가 흔히 알고있는 REST API
의 가이드라인을 만들어 발표하기까지 이른다.

	``` 
	- uri는 http://{sericeRoot}/{collection}/{id} 형식이어야함
	- GET,PUT,DELETE,POST,HEAD,PATCH,OPTIONS 를 지원해야함  
	- API 버저닝은 Major.minor로 하고 uri에 버전 정보를 포함시킨다. 등등..
	```
하지만.. 필딩은 모두 REST가 아니다고 얘기한다. 뭐지.. REST 아키텍처 스타일을 따르지 않았단다. REST API를 위한 최고의 버저닝 전략은 버저닝을 안하는것이며, REST API는 하이퍼텍스트 기반이어야 한다는것이다. 위언급했듯 REST는 분산 하이퍼 미디어 시스템(WEB) 을 위한 아키텍처 스타일이다. 아키텍처 스타일은 일종의 제약조건들의 집합인데 이 요소들을 하이브리드 아키텍처스타일로 구성되어야한다는것이다. REST 구성의 제약조건은 다음과 같다.
 - __client-server__ : 자원을 갖고 있는 Server와 그 자원을 요청하는 Client 관계에서 서버는 API를 제공하고 비즈니스 로직 처리 및 데이터저장의 책임을 지니고, 클라이언트에서 컨텍스트정보들은 자기가 직접관리하여 서로간의 의존성이 줄어든다.
 - __stateless__ : Client의 컨텍스트정보들을 Server에 저장하지 않아서 세션과 쿠키와 같은 context 정보를 신경쓰지 않아도 되므로 구현이 단순해진다.
 - __cacheable__ : WEB에서 사용하는 기존의 인프라를 그대로 활용가능.
 - __uniform interface__ : HTTP 표준에만 따른다면, 어떠한 기술이든 적용 가능.
 		- identification of resources : 리소스가 uri로 식별기능.
		- manipuation of resources through representations : 메세지로 리소스 CRUD제어가능.
		- self-descriptive messages : 메세지는 스스로 설명 할 줄 알아야함.
		- hypermedia as the engine of application state(HATEOAS) : 애플리케이션의 상태는 Hyperlink를 이용해 전이 되어야한다.
 - __layered system__ : Client ► 보안,로드밸런싱,암호화,사용자인증 등 유연한 구조가 가능. ► 순수 비지니스로직 API Server ► 다중계층으로 구성가능.
 - __code-on-demand__(optional) : 서버의 코드를 클라이언트에서 운영가능.(ex. 자바스크립트)

로이필딩은 현재 위 REST 제약조건에 따르지않고 HTTP API 형태 임에도 불구 명칭을 REST-API라고 칭하는 관계자들에게 제발 제약도건에 따르던제 아니면 다른단어를 써라라고 전달했다. 하지만 사실 구조가체가 다르다. REST를 따를수있는 대샹이 웹페이지와 HTTP API 는 기계와 사람이기때문에 mediaType 이 시각적인 측면에서 html과 json으로 구분되기 때문이다. 
예를들어 HTML의 Self-descriptive는 응답메세지의 Content-Type이 text/html임을 확인이 되면 HTTP 명세에 media type IANA에 등록되어있고, IANA에서 text/html의 설명을 찾는다. text/html 명세에는 http://www.w3.org/TR/html 의 링크에 명세 해석이 가능하다. 
HATEOAS 조건또한 ```<a href=""></a>``` 태그를 이용해 표현된 링크를 통해 다음상태로 전달될 수 있으므로 HATEOAS를 만족한다. 반면에 JSON도 media type이 application/json 으로 표현되어 IANA에 등록도 되어있고 관련 링크를 찾아가 json 문서를 파싱하여 해석도가능하다. 하지만 내용의 정확한 속성 ```ex) [{"op":remove,"path":"/a/s/d"}]``` "op"나 "path"의 각 의미들이 무엇을 의미하는지는 알수없어서 온전한 해석이 불가능하기 때문에 Self-descriptive 하지않고 다음 상태로 전이 할 링크가 없으므로 HATEOAS 또한 하지않다.

REST는 SOAP보다 사용하기 쉽고 간편한줄만 알았는데 복잡하고 어렵다. 사실 REST 제약조건을 모두 맞춰가며 시스템을 구현하는 RESTful 사례가 적다. 그래서 이렇게 굳이 제약조건을 전부 따져가며 구축을해야 하는가? 로이필드는 "시스템 전체를 통제(서버와 클라이언트를 전체를 제어)할 수 있다고 생각하거나, 진화에 관심이 없다면 REST에 대해 따지느라 시간을 낭비하지 말았으면한다."라고 답변했다. 

> ___그럼 Self-descriptive 와 HATEOAS를 만족할 수 있는방법은?___

- Self-descriptive은 Media type을 IANA에 등록 하여 custom media type이나 profile link relation 등으로 만족.
- HATEOAS는 HTTP 헤더나 본문에 __링크__ 를 담아 REST의 조건을 만족.

Media type을 IANA에 등록을 꼭 해야하는건 아니다. 하지만 등록을하면 여러이점이있는데 누가나 쉽게 사용할 수 있고, 이름 충돌을 피할 수 있으며, 등록은 어렵지 않다고? 한다.

오늘날의 RESTAPI는 REST를 거의 따르지않는다. 제약조건중에서 특히 __Self-descriptive__ 와 __HATEOAS__ 를 만족하기 힘들기 때문이다. 그래도 REST를 따르며 API를 설계하겠다면 조건을 만족시켜 설계하시기 바랍니다.

## References  
- [https://ko.wikipedia.org/wiki/REST](https://ko.wikipedia.org/wiki/REST)  
- [https://blog.toss.im/2017/10/25/team/people/toss-developer-deview-conference/](https://blog.toss.im/2017/10/25/team/people/toss-developer-deview-conference/)
- [https://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven](https://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)