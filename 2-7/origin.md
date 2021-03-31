# 7장 캐시

### 웹 캐시란?

- 자주 쓰이는 문서의 사본을 자동으로 보관하는 HTTP 장치이다.

웹 **캐시의 이점**

- 불필요한 데이터 전송을 줄인다.
- 네트워크 병목을 줄여준다.
- 원 서버에 대한 요청을 줄여준다.
- 거리로 인한 지연을 줄여준다.

## 7.1 불필요한 데이터 전송

복수의 클라이언트가 자주 사용되는 서버로 접근할 때, 똑같은 문서를 각각 클라이언트들에게 전달하면 낭비

(대역폭, Latency, 서버에 부하)

→ 따라서, 캐시를 이용하면 낭비가 줄어들게 된다.

## 7.2 대역폭 병목

![Blank_diagram_-_Page_2](https://user-images.githubusercontent.com/26040955/113157600-00abfe00-9276-11eb-897c-ce0e5bb1af3a.png)

캐시는 네트워크 병목을 줄여준다.

→ 원래 서버로부터 계속 가져온다면, 1.4Mbit/sec

→ LAN 캐시를 사용할 경우 100Mbit/sec

## 7.3 갑작스런 요청 쉐도(Flash Crowds)

캐싱은 갑작스런 요청 쉐도에 대처하기 위해 특히 중요하다.

→ 갑작스런 사건으로 인해 많은 사람이 거의 동시에 웹 문서에 접근할 때 이런 일 발생

## 7.4 거리로 인한 지연

대역폭이 문제가 되지 않더라도, 거리가 문제 될 수 있다. 모든 네트워크 라우터는 제각각 인터넷 트래픽을 지연시킨다. 

//그림 추가

## 7.5 적중과 부적중

Cache Hit: 요청이 도착했을 때, 그에 맞는 사본이 있으면 이를 이용하여 사본 처리

Cache Miss: 캐시에 원하는 사본이 없을 때.

### 7.5.1 재검사(Revalidation)

원 서버 콘텐츠는 변경될 수 있기 때문에, 캐시는 반드시 그들이 갖고 있는 사본이 여전히 최신인지 서버를 통해 때때로 점검해야 한다. 

→ 대부분의 캐시는 클라이언트가 사본을 요청하였으며 그 사본이 검사를 할 필요가 있을 정도로 충분히 오래된 경우에만 재검사

![image](https://user-images.githubusercontent.com/26040955/113158341-acede480-9276-11eb-83f3-357d4f0db99a.png)

Cache Hit: 304 Not Modified

Cache Miss: 200 OK

객체 삭제: 404 Not Found → 이후에 Client에서 캐시 사본 삭제

### 7.5.2 문서 적중률

100%에 근접하게 되는 것이 좋지만, 적중률 40%면 웹 캐시로 괜찮다.

측정 방법: 캐시에서 사본을 가져온 % 측정

→ 얼마나 많은 웹 트랜잭션을 외부로 내보내지 않았는지 보여준다. 문서 적중률을 개선하면 전체 대기 시간이 줄어든다.(트랜잭션에는 고정된 소요 시간 TCP Connection 맺는 경우 등이 있기에..)

### 7.5.3 바이트 적중률

문서들이 모두 같은 크기가 아니기 때문에 문서 적중률이 모든 것을 말해주지는 않는다.

몇몇 큰 객체는 덜 접근되지만 그 크기 때문에 전체 트래픽에는 더 크게 기여한다.

측정 방법: 캐시를 통해 제공된 모든 바이트의 비율을 표현

→ 얼마나 많은 바이트가 인터넷으로 나가지 않았는지 보여준다. 바이트 단위 적중률의 개선은 대역폭 절약을 최적화한다.

### 7.5.4 적중과 부적중의 구별

재검사말고 일반적으로 HTTP는 클라이언트에게 응답이 캐시 적중이었는지 원 서버 접근인지 말해줄 수 있는 방법을 제공하지 않는다.

→ 응답의 Date 헤더 값을 현재 시각과 비교할 수 있겠다..

→ or Age 헤더 이용 //부록 참고

## 7.6 캐시 토폴로지
![Untitled 1](https://user-images.githubusercontent.com/26040955/113157665-10c3dd80-9276-11eb-8dca-42dd0e3d1e18.png)

Private Cache: 한 명에게만 할당된 캐시 → 브라우저 캐시

Public cache: 공유된 캐시 → 캐시 프락시 서버에서 제공
ex) [https://www.akamai.com/kr/ko/cdn/what-is-a-cdn.jsp](https://www.akamai.com/kr/ko/cdn/what-is-a-cdn.jsp)

![Untitled](https://user-images.githubusercontent.com/26040955/113157688-17525500-9276-11eb-8639-9b7f9178ee30.png)

### **Cache-Control**

**No Cache** → Cache-Control: no-store

**Shared Cache** → Cache-Control: public (Local Cache여도 상관 없음)

**Local Cache** → Cache-Control: private

### Proxy Cache 계층

작은 캐시에서 캐시 부적중이 발생했을 때 더 큰 부모 캐시가 그 '걸러 남겨진' 트래픽을 처리하도록 하는 계층을 만드는 방식이 합리적인 경우가 많다.

![image](https://user-images.githubusercontent.com/26040955/113158460-c4c56880-9276-11eb-93e8-7002ec94b672.png)


## 참고

HTTP Caching: [https://developer.mozilla.org/ko/docs/Web/HTTP/Caching](https://developer.mozilla.org/ko/docs/Web/HTTP/Caching)
