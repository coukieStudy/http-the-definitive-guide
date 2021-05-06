### 16.4 언어 태그와 HTTP
Charset = 코딩된 문자 집합 + 문자 인코딩 구조

UTF-8 = 유니코드(UCS, 코딩된 문자 집합) + 가변 인코딩 방식
- i18n vs l10n
  - internalization vs localization

언어 태그 : 언어에 짧게 표준화된 이름을 붙인 것(en, de, ko)
- Content-Language 헤더
  - 서버가 컨텐츠(문서, 오디오, 동영상 등)이 어떤 언어로 되어있는지 언어태그를 이용해서 표현
    - `Content-Language: fr`
    - `Content-Languate: mi, en` -> 여러 개도 가능
- Accept-Language 헤더
  - 클라이언트가 원하는 언어를 요청하는 것
  - `Accept-Language: es`
- 언어태그의 종류
  - RFC 3066에 포함
  - 특정 국가의 언어(영국 영어 en-GB), 방언, 지방어, 그외의 표준 언어(나비어), 비표준 언어 모두 가능
- 서브 태그
  - 언어 태그가 - 로 분리되어 서브태그를 가진다
  - `sgn-US-MA`
    - sgn : 주 서브태그, 표준화 되어있음
    - US : 두 번째 서브태그, 선택적, 자신만의 이름 표준
    - MA : IANA에 등록되어 있지 않음, 8자 이하의 알파벳과 숫자
- 대소문자 구분
  - 실제로 구분하지는 않으나, 관용적으로 언어는 소문자(fr) 나라는 대문자(FR) 사용한다
- IANA 언어 태그 등록
  - 표준이 있는 첫 번째, 두 번째 언어 서브태그는 모두 IANA에서 관리

### 16.5 국제화된 URI
- URI에서 사용하는 문자 : US-ASCII 문자들의 부분 집합
- ![문자집합](https://img1.daumcdn.net/thumb/R800x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F991D83345C17154A08)
- control character(CR, LF, ...)
  - https://en.wikipedia.org/wiki/Control_character#cite_note-1
- URI escaping
  - %<HEX><HEX> : 스페이스(ASCII 32 문자코드) -> %20
- 문자가 **한 번만 이스케이핑 되도록 주의**
- 국제 문자를 이스케이핑
  - 책에서는 US-ASCII(0-127, 0x00 - 0x7F) 문자코드만 이스케이핑 하도록 주장
  - 그러나 **요즘은 utf-8을 퍼센트 인코딩하여 사용한다**
  - https://ko.wikipedia.org/wiki/%ED%8D%BC%EC%84%BC%ED%8A%B8_%EC%9D%B8%EC%BD%94%EB%94%A9
    - 퍼 = U+D37C + utf-8 인코딩 -> ED 8D BC + 퍼센트 인코딩 -> %ED%8D%8C
    - 센 = U+C13C + utf-8 인코딩 -> EC 84 BC + 퍼센트 인코딩 -> %EC%84%BC
    - 트 = U+D2B8 + utf-8 인코딩 -> ED 8A B8 + 퍼센트 인코딩 -> %ED%8A%B8
    - 퍼센트_인코딩 -> %ED%8D%BC%EC%84%BC%ED%8A%B8_%EC%9D%B8%EC%BD%94%EB%94%A9

### 16.6 기타 고려사항
- 국제화된 HTTP 애플리케이션을 작성할 때
  - 헤더와 명세에 맞지 않는 데이터
    - HTTP 헤더는 반드시 ascii 문자 집합으로 이루어져야 하나 127 보다 큰 코드로 구현된 클라이언트와 서버가 있을 수 있다
    - 이를 처리할 때 에러가 발생하지 않게, 라이브러리의 문서를 잘 살피자
  - 날짜
  - 도메인 이름
    - 구제화 도메인 이름을 퓨니코드를 이용해 지원한다
    - 퓨니코드 : 유니코드를 호스트 명에 사용 가능한 문자만으로 이루어진 문자열로 변환
    - 한글.com -> http://xn--bj0bj06e.com/
