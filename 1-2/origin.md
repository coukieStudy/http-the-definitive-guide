# 2장 URL과 리소스

## URI(Uniform Resource Identifier)

- URL(Uniform Resource Location)
    - '스킴://서버위치/경로' 구조
    - ex) [http://www.joes-hardware.com/seasonal/index-fall.html](http://www.joes-hardware.com/seasonal/index-fall.html)
        - 스킴(어떻게): 웹 클라이언트가 리소스에 어떻게 접근하는지 알려준다. 위의 경우는 http:// 부분으로 HTTP 프로토콜을 사용 (http://)
        - 서버 위치(어디에): 웹 클라이언트가 리소스가 어디에 호스팅되어 있는지 알려준다. (www.joes-hardware.com)
        - 경로(무엇을): 리소스의 경로. 요청받은 리소스가 무엇인지 알려준다.  (seansonal/index-fall.html)
- URN(Uniform Resource Name)

## URI, URN, URL 관계 - 그림

![2%E1%84%8C%E1%85%A1%E1%86%BC%20URL%E1%84%80%E1%85%AA%20%E1%84%85%E1%85%B5%E1%84%89%E1%85%A9%E1%84%89%E1%85%B3%20859e64fe3bc74277a9fe10f917f518a0/Untitled.png](2%E1%84%8C%E1%85%A1%E1%86%BC%20URL%E1%84%80%E1%85%AA%20%E1%84%85%E1%85%B5%E1%84%89%E1%85%A9%E1%84%89%E1%85%B3%20859e64fe3bc74277a9fe10f917f518a0/Untitled.png)

## URL 문법

```
<스킴>://<사용자 이름>:<비밀번호>@<호스트>:<포트>/<경로>;<파라미터>?<질의>#<프래그먼트>
```

### 스킴: 사용할 프로토콜

위에서 이야기한 http://와 같이 쓰면 HTTP 프로토콜, ftp로 쓰면 ftp

### 호스트와 포트

- 호스트: 위에서처럼 www.joes-hardware.com
- 포트: 서버가 열어놓은 포트를 가리킨다

(내부적으로 TCP 프로토콜을 사용하는 HTTP는 기본 포트로 80)

### 사용자 이름과 비밀번호

많은 서버들이 자신이 가지고 있는 데이터에 접근을 허용하기 전에 사용자 이름과 비밀번호를 요구한다.

### 경로

URL의 경로 컴포넌트는 리소스가 서버의 어디에 있는지 알려준다. 

HTTP URL에서 경로 컴포넌트는 '/' 문자를 기준으로 경로조각으로 나뉜다.

### 파라미터(';'로 구분해서 기술)

URL의 paramter component는 Application이 서버에 정확한 요청을 하기 위해 필요한 입력 파라미터를 받는데 사용한다. 프로토콜 파라미터가 없으면 서버는 그 요청을 잘못 처리하거나 처리를 하지 않을 것

### 질의 문자열

데이터베이스 같은 서비스들은 요청받을 리소스 형식의 범위를 좁히기 위해 질문이나 질의를 받을 수 있다.

- ex) [http://www.joes-hardware.com/inventory-check.cgi?item=12731](http://www.joes-hardware.com/seasonal/index-fall.html)
    - item 번호 12731의 재고가 있는지 확인하기 위해서 다음과 같이 질의

### 프래그먼트

리소스의 특정 부분을 가리킬 수 있도록, URL은 리소스 내의 조각을 가리킬 수 있는 프래그먼트 컴포넌트를 제공한다. 

→ 직접 인터넷에서 설명

## 단축 URL

### 상대 URL

URL은 상대 URL과 절대 URL로 나뉘는데, 지금까지 우리가 본 것은 절대 URL

- 절대 URL: 리소스에 접근하는데 필요한 모든 정보를 가지고 있다.
- 상대 URL: 리소스에 접근하는데 필요한 모든 정보를 얻기 위해서는 기저 URL을 사용해야 한다.
- Example: [https://ry-kor-blog.tistory.com/6](https://ry-kor-blog.tistory.com/6)

### [상대 URL 변환 과정] 첫번째, 기저 URL 찾기

기저 URL을 가져오는 몇 가지 방법

1. 리소스에서 명시적으로 제공
    - 예를 들어, HTML 문서에서는 그 안에 있는 모든 상대 URL을 변경하기 위해서 기저 URL을 가리키는 <BASE> HTML 태그를 기술할 수 있다.
2. 리소스를 포함하고 있는 기저 URL
    - 해당 리소스의 URL을 기저 URL로 쓸 수 있다
3. 기저 URL이 없는 경우
    - 기저 URL이 없는 경우, 절대 URL만으로 이루어져 있다는 의미.
    - 하지만, 깨진 URL인 가능성도 높다.

### [상대 URL 변환 과정] 두번째, 상대 URL과 기저 URL을 각각의 컴포넌트 조각으로 나누는 것

사실상 URL 파싱!

**TBD → 파싱 알고리즘**

## URL 확장

어떤 브라우저들은 URL을 입력한 다음이나 입력하고 있는 동안에 자동으로 URL 확장

**호스트명 확장**

브라우저 주소창에 yahoo를 입력하면, www와 .com을 붙여서 [www.yahoo.com](http://www.yahoo.com) 을 만들어준다.

**히스토리 확장**

사용자의 URL 작성 시간을 줄이고자 기록을 저장해두고 자동완성

## 안전하지 않은 문자

URL은 인터넷에 있는 모든 리소스가 여러 프로토콜을 통해 잘 전달될 수 있도록 **안전하게 설계**하는 것이 중요했다. 또한, **가독성**도 있어야 했다.

→ SMTP 같은 프로토콜은 특정 문자를 제거할 수도 있는 전송 방식을 사용하기에 URL은 상대적으로 **작고 일반적으로 안전한 알파벳 문자만 포함(안전성)**하도록 한다.

### URL 문자 집합

보통 이스케이프 문자열 사용하는데, 안전하지 않은 문자는 퍼센티지 기호(%)로 시작해서 ASCII코드로 표현되는 두 개의 16진수 숫자로 이루어진 '이스케이프' 문자로 바꾼다.

- URL Encoding/Decoding: [https://meyerweb.com/eric/tools/dencoder/](https://meyerweb.com/eric/tools/dencoder/)
- Percent Encoding: [https://ko.wikipedia.org/wiki/퍼센트_인코딩](https://ko.wikipedia.org/wiki/%ED%8D%BC%EC%84%BC%ED%8A%B8_%EC%9D%B8%EC%BD%94%EB%94%A9)

ex) [http://www.joes-hardware.com/~joe](http://www.joes-hardware.com/~joe에) 와 같은 식으로 '~' 문자를 인코딩하지 않고 요청할 때, 어떤 전송 프로토콜에서는 문제가 되지 않을 수 있는데 이 부분은 실수!

### 인코딩을 하는 위치!

- 브라우저와 같은 사용자로부터 최초로 URL을 입력받는 애플리케이션에서 하는 것이 가장 적절하다.

### 일반적인 스킴

**HTTP**

- 포트 값이 생각되어있으면, 기본값을 80

```
http://<호스트>:<포트>/<경로>?<질의>#<프래그먼트>
```

**HTTPS**

- HTTP의 커넥션의 양 끝단에서 암호화하기 위해 SSL(Secure Socket Layer)을 사용한다는 것이 HTTP와 다른 점
    - 참고: [https://opentutorials.org/course/228/4894](https://opentutorials.org/course/228/4894)
- 포트 값이 생각되어있으면, 기본값을 443

```
http://<호스트>:<포트>/<경로>?<질의>#<프래그먼트>
```

## 미래

URL 

- 장점: 일관된 작명 규칙을 통해 모든 객체에 이름을 지을 수 있게 한다.
- 단점: 위치가 고정된다. 리소스가 옮겨지면 해당 URL은 더 이상 사용 불가

→ 이를 위해, 객체의 위치와 상관없이, 그 객체를 가리키는 실제 객체의 이름을 사용하는 것

URN(Uniform Resource Name)

- 지속 통합 자원 지시자(Persistent Uniform Resource Locators, PURL)을 사용하면 URL로 URN의 기능을 제공할 수 있다. PURL은 리소스의 실제 URL 목록을 관리하고 추적하는 리소스 위치 중개 서버를 두고, 해당 리소스를 우회적으로 제공

→ URN은 시간을 많이 두고 진행 중. 기존 URL이 있으니....

### 문서

- URL - RFC1738: [https://www.ietf.org/rfc/rfc1738](https://www.ietf.org/rfc/rfc1738)