# 스프링 MVC 동작과정

## Dispatcher-Servlet이란

```
서블릿 컨테이너에서 HTTP 프로토콜을 통해 들어오는 모든 요청을 프레젠테이션 계층의 가장 앞에 둬서 중앙집중식으로 처리해주는 프론트 컨트롤러
```

설명하면, 클라이언트로부터의 어떤 요청을 톰캣과 같은 서블릿 컨테이너가 받는데, 이때 **제일 앞에서 서버로 들어오는 모든 요청을 처리하는 프론트 컨트롤러**라고 스프링에서 정의하였고, 이를 Dispatcher-Servlet이라 한다.

그래서 공통 처리 작업을 Dispatcher-Servlet이 처리한 후, 적절한 세부 컨트롤러(사용자가 생성한)로 작업을 위임해준다.

## Dispatcher-Servlet 흐름

- Spring MVC는 해당 서블릿이 등장함에 따라 web.xml의 역할을 상당히 축소시켜줬다.
- 기존에는 모든 서블릿에 대해 URL 매핑을 활용하기 위해 web.xml에 모두 등록해줘야 했지만, **Dispatcher-Servlet이 해당 어플리케이션으로 들어오는 모든 요청을 핸들링**해주면서 작업을 편리하게 처리할 수 있게 되었다.

![image](https://user-images.githubusercontent.com/47625368/123147824-7650f000-d49a-11eb-87cb-ea71bf8e4ea9.png)

1. 클라이언트의 모든 요청은 DispatcherServlet이 받는다.
2. DispatcherServlet은 handlerMapping을 통해 요청에 해당하는 Controller를 실행한다.
3. Controller가 처리한 비즈니스 결과 및 View의 이름을 포함한 데이터를 DispatcherServlet에 리턴한다.
4. Controller에서 보내온 데이터를 ViewResolver을 통해 전달받을 View가 있는지 검색한다.
5. 전달 받은 View가 있다면 View에게 전달된 결과(처리 데이터)를 전송한다.
6. View는 전달 받은 결과를 다시 DispatcherServlet에게 보낸다.
7. DispatcherServlet은 이 결과를 클라이언트에게 렌더링하여 전달한다.

물론 이렇게 DispatcherServlet이 요청을 Controller로 넘겨주는 방식이 효율적으로 보이지만, **모든 요청을 처리하다보니 이미지 또는 HTML 파일 등을 불러오는 요청마저 전부 Controller로 넘겨버리게 된다.** 추가로 resource 파일에 대한 요청까지 모두 DispatcherServlet이 가로채는 까닭에 자원을 불러오지도 못하는 상황도 발생한다.

이를 해결하기 위해, <mvc:resources />를 이용한 방법이 있다. 이는 만약 **DispatcherServlet에서 해당 요청에 대한 컨트롤러를 찾을 수 없을 때, 2차적으로 설정된 경로에서 요청을 탐색하여 자원을 찾아낼 수 있다.** 이렇게 영역을 분리하면 효율적인 리소스 관리를 지원할 뿐만 아니라 추후에 확장을 용이하게 해준다는 장점이 있다.

### 참고자료

- https://mangkyu.tistory.com/18?category=761302
