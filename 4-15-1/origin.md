# 15. 엔터티와 인코딩

## 15.1 메시지는 컨테이너, 엔터티는 화물

HTTP 메시지를 운송 시스템의 컨테이너라고 생각한다면, 엔터티는 메시지의 실질적인 화물이다.

**엔터티 관련 헤더**

- `Content-Type` 엔터티에 의해 전달된 객체의 종류
- `Content-Length` 전달되는 메시지의 길이나 크기
- `Content-Language` 전달되는 객체와 가장 잘 대응되는 자연어
- `Content-Encoding` 객체 데이터에 대해 행해진 변형(압축)
- `Content-Location` 요청 시점을 기준으로, 객체의 또 다른 위치
- `Content-Range` 부분 엔터티라면 전체에서 어느 부분에 해당하는지 정의
- `Content-MD5` 엔터티 본문의 컨텐츠에 대한 체크섬
- `Last-Modified` 서버에서 이 컨텐츠가 생성, 수정된 날
- `Expires` 이 엔터티 데이터가 더 이상 신선하지 않은 것으로 간주되기 시작하는 날짜와 시각
- `Allow` 이 리소스에 대해 어떤 요청 메서드가 허용되는지
- `ETag` 이 인스턴스에 대한 고유한 검사기
- `Cache-Control` 어떻게 이 문서가 캐시될 수 있는지에 대한 지시자

**엔터티 본문**

- 엔터티 본문은 `가공되지 않은 데이터`만을 담고 있다. 
- 본문에 대한 다른 정보들은 모두 헤더에 담겨 있다. 데이터의 타입, 인코딩 여부 등을 통해 데이터의 해석 방법을 다르게 가져갈 수 있다.
- 엔터티 본문은 헤더 필드의 끝을 의미하는 `빈 CRLF` 줄 바로 다음부터 시작한다.



## 15.2 Content-Length

- 메시지의 엔터티 본문의 크기를 바이트 단위로 나타낸 것.
- 압축이 된 경우라면 압축된 후의 크기를 말함.
- 청크 인코딩으로 전송하지 않는 이상, 엔터티 본문을 포함한 메세지에는 필수.
- 서버 충돌로 인해 메세지가 잘렸는지 감지, 지속 커넥션을 공유하는 메시지를 올바르게 분할하고자 할 때 필요.

### 15.2.1 잘림 검출

- 옛날 버전의 HTTP: 커넥션이 닫힌 것을 보고 메세지가 끝났음을 인지
  - Content-Length가 없다면 클라이언트는 커넥션이 정상적으로 닫힌 것인지, 메세지 전송 중에 서버에 충돌이 발생한 것인지 구분하지 못함
- 메세지 잘림은 캐싱 프락시 서버에서 특히 취약하다.
  - 캐시가 잘린 메세지를 수신했으나 잘렸다는 것을 인식하지 못했다면 캐시는 결함이 있는 콘텐츠를 저장하고 계속 제공하게 될 것이다.
  - 캐싱 프락시 서버는 명시적으로 Content-Length 헤더를 갖고 있지 않은 HTTP본문은 **보통 캐시하지 않는다.**

### 15.2.2 잘못된 Content-Length

- 더 큰 피해 유발 가능 -> 공식적으로 HTTP/1.1 사용자 에이전트는 잘못된 길이를 받고 그 사실을 인지했을 때 사용자에게 알려주게 되어 있다.

### 15.2.3 Content-Length와 지속 커넥션

- 지속커넥션으로 응답이 온 경우, 또 다른 응답이 즉시 뒤를 이을 것이다.
- 이때 Content-Length 헤더 없이는 어디까지가 엔터티 본문이고 어디서부터가 다음 메시지인지 알 수 없다.
- 예외) 청크 인코딩(15.6)

### 15.2.4 콘텐츠 인코딩

- 인코딩 된 경우 Content-Length 헤더는 인코딩된 본문의 길이를 바이트 단위로 정의한다.
- HTTP/1.1 인코딩 되지 않은 원 본문의 길이를 보내는 헤더가 없어 디코딩 검증이 어렵다.

### 15.2.5 엔터티 본문 길이 판별을 위한 규칙

아래 규칙을 순서대로 적용한다.

1. 본문을 갖는 것이 허용되지 않은 특정타입의 HTTP 메세지 Content-Length는 무시된다. ex)  HEAD
2. Transfer-Encoding 헤더를 포함하고 있다면(기본 "identity" 인코딩과 다른) 엔터티는 ‘0 바이트 청크’라 불리는 특별한 패턴으로 끝나야 한다. Content-Length는 무시된다.
3. 메세지가 Content-Length 헤더를 갖고, Transfer-Encoding 헤더가 없는 경우 본문의 길이를 담는다.
4. 'multipart/bytearnges' 미디어 타입이고 Content-Length가 정의되지 않았다면, 멀티파트 메시지의 각 부분은 각자가 스스로의 크기를 정의할  것이다.
   - 이 멀티마트 유형은 자신이 크기를 스스로 결정할 수 있는 유일한 엔터티 본문 유형이다. 따라서 수신자가 이것을 해석할 수 있다는 것을 송신자가 알기 전까지는 절대 보내면 안된다.
5. 위의 어떤 규칙에도 해당하지 않는다면, 엔터티는 커넥션이 닫힐 때 끝난다.
6. 만약 요청에는 본문이 있지만 Content-Length가 없는 경우
   - 메시지의 길이를 판별할 수 없다면 400 Bad Request 응답을 보내고
   - Content-Length를 요구하고 싶다면 411 Length Required 응답을 보낸다.



## 15.3 엔터티 요약

- 엔터티는 전송 중에 변형되는 일이 일어날 수 있다. 
- 종단 간의 데이터에 대한 체크섬을 생성할 수 있다. ex) **Content-MD5** 헤더
- 오직 처음 만든 서버만이 Content-MD5 헤더를 계산.
- 중간 프락시와 캐시는 그 헤더를 변경하거나 추가하지 않을 것이다.
- 클라이언트가 응답에 대해 기대하는 `요약 유형`을 정의할 수 있는 새로운 헤더인 **Want-Digest**를 제안했다.

```
Want-Digest: sha-256
Want-Digest: SHA-512;q=0.3, sha-256;q=1, md5;q=0
```



## 15.4 미디어 타입과 Charset

- Content-Type: 엔터티 본문의 MIME 타입(**I**nternet **A**ssigned **N**umbers **A**uthority에 등록된)을 기술한다.
- Content-Type이 원본 엔터티 본문의 미디어 타입을 명시한다는 것은 중요하다.

### 15.4.1 텍스트 매체를 위한 문자 인코딩

- Content-Type은 내용 유형을 더 자세히 지정하기 위한 선택적 매개변수도 지원한다.

```
Content-Type: text/html; charset=iso-8859-7
```

### 15.4.2 멀티파트 미디어 타입

- 서로 붙어있는 여러 개의 메세지를 포함하며, 하나의 복합 메세지로 보내진다.
- 각 구성요소는 자족적으로 자신에 대해 서술하는 헤더를 포함한다.
- 문자열 하나로 서로의 경계가 식별된다.

### 15.4.3 멀티파트 폼 제출

- `multipart/form-data`은 브라우저에서 서버로 html form의 내용을 전송 시 사용할 수 있다.

```html
<form action="http://localhost:8000/" method="post" enctype="multipart/form-data">
  <input type="text" name="myTextField">
  <input type="checkbox" name="myCheckBox">Check</input>
  <input type="file" name="myFile">
  <button>Send the file</button>
</form>
```

```
POST / HTTP/1.1
Host: localhost:8000
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=---------------------------8721656041911415653955004498
Content-Length: 465

-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myTextField"

Test
-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myCheckBox"

on
-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myFile"; filename="test.txt"
Content-Type: text/plain

Simple file.
-----------------------------8721656041911415653955004498--
```

### 15.4.4 멀티파트 범위 응답

- `multipart/byteranges`는 브라우저로 회신하는 부분적인 응답 전송의 컨텍스트 내에서 사용된다.
- 각각의 다른 파트들은 문서의 실제 타입을 가진 [`Content-Type`](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Content-Type) 헤더와 그들이 나타내는 범위를 가진 [`Content-Range`](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Content-Range)를 가지고 있다.

```
HTTP/1.1 206 Partial Content
Accept-Ranges: bytes
Content-Type: multipart/byteranges; boundary=3d6b6a416f9b5
Content-Length: 385

--3d6b6a416f9b5
Content-Type: text/html
Content-Range: bytes 100-200/1270

eta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="vieport" content
--3d6b6a416f9b5
Content-Type: text/html
Content-Range: bytes 300-400/1270

-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: "Open Sans", "Helvetica
--3d6b6a416f9b5--
```



## 15.5 콘텐츠 인코딩

- HTTP 애플리이션은 압축, 암호화 등의 인코딩 작업을 콘텐츠를 보내기 전에 하려고 한다.

### 15.5.1 콘텐츠 인코딩 과정

1. 웹 서버가 원본 `Content-Type`과 `Content-Length` 헤더를 수반한 원본 응답 메시지를 생성한다.
2. `콘텐츠 인코딩 서버` (원 서버 or 다운스트림 프락시)가 `인코딩된 메시지`를 생성한다. 
   - 인코딩된 메시지에 따라 Content-Type은 변하지 않지만 Content-Length는 변한다. 
   - 콘텐츠 인코딩 서버는 `Content-Encoding` 헤더를 인코딩된 메시지에 추가하여, 수신 측 애플리케이션이 메시지를 디코딩할 수 있도록 한다.
3. 수신 측 프로그램은 인코딩된 메시지를 받아서 디코딩하고 원본을 얻는다.

### 15.5.2 콘텐츠 인코딩 유형

- gzip, compress, deflate 인코딩은 전송되는 메세지의 크기를 정보의 손실 없이 줄이기 위한 **무손실 압축 알고리즘**이다.

### 15.5.3 Accept-Encoding 헤더

- 클라이언트는 자신이 지원하는 인코딩의 목록을 Accept-Encoding 헤더를 통해 전달
- 이 헤더가 없다면 어떤 인코딩이든 받아들인다는 의미.
- q(quality) 값을 매개변수로 선호도 나타낼 수 있다. (0-싫음 < 1- 좋음)

```
Accept-Encoding: compress, gzip
Accept-Encoding:
Accept-Encoding: *
Accept-Encoding: compress; q=0.5, gzip; q=1.0
Accept-Encoding: gzip; q=1.0, identiy; q=0.5, *; q=0
```

