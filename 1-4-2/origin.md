## 4.5 지속 커넥션

##### 들어가기 전에 다시 되돌아 보기
- 대부분의 http 성능은 TCP에서 나온다
- HTTP 1.0+ 에서의 해결 방법
  - 병렬 커넥션 (parallel)
  - 지속 커넥션 (persistent)
  - 파이프라인 커넥션 (pipelined)
  - 다중 커넥션 (multiplexed)

##### 지속커넥션이란
한 번 연결한 TCP 커넥션을 그대로 사용하는 것
- 한 번의 TCP 연결로 http request를 한 번만 보내고 끝내는게 아니라 더 사용하는 것

###### 장점
- TCP 커넥션의 횟수가 줄어든다
- 커넥션을 유지해서 TCP 느린 시작을 어느정도 해결할 수 있다

###### 단점
- Connection을 지속한다고 하고 사용 안하면 과도한 리소스를 사용하게 된다

##### 지속 커넥션과 병렬 커넥션
- 병렬 커넥션은 여러 서버와 동시에 가능하지만, 지속 커넥션은 한 개의 서버와만 가능하다
- 실제로는 둘 다 함께 사용한다
  - 병렬 커넥션으로 여러 커넥션을 동시에 열고, 이를 지속 커넥션으로 계속 사용

##### 지속 커넥션 사용 방법
###### Keep-Alive
- HTTP/1.0 부터 나왔고 HTTP/1.1에서는 빠졌지만, 널리 사용되고 있다
- Request
  - `Connection: Keep-Alive`를 해더를 추가하여 요청
  - Connection을 유지하기 위해서는 매번 포함해서 보내야 한다
- Response
  - `Connection: Keep-Alive`해더를 포함해서 지원한다고 응답
  - `Keep-Alive: max=5, timeout=120` 과 같이 `Keep-Alive` 헤더를 이용해서 다양한 추가 옵션을 추가
    - max : 최대 지원 트랜잭션 개수
      - 트랜잭션 : request부터 response까지 한 세트
    - timeout : 유지 시간
    - 하지만 지원한다는 보장은 없다
- 주의점
  - TCP 커넥션을 계속 사용하기 때문에 어디까지가 트랜잭션의 끝이고 시작인지 알 수 없다 따라서
    - 헤더에서 Cotent-Length + Conteny-Type으로 MIME와 그 크기를 명시핳거나
    - `Transfer-Encoding: chunked`로 청크 전송 인코딩이 되어야 한다
  - 프락시와 게이트웨이에서 Connection 헤더의 규칙을 철저히 지켜야 한다
    - **즉 프락시가 서버에 전송할 때 Connection 헤더를 버려야 한다(hop-by-hop)**
    - 안지킬시 dump proxy 문제가 존재
    - HTTP/1.0 으로 keep-alive가 오면 위의 문제가 가능하기 때문에 무시되어야 한다
    - 혹은 `Proxy-Connection: Keep-Alive`라는 비표준 헤더를 사용할 수 있다
**Dump proxy문제**
- ![dump proxy](https://goodgid.github.io/assets/img/http/HTTP-Keep-Alive_3.png)

###### HTTP/1.1 지속 커넥션
- 기본으로 활성화 되어있다
- `Connection: close`가 response에 없다면 커넥션을 유지한다고 한다
- client가 `Connection: close`를 포함해서 연결을 끊을 수 있다
- 주의점
  - Keep-Alive와 같이 content length, 혹은 청크 전송이 필요하다
  - HTTP/1.1 프록시는 클라이언트와 서버 각각에 대해 별도 지속 커넥션을 관리해야 한다
  - 또한 client가 지속커넥션을 지원하는지 확실하지 않다면 사용하면 안된다
    - client가 dump proxy일 수 있다
  - 커넥션이 중간에 끊어졌을 때 복구하고 다시 요청할 수 있어야 한다
  - 한 서버당 2개의 지속커넥션 유지를 추천

## 4.6 파이프라인 커넥션
##### 파이프라인 커넥션이란
- 지속 커넥션을 이용하여, 한 개의 트랜잭션(request + response)가 끝날 때까지 기다리는게 아니라, 응답을 받기 전에 여러 개의 요청을 하는 것
- ![pipelining](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FVUrce%2FbtqFcNm19vf%2FbQHr47gBBU4Ys6fvjMYgQ0%2Fimg.png)

##### 주의점
- 응답순서는 요청순서와 같아야 한다
- > HTTP 파이프라이닝은 모던 브라우저에서 기본적으로 활성화되어있지 않습니다:
  - https://www.chromium.org/developers/design-documents/network-stack/http-pipelining
- > GET, HEAD, PUT 그리고 DELETE 메서드같은 idempotent 메서드만 가능합니다
- 커넥션이 끊어질 경우 다시 요청을 보내야하기 때문에 idempotent한 메서드만 포함해야 한다
  - 어떤게 처리되었는지 알 수 없다 -> 이미 서버에서 처리하여 응답이 되었어도, 클라이언트에서 도착하지 않았을 수 있다
- > 파이프라이닝은 더 나은 알고리즘인 멀티플렉싱으로 대체되었는데, 이는 HTTP/2에서 사용됩니다.

## 4.7 커넥션 끊기에 대한 미스터리
- 커넥션은 서버, 클라이언트, 프락시 등 어디서든 끊을 수 있다
- 따라서 재시도해도 되는 트랜잭션은 클라이언트가 다시 커넥션을 맺고 전송을 시도해야 한다

##### 우아한 커넥션 끊기
- TCP 기본 정보
  - TCP는 Bidirectional이고 Full duplex이다
  - 따라서 TCP 커넥션 양쪽에는 입력 큐와 출력큐가 있다
- 전체 끊기
  - 입력 채널과 출력 채널을 모두 끊는 것
  - close()
- 절반 끊기
  - 입력 채널과 출력채널 중 하나만 끊는 것
  - shutdown()
- 안전하게 커넥션을 끊기 위해서는 출력 채널을 끊고 기다리는 것
- 입력 채널이 끊겼는데 데이터를 전송하면 TCP의 `connection reset by peer`오류가 발생
  - 이 때 운영체제는 버퍼에 읽히지 않은 데이터를 삭제한다

## More

### 멀티플렉싱
HTTP/2.0에 추가된 기능으로 파이프라이닝과 유사하지만, 리턴 순서가 달라도 된다는 특징이 있다
![Multiplexing](https://i.stack.imgur.com/GawCa.png)

- https://freecontent.manning.com/animation-http-1-1-vs-http-2-vs-http-2-with-push/
  - 1. Parralel + keep-alive
  - 2. multiplexing
  - 3. multiplexing + push 라는 기능으로 더 빠르게도 가능

### 청크 전송 인코딩
- 동적으로 크기가 정해지는 등, 크기를 알 수 없을 때
- 보안적 이유로 인코딩하기도 함
- ![Encoding](https://t1.daumcdn.net/cfile/tistory/23461B3D57DAB4D528)

### 개발자 도구 보기
- ConnectionId : 같은 TCP 연결일 때 같은 값을 가진다 -> 즉 TCP의 지속 커넥션을 확인할 수 있다
- protocol column을 추가해서 어떤 protocol을 사용했는지 확인 가능
- 더 자세한 분석은
  - https://webcache.googleusercontent.com/search?q=cache:JfBs3jE5OHcJ:https://groups.google.com/d/topic/google-chrome-developer-tools/OBQKVhaGE-s+&cd=4&hl=ko&ct=clnk&gl=kr

### HTTP3와 UDP
- https://evan-moon.github.io/2019/10/08/what-is-http3/

## Reference
- https://developer.mozilla.org/ko/docs/Web/HTTP/Connection_management_in_HTTP_1.x
- https://linuxism.ustd.ip.or.kr/783
- https://help.marklogic.com/knowledgebase/article/View/465/0/content-length-keepalive-and-connection-close
- https://goodgid.github.io/HTTP-Keep-Alive/
- https://evan-moon.github.io/2019/11/10/header-of-tcp/
- https://www.khanacademy.org/computing/computers-and-internet/xcae6f4a7ff015e7d:the-internet/xcae6f4a7ff015e7d:transporting-packets/a/transmission-control-protocol--tcp
- https://stackoverflow.com/questions/34184994/chrome-developer-tools-connection-id
- https://stackoverflow.com/questions/36517829/what-does-multiplexing-mean-in-http-2
- https://eminentstar.tistory.com/48
- https://freecontent.manning.com/animation-http-1-1-vs-http-2-vs-http-2-with-push/
- https://evan-moon.github.io/2019/10/08/what-is-http3/
