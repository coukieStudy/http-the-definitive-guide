# 3장 HTTP 메시지

## 3.4 상태 코드

#### 3.4.1 100-199 정보성 상태 코드

정보성 상태 코드는 HTTP/1.1에서 도입되었다.

- 100 Continue
  : 요청의 시작 부분이 일부가 받아 들여 졌으며, 클라이언트는 나머지를 계속 이어서 보내야 함을 의미
- 101 Switching Protocols
  : 클라이언트가 Upgrade 헤더에 나열한 것 중 하나로 서버가 프로토콜을 바꾸었음을 의미

100 Continue 상태 코드는 약간 혼란스럽다. 100 Continue는 HTTP 클라이언트 애플리케이션이 서버에 엔터티 본문을 전송하기 전에 그 엔터티 본문을 서버가 받아들일 것인지 확인하려고 할 때, 그 확인 작업을 최적화하기 위한 의도로 도입된 것이다.



**클라이언트와 100 Continue**

클라이언트는 100 Continue응답을 받고 싶다면 값을 100 Continue로 하는 Expect 요청 헤더를 보낼 필요가 있다. 만약 클라이언트가 엔터티를 보내지 않으려 한다면, 엔터티를 보낼 것이라고 생각하게 만들어 서버를 혼란에 빠뜨릴 뿐이기 때문이다.(?!) 따라서 클라이언트도 서버도 적당히 알아서 잘 처리해야한다. Ex) time out



**서버와 100 Continue**

서버가 100 Continue 값이 담긴 Expect 헤더가 포함된 요청을 받는다면, 100 Continue 응답 혹은 에러 코드로 답해야한다. 서버는 절대로 100 Continue 응답을 받을 것을 의도하지 않은 클라이언트에게 100 Continue 상태 코드를 보내서는 안 된다.(그렇게 하면 어떻게 되지..?)

서버가 100 Continue 응답을 보내기 전에 엔티티의 일부(혹은 전체)를 수신하면, 서버는 100 Continue를 보낼 필요가 없다. 클라이언트가 이미 계속 전송하기로 결정하였기 때문이다.

서버가 100 Continue 응담을 받을 것을 의도한 요청을 받고 난 상태에서, 엔터티 본문을 읽기 전에 요청을 끝내기로 결정 했다면, 서버는 그냥 응답을 보내고 연결을 닫아서는 안 된다. 클라이언트가 응답을 받을 수 없게 되기 때문이다.(4장의 "TCP 끊기와 리셋 에러" 참고)



**프락시와 100 Continue**

프락시는 버전에 따라 100 Continue 응답을 의도한 요청을 처리해야한다. 다음 홉(next-hop) 서버(6장 참고)가 HTTP/1.1 이상 혹은 어떤 버전인지 모른다면, 요청을 전달하고 1.1미만의 버전이면  417 Expectation Failed 에러로 응답해야한다.

이를 위해 다음 홉 서버들에 대한 상태 등을 캐싱하면 프락시에 도움이 된다.

- 100 Continue
: 요청의 시작 부분이 일부가 받아 들여 졌으며, 클라이언트는 나머지를 계속 이어서 보내야 함을 의미
- 101 Switching Protocols
: 클라이언트가 Upgrade 헤더에 나열한 것 중 하나로 서버가 프로토콜을 바꾸었음을 의미

#### 3.4.2 200-299 성공 상태 코드

- **200 OK**
  : 요청은 정상이고, 엔티티 본문은 요청된 리소스를 포함하고 있음
- **201 Created**
  : 서버 리소스를 생성 요청(예: POST)에 대한 응답
- 202 Accepted
  : 요청은 받아 들여졌으나 서버는 아직 그에 대한 어떤 동작도 수행하지 않았음
- 203 Non-Authoritative Information
- **204 No Content**
  - 응답 메시지는 헤더와 상태줄을 포함하지만 엔티티 본문은 포함하지 않음
  - DELETE와 같이 실행 후 반환 할 값이 없는 경우 사용
  - GET 요청 시 리소스가 없는 경우에는 200과 함께 빈 값을 반환
- 205 Reset Content
- 206 Partial Content


#### 3.4.3 300-399 리다이렉션 상태 코드

리다이렉션 상태 코드는 클라이언트가 관심있어 하는 리소스에 대해 다른 위치를 사용하라고 말해주거나 그 리소스의 내용 대신 다른 대안 응답을 제공한다.

- 300 Multiple Choices
  : 클라이언트가 동시에 여러 리소스를 가리키는 URL을 요청한 경우, 그 리소스의 목록과 함께 반환
- **301 Moved Permanently**
  : 요청한 URL이 옮겨졌을 때 사용
- 302 Found
- 303 See Other
- **304 Not Modified**
  : GET과 함께 수정 일에 대한 조건부 헤더를 보낸 경우. 수정되지 않은 경우 사용
- 305 Use Proxy
- 307 Temporary Redirect

#### 3.4.4 400-499 클라이언트 에러 상태 코드

클라이언트가 서버에서 처리할 수 없는 요청 메세지를 보냈을 경우에 사용하는 코드다.

- 400 Bad Request
  : 클라이언트가 잘못 된 요청을 보냈다고 말해줌
- **401 Unauthorized**
  : 인증이 필요하다고 알려줌
- 402 Payment Required
- **403 Forbidden**
  : 요청이 서버에 의해 거부 되었음을 알려줌. 거부 된 이유는 본문에 포함시킬 수 있음
- **404 Not Found**
  : 서버가 요청한 URL
- **404 Not Found**
  : 서버가 요청한 URL을 찾을 수 없을 때 사용
- **405 Method Not Allowed**
  : 요청한 URL에 대해, 지원하지 않는 메서드로 요청 받았을 때 사용
- 406 Not Acceptable
- 407 Proxy Authentication Required
- **408 Request Timeout**
  : 클라이언트의 요청을 완수하기에 시간이 너무 많이 걸리는 경우, 서버는 이 상태 코드로 응답하고 연결을 끊을 수 있음
- 409 Conflict
- 410 Gone
- 411 Length Required
- 412 Precondition Failed
- 413 Request Entity Too Large
- 414 Request URI Too Long
- 415 Unsupported Media Type
- 415 Requested Range Not Satisfiable
- 417 Expectation Failed
- **422 Unprocessable Entity**
  : 해석 가능한 Content-Type의 엔티티이지만, 데이터 필드 등이 유효하지 않은 경우 (하지만 표준은 아님)
  
#### 3.4.5 500-599 서버 에러 상태 코드

서버 자체에서 발생하는 에러 코드다.

- **500 Internal Server Error**
  : 서버가 요청을 처리할 수 없게 만드는 에러는 만났을 때 사용
- 501 Not Implemented
- **502 Bad Gateway**
  : Gateway로 부터 올바른 응답을 받지 못했을 때 사용
- **503 Service Unavailable**
  : 현재는 서버가 요청을 처리해 줄 수 없지만 나중에는 가능함을 의미
- **504 Gateway Timeout**
  : 다른 서버에게 요청을 보내고 응답을 기다리다가 타임아웃이 발생한 Gateway나 Proxy에서 온 응답
- 505 HTTP Version Not Supported
  : 서버가 지원할 수 없거나 지원하지 않으려고 하는 버전의 프로토콜로 된 요청을 받았을 때

## 3.5 헤더

헤더는 크게 아래 다섯가지로 분류 가능

- **일반 헤더**

  클라이언트와 서버 모두 사용하는 헤더. Ex) Date: Tue, 3 oct 1974 02:16:00 GMT


- **요청 헤더**

  요청 메세지에서만 사용하는 헤더. Ex) Accept : */* (어떤 미디어 타입이든 받을 수 있다고 서버에 알려줌)
  - **Host : 요청의 대상이 되는 서버의 호스트 명과 포트**
  - **Referer : 현재의 요청 URL이 들어있었던 문서의 URL**
  - **User-Agent : 요청을 보낸 애플리케이션의 이름**

- **응답 헤더**

  응답 메세지에서만 사용하는 헤더. Ex) Server: Tiki-Hut/1.0 (클라이언트에게 서버의 종류와 버전을 알려줌)
  - Accept : 서버가 보내도 되는 미디어 종류
  - Accept-Charset : 서버가 보내도 되는 문자 집합
  - Accept-Encoding : 서버가 보내도 되는 인코딩
  - Aceept-Language : 서버가 보내도 되는 언어

- **엔터티 헤더**

  엔터티 본문에 대한 헤더. Ex) Content-Type : text/html; charset=iso-latin-1 (엔터티가 iso-latin-1 문자집합으로 된 HTML문서임을 알려줌)

- **확장 헤더**

  비표준 헤더.
  
  https://camo.githubusercontent.com/d09839bf7ae593fa403793326a9af335e9392d622f89ea3ee13b889c02ece2fc/68747470733a2f2f7261776769746875622e636f6d2f666f722d4745542f687474702d6465636973696f6e2d6469616772616d2f6d61737465722f6874747064642e706e67
  

#### 3.5.1 일반 헤더
- Date : 메시지가 언제 만들어 졌는지에 대한 날짜와 시간을 제공
- Upgrade : 발송자가 '업그레이드'하길 원하는 새 버전이나 프로토콜을 알려줌
- **Via : 이 메시지가 어떤 중개자(Proxy, Gateway)를 거쳐 왔는지 보여줌**

#### 3.5.2 요청 헤더
- **Host : 요청의 대상이 되는 서버의 호스트 명과 포트**
- **Referer : 현재의 요청 URL이 들어있었던 문서의 URL**
- **User-Agent : 요청을 보낸 애플리케이션의 이름**

**Accept 관련 헤더**
- Accept : 서버가 보내도 되는 미디어 종류
- Accept-Charset : 서버가 보내도 되는 문자 집합
- Accept-Encoding : 서버가 보내도 되는 인코딩
- Aceept-Language : 서버가 보내도 되는 언어

**조건부 요청헤더**
- Expect : 클라이언트가 요청에 필요한 서버의 행동을 열거할 수 있게 해줌
- If-Match : 문서의 엔티티 태그가 주어진 엔티티 태그와 일치하는 경우에만 문서를 가져옴
- If-None-Match : 문서의 엔티티 태그가 주어진 엔티티 태그와 일치하지 않는 경우에만 문서를 가져옴
- **If-Modified-Since : 주어진 날짜 이후에 리소스가 변경 된 경우에만 리소스를 가져옴**
- If-Unmodified-Since : 주어진 날짜 이후에 리소스가 변경되지 않은 경우에만 리소스를 가져옴
- If-Range : 문서의 특정 범위에 대한 요청을 할 수 있게 해줌
- Range : 서버가 범위 요청을 지원한다면, 리소스에 대한 특정 범위를 요청

**요청 보안 헤더**
- **Authorization : 클라이언트가 서버에게 제공하는 인증 그 자체에 대한 정보 (예: JWT Token, OAuth Token 등)**
- **Cookie : 클라이언트가 서버에게 토큰을 전달할 때 사용**
- Cookie2 : 요청자가 지원하는 쿠키의 버전을 알려줄 때 사용

#### 3.5.3 응답 헤더
- Accept-Ranges : 서버가 자원에 대해 받아들일 수 있는 범위의 형태
- Vary : 서버가 확인해 보아야 하고 그렇기 때문에 응답에 영향을 줄 수 있는 헤더들의 목록. 예) 권한 관리를 위한 Access-Control-Request-Method, Access-Control-Request-Headers, Origin

**응답 보안 헤더**
- Proxy-Authenticate : Proxy에서 클라이언트로 보낸 인증요구의 목록
- Set-Cookie : 서버가 클라이언트를 인증할 수 있도록 클라이언트 측에 토큰을 설정하기 위해 사용
- WWW-Authenticate : 서버에서 클라이언트로 보낸 인증요구 목록

#### 3.5.4 엔터티 헤더
- Allow : 이 엔티티에 대해 수행될 수 있는 요청 메서드 목록
- Location : 클라이언트에게 엔티티가 어디에 위치하고 있는지 알려줌

**콘텐츠 헤더**
- Content-Base : 본문에서 사용 된 상대 URL을 계산하기 위한 Base URL
- Content-Encoding : 본문에 적용 된 인코딩
- Content-Language : 본문을 이해하기 위한 언어
- Content-Length : 본문의 길이나 크기
- Content-Location : 리소스의 실제 위치
- Content-MD5 : 본문의 MD5 체크섬
- Content-Range : 전체 리소스에서 이 엔티티가 해당하는 범위를 바이트 단위로 표현
- Content-Type : 본문의 객체 종류

**엔터티 캐싱 헤더**
- ETag : 엔티티에 대한 엔티티 태그
- Expires : 엔티티의 만료 일시
- Last-Modified : 가장 최근 엔티티가 변경된 일시


