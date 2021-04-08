
## 11장 클라이언트 식별과 쿠키
- 서버가 서로 다른 클라이언트를 어떻게 구분하고 추적할 것인지
- HTTP 자체는 익명, stateless 하기 때문에 추가 정보가 필요하다
- Stateless한 HTTP에서 세션을 유지하기 위해서는 (ex. 장바구니) 클라이언트 식별이 필수적이다
- 이를 위한 여러가지 방법을 공부

### 11.2 HTTP 헤더
- 사용자 정보 전달에 사용하는 헤더의 종류
  - From
    - 클라이언트의 이메일 주소
  - User-Agent
    - 브라우저 정보
    - 형식이 표준화되어있지 않아, 완벽한 파싱은 힘들다
    - **현재 프라이버시 이슈로 퇴출하려고 크롬에서 앞장서는 중**
    - Client Hint 를 대체품으로 권고
    - https://d2.naver.com/helloworld/6532276
  - Referer
    - 클라이언트가 현재 링크를 접속하게 된 이전 페이지
  - Authorization
  - Client-ip
  - X-Forwarded-For
  - Cookie

### 11.3 클라이언트 IP 주소
HTTP에서는 없지만, 그 하위 레이어인 TCP/IP 레이어를 통해 알 수 있다

단점
- 한 컴퓨터로 여러 사용자가 이용시 구별할 수 없다
- ISP(SK 브로드밴드와 같은 인터넷 제공 기업)은 동적으로 IP를 바꾼다
- IP주소 부족 및 보안 문제로 NAT 방화벽 사용시 실제 내부 private ip들은 모두 하나의 public 아이피로 변환된다
  - https://velog.io/@hidaehyunlee/%EA%B3%B5%EC%9D%B8Public-%EC%82%AC%EC%84%A4Private-IP%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90
  - ex. 172.16.0.0:80 -> 201.1.1.1:100, 172.16.0.1:80 -> 201.1.1.1:101
- HTTP 프락시 서버나 게이트웨이도 새로운 TCP를 연결하여 프락시 서버의 IP가 전달된다
  - 이 때 클라이언트의 아이피 전달을 위해 Client-ip, X-Forwarded-For 을 사용한다
- 한 마디로 **접근권한 제어**에 사용하기에는 불명확하다

### 11.4 사용자 로그인
- 사용자 인증의 기본은 아이디와 비밀번호를 통한 로그인
- WWW-Authenticate, Authorization 헤더를 HTTP에서 제공, 사용자의 이름을 전달
  - Authentication : 클라이언트가 주장하는 사용자가 맞는지 확인 (로그인)
  - Authorization : 사용자가 특정 권한이 있는지 확인
  - **단 암호화를 제공하지는 않는다**
- 클라이언트가 접근할 때, 로그인이 필요하다면 `401 Login Required` 응답을 보낼 수 있다
  - http://iloveulhj.github.io/images/basic-auth/basic-auth2.png
  - https://mc.coupang.com/ssr/desktop/order/list
- 로그인이 성공한 이후에는 식별정보 토큰을 Authorization 헤더에 담아 session 유지

### 11.5 뚱뚱한 URL
- 사용자의 상태정보를 포함하는 url을 동적으로 생성하여 사용

문제점
- 긴 URL
- 공유할 수 없는 URL
  - URL에 개인 상태정보가 포함되기 때문에, 개인정보도 같이 공유될 수 있다
- 캐시 사용 불가
- 이탈
  - URL만을 이용하여 세션을 유지할 경우, 세션을 이탈하기 쉽다
- 세션 간 지속성의 부재
  - 로그아웃 시 모든 정보를 잃는다

### 11.6 쿠키
- 쿠키의 타입
  - session cookie
    - 세션을 위한 쿠키로 브라우저 닫으면 삭제
    - Discard 파라미터가 있을 때
    - 혹은 Expires, Max-Agerk 가 없을 떄
  - persistent cookie
    - 디스크에 저장되어 계속 유지
- 쿠키의 형식
  - `Set-Cookie`혹은 `Set-Cookie2`헤더 사용 -> `Set-Cookie2`사용 이제 안함
  - key=value 의 구조
  - `NM_THEME_EDIT=; BMR=s=1617857251356&r=https://m.blog.naver.com/PostView.nhn?blogId=windkiy&logNo=150019086403&proxyReferer=https:%2F%2Fwww.google.com%2F&r2=https://www.google.com/; PM_CK_loc=e6d5fd2724d71735e7b78d0e29e6816836b53396669f13e25ea675b6795bea8a`

https://lh3.googleusercontent.com/proxy/vQF72iNdd2SyZqiZ9YBwXdPhdPotVyIFIddyE8zO240xDt7Qa29Jk0WDfphlmEdcpzHVDcDJg2W2MvqPdahsYRGJ8eDhjLNHi2aYq0FHi5AH1YRF5RqSDoaQ4PkXd2QLEHBHopwOfAzbqpmpIoXvzULukDJXVGE8QECzoTruNQFy7eNFqn-QR6A7sB_-_bY

- 각 브라우저는 서로 다른 방식으로 쿠키를 저장
  - 크롬은 SQLite
- 브라우저는 보통 쿠키를 생성한 서버에게만 쿠키를 보낸다
  - 어떤 사이트에 어떤 쿠키를 보낼지를 여러 속성을 이용해 정할 수 있다
- 쿠키의 속성
  - domain
    - `domain`이라는 키를 이용해서 특정 도메인에서 쿠키를 읽을 수 있게 제어할 수 있다
  - path
    - 도메인의 특정 path에 접근할 때 쿠키를 전달한다
  - secure
    - https일 경우만 사용한다
  - expires
    - `요일, DD-MM YY HH:MM:SS GMT`형식

- 쿠키를 이용한 세션 유지 예시
https://bebiangel.github.io/images/http-guide-chap11.png

- 쿠키와 캐싱 주의점
  - 캐시되지 말아야 할 문서는 표기
    - Control: no-cache="Set-Cookie" -> 쿠키를 제외하고 문서만 캐싱
  - Set-Cookie 헤더를 캐시하는 것에 유의
  - Cookie 헤더를 가지고 있는 request 주의
    - 보수적으로 유지할 때는 Cookie 헤더 포함 요청의 응답은 캐시하지 않는다

### More
- Cookie vs (Bearer) Token vs JWT
  - Cookie는 앞서 배운 것
    - https://t1.daumcdn.net/cfile/tistory/994BEA345B53368401
  - Bearer Token은 Authorization 헤더에 사용하는 것
    - https://t1.daumcdn.net/cfile/tistory/995EC2345B53368912
    - Stateless하다는 장점이 존재
  - JWT는 값을 인코딩 하는 방식

### Reference
- https://tansfil.tistory.com/58
- https://stackoverflow.com/questions/37582444/jwt-vs-cookies-for-token-based-authentication
- https://dzone.com/articles/cookies-vs-tokens-the-definitive-guide
  - User-Agent
    - 브라우저 정보
    - 형식이 표준화되어있지 않아, 완벽한 파싱은 힘들다
    - **현재 프라이버시 이슈로 퇴출하려고 크롬에서 앞장서는 중**
    - Client Hint 를 대체품으로 권고
    - https://d2.naver.com/helloworld/6532276
  - Referer
    - 클라이언트가 현재 링크를 접속하게 된 이전 페이지
  - Authorization
  - Client-ip
  - X-Forwarded-For
  - Cookie

### 11.3 클라이언트 IP 주소
HTTP에서는 없지만, 그 하위 레이어인 TCP/IP 레이어를 통해 알 수 있다

단점
- 한 컴퓨터로 여러 사용자가 이용시 구별할 수 없다
- ISP(SK 브로드밴드와 같은 인터넷 제공 기업)은 동적으로 IP를 바꾼다
- IP주소 부족 및 보안 문제로 NAT 방화벽 사용시 실제 내부 private ip들은 모두 하나의 public 아이피로 변환된다
  - https://velog.io/@hidaehyunlee/%EA%B3%B5%EC%9D%B8Public-%EC%82%AC%EC%84%A4Private-IP%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90
  - ex. 172.16.0.0:80 -> 201.1.1.1:100, 172.16.0.1:80 -> 201.1.1.1:101
- HTTP 프락시 서버나 게이트웨이도 새로운 TCP를 연결하여 프락시 서버의 IP가 전달된다
  - 이 때 클라이언트의 아이피 전달을 위해 Client-ip, X-Forwarded-For 을 사용한다
- 한 마디로 **접근권한 제어**에 사용하기에는 불명확하다

### 11.4 사용자 로그인
- 사용자 인증의 기본은 아이디와 비밀번호를 통한 로그인
- WWW-Authenticate, Authorization 헤더를 HTTP에서 제공, 사용자의 이름을 전달
  - Authentication : 클라이언트가 주장하는 사용자가 맞는지 확인 (로그인)
  - Authorization : 사용자가 특정 권한이 있는지 확인
  - **단 암호화를 제공하지는 않는다**
- 클라이언트가 접근할 때, 로그인이 필요하다면 `401 Login Required` 응답을 보낼 수 있다
  - http://iloveulhj.github.io/images/basic-auth/basic-auth2.png
  - https://mc.coupang.com/ssr/desktop/order/list
- 로그인이 성공한 이후에는 식별정보 토큰을 Authorization 헤더에 담아 session 유지

### 11.5 뚱뚱한 URL
- 사용자의 상태정보를 포함하는 url을 동적으로 생성하여 사용

문제점
- 긴 URL
- 공유할 수 없는 URL
  - URL에 개인 상태정보가 포함되기 때문에, 개인정보도 같이 공유될 수 있다
- 캐시 사용 불가
- 이탈
  - URL만을 이용하여 세션을 유지할 경우, 세션을 이탈하기 쉽다
- 세션 간 지속성의 부재
  - 로그아웃 시 모든 정보를 잃는다

### 11.6 쿠키
- 쿠키의 타입
  - session cookie
    - 세션을 위한 쿠키로 브라우저 닫으면 삭제
    - Discard 파라미터가 있을 때
    - 혹은 Expires, Max-Agerk 가 없을 떄
  - persistent cookie
    - 디스크에 저장되어 계속 유지
- 쿠키의 형식
  - `Set-Cookie`혹은 `Set-Cookie2`헤더 사용 -> `Set-Cookie2`사용 이제 안함
  - key=value 의 구조
  - `NM_THEME_EDIT=; BMR=s=1617857251356&r=https://m.blog.naver.com/PostView.nhn?blogId=windkiy&logNo=150019086403&proxyReferer=https:%2F%2Fwww.google.com%2F&r2=https://www.google.com/; PM_CK_loc=e6d5fd2724d71735e7b78d0e29e6816836b53396669f13e25ea675b6795bea8a`

https://lh3.googleusercontent.com/proxy/vQF72iNdd2SyZqiZ9YBwXdPhdPotVyIFIddyE8zO240xDt7Qa29Jk0WDfphlmEdcpzHVDcDJg2W2MvqPdahsYRGJ8eDhjLNHi2aYq0FHi5AH1YRF5RqSDoaQ4PkXd2QLEHBHopwOfAzbqpmpIoXvzULukDJXVGE8QECzoTruNQFy7eNFqn-QR6A7sB_-_bY

- 각 브라우저는 서로 다른 방식으로 쿠키를 저장
  - 크롬은 SQLite
- 브라우저는 보통 쿠키를 생성한 서버에게만 쿠키를 보낸다
  - 어떤 사이트에 어떤 쿠키를 보낼지를 여러 속성을 이용해 정할 수 있다
- 쿠키의 속성
  - domain
    - `domain`이라는 키를 이용해서 특정 도메인에서 쿠키를 읽을 수 있게 제어할 수 있다
  - path
    - 도메인의 특정 path에 접근할 때 쿠키를 전달한다
  - secure
    - https일 경우만 사용한다
  - expires
    - `요일, DD-MM YY HH:MM:SS GMT`형식

- 쿠키를 이용한 세션 유지 예시
https://bebiangel.github.io/images/http-guide-chap11.png

- 쿠키와 캐싱 주의점
  - 캐시되지 말아야 할 문서는 표기
    - Control: no-cache="Set-Cookie" -> 쿠키를 제외하고 문서만 캐싱
  - Set-Cookie 헤더를 캐시하는 것에 유의
  - Cookie 헤더를 가지고 있는 request 주의
    - 보수적으로 유지할 때는 Cookie 헤더 포함 요청의 응답은 캐시하지 않는다

### More
- Cookie vs (Bearer) Token vs JWT
  - Cookie는 앞서 배운 것
    - https://t1.daumcdn.net/cfile/tistory/994BEA345B53368401
  - Bearer Token은 Authorization 헤더에 사용하는 것
    - https://t1.daumcdn.net/cfile/tistory/995EC2345B53368912
    - Stateless하다는 장점이 존재
  - JWT는 값을 인코딩 하는 방식

### Reference
- https://tansfil.tistory.com/58
- https://stackoverflow.com/questions/37582444/jwt-vs-cookies-for-token-based-authentication
- https://dzone.com/articles/cookies-vs-tokens-the-definitive-guide
