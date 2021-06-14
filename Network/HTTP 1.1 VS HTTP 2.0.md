# HTTP/1.1 VS HTTP/2.0

## HTTP/1.0

- 단기 커넥션(Short-lived connections)
- 요청을 보낼 때마다 매번 새로운 커넥션을 생성하고, 응답이 도착한 이후에 연결을 바로 닫는 형태
- TCP 연결을 매번 여는 것은 자원 소비하기 때문에 성능상의 제약이 발생한다.
- **Keep-alive 속성**을 통해 일정 시간 동안 연결을 끊지 않고, 하나의 연결을 여러 요청에 재사용할 수 있도록 했다.

## HTTP/1.1 - 표준 프로토콜

- 1.0과 마찬가지로, 기본은 커넥션당 하나의 요청을 처리하도록 설계되었다.
- 여기에 두 가지 모델이 추가
  - 영속적인 커넥션 모델(Persistent connection)
    - Keep-alive를 따로 설정하지 않아도 디폴트로 지속 커넥션이 활성화되어있다.
    - TCP 연결 비용을 아끼고 성능 향상에 기여할 수 있다.
  - HTTP 파이프라이닝(HTTP Pipelining)
    - 같은 영속 커넥션을 통해, 응답을 기다리지 않고 요청을 연속적으로 보내는 기능
    - 요청 레이텐시를 줄일 수 있다.

![image](https://user-images.githubusercontent.com/47625368/121848424-f5b92380-cd24-11eb-8667-522351b59bfe.png)

## HTTP/1.1의 단점

- 기본적으로 연결당 하나의 요청과 응답을 처리하기 때문에 동시 전송 문제와 다수의 리소스를 처리하기에 속도와 성능 이유가 존재한다.
- Head Of Line(HOL) Blocking(특정 응답 지연)
  - 파이프라이닝 연결에서 발생
  - 요청의 순서와 응답 순서는 동기화 되어야 하는데, 앞 순서의 요청의 지연이 발생하면 그만큼 다른 요청을 처리하는데 지연이 발생한다.
- 무거운 헤더 구조(특히 Cookie)
  - 많은 메타정보들을 저장한다.
  - 매 요청마다 중복된 헤더를 보내기 때문에 비효율적이다.

## HTTP/1.1 단점 극복 방법

### 이미지 스프라이팅(Image Spriting)

- 다수의 리소스 요청을 보내게 되면 HOL Blocking이 발생할 수도 있기 때문에 이미지들을 하나로 합쳐서 리소스 요청을 한 번만 보내는 방식

### 도메인 샤딩(Domain Sharding)

- 다수의 커넥션을 생성해서 병렬로 요청을 보낸다.
- 하지만, 브라우저 별로 도메인당 커넥션 개수의 제한이 존재하기 때문에 근본적인 해결책은 아니다.
  ![image](https://user-images.githubusercontent.com/47625368/121849989-11bdc480-cd27-11eb-900a-7f5e8bd72549.png)

### Minify CSS/Javascript

- CSS와 Javascript 파일을 압축시킨다.

### SPDY 등장

- HTTP/1.1의 단점을 근본적으로 해결하지 못했다.
- 구글은 더 빠른 웹을 실현하기 위해 throughput 관점이 아닌 Latency 관점에서 HTTP를 고속화한 SPDY(스피디)라 불리는 새로운 프로토콜을 구현했다.
- HTTP를 대치하는 프로토콜이 아니고 HTTP를 통한 전송을 재정의하는 형태로 구현 - HTTP/2.0 초안의 참고 규격이 되었다.

## HTTP/2.0 주요 특징

### Multiplexed Streams

- 한 커넥션으로 동시에 여러 개의 메시지를 주고 받을 수 있으며, **응답은 순서에 상관없이 stream으로 주고 받는다.**
- HTTP/1.1의 Connection Keep-Alive, Pipelining을 개선했다.
- 아래의 이미지처럼 **하나의 커넥션에서 여러 병렬 스트림이 존재**할 수 있다. 스트림이 뒤섞여서 전송될 경우, stream number을 이용해 수신측에서 재조합된다.
  ![image](https://user-images.githubusercontent.com/47625368/121850494-d2dc3e80-cd27-11eb-9b5e-9a600b32eb02.png)

### Stream Prioritization

- 만약 클라이언트가 요청한 HTML 문서 안에 CSS 파일 1개와 Image 파일 2개가 존재하고, 이를 요청했을 때, Image 파일보다 CSS 파일의 수신이 늦어지는 경우 브라우저의 렌더링 문제가 발생
- 이때 **리소스간 의존관계(우선순위)를 설정**하여 문제를 해결한다.

### Server Push

- **서버는 클라이언트의 요청에 대해 요청하지도 않은 리소스를 보내줄 수 있다.**
- 클라이언트가 HTML 문서를 요청하고 여러 개의 리소스가 포함되어 있을 때, HTTP/1.1의 경우 클라이언트는 요청한 HTML 문서를 수신한 후 이를 해석하면서 필요한 리소스를 재요청하였다.
- 반면, HTTP/2.0의 경우 Server Push 기법을 통해 **클라이언트가 요청하지 않은 리소스를 서버 측에서 미리 알아서 Push하여, 클라이언트의 요청을 최소화하고 성능 향상을 이끌어낸다.**
- PUSH_PROMISE라고 부른다.
  ![image](https://user-images.githubusercontent.com/47625368/121851075-a83eb580-cd28-11eb-97ac-f4e2505c15ee.png)

### Header Compression

- 무거운 헤더 정보를 압축하기 위해, Header Table과 Huffman Encoding 기법을 사용하여 처리한다(HPACK 압축방식)
  ![image](https://user-images.githubusercontent.com/47625368/121851319-ecca5100-cd28-11eb-8180-38f3b60293ab.png)
- 위 그림처럼 클라이언트가 두 번의 요청을 보낸다고 할 때, HTTP/1.x의 경우 헤더의 중복 값이 존재해도 그냥 중복 전송한다.
- 반면, HTTP/2.0에서는 헤더에 중복값이 존재할 때 **중복 헤더를 검출한 후 중복된 헤더의 인덱스만 전송**하고, 중복되지 않은 헤더 정보는 Huffman Encoding 기법으로 인코딩 처리하여 전송한다.
  ![image](https://user-images.githubusercontent.com/47625368/121851550-3dda4500-cd29-11eb-9c20-7964be93b940.png)

### 참고자료

- https://developer.mozilla.org/ko/docs/Web/HTTP/Connection_management_in_HTTP_1.x
- https://ijbgo.tistory.com/26
- https://velog.io/@hoo00nn/HTTP1.1-vs-HTTP2.0
- https://goodgid.github.io/HTTP-1.1/
