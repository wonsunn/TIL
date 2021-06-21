# 동기 VS 비동기, Blocking VS Non-Blocking

## 동기(Synchronous) VS 비동기(Asynchronous)

```
처리해야 할 작업들을 어떠한 '흐름'으로 처리할 것인가
호출되는 함수의 작업 완료 여부를 누가 신경쓰는지
```

### 동기(Synchronous)

- 작업 요청을 했을 때 요청의 결과값(return)을 직접 받는다.
- **호출하는 함수가** 호출되는 함수의 작업을 완료한 후의 **return 값을 기다린다.**
- 호출되는 함수로부터 바로 return을 받더라도 **호출하는 함수가 작업 완료 여부를 확인하면서 신경 쓴다.**

### 비동기(Asynchronous)

- 작업 요청을 했을 때 결과값(return)을 간접적으로 받으며, 해당 요청 작업은 **별도의 스레드에서 실행**된다.
- 호출되는 함수에게 콜백을 전달하여 호출되는 함수의 작업이 완료되면, **호출되는 함수가 전달받은 콜백을 실행**한다.
- **호출한 함수는 작업 완료 여부를 신경쓰지 않는다.**
  - 콜백(Callback) 함수로 처리한다.

## Blocking VS Non-blocking

```
처리되어야 하는 (하나의) 작업이 전체적인 작업 '흐름'을 막느냐 안막느냐에 대한 관점
호출되는 함수가 바로 리턴하는지
주로 I/O 동작에서 적용된다.
```

### Blocking

- 다른 작업 주체가 하는 작업의 시작부터 끝까지 기다렸다가 다시 자신의 작업을 시작하는 형태.
- 호출된 함수가 **자신의 작업을 모두 마칠 때까지** 호출한 함수에게 **제어권을 넘겨주지 않고 대기**하게 만든다.

### Non-Blocking

- 다른 주체의 작업과 상관없이 자신의 작업을 계속 진행하는 형태.
- 호출된 함수가 **바로 리턴해서 호출한 함수에게 제어권을 넘겨줘**, 호출한 함수가 **다른 일을 할 수 있도록** 한다.

## 동기(Synchronous) + Blocking, 비동기(Asynchronous) + Non-Blocking

![image](https://user-images.githubusercontent.com/47625368/122721654-e2b9cc80-d2ab-11eb-9377-2ed5a42081e1.png)

### 동기(Synchronous) + Blocking

- 호출한 함수는 호출된 함수로부터 응답이 올 때까지 기다린다.(Synchronous)
- 호출된 함수는 I/O 작업을 모두 완료한 후에 응답을 보낸다.(Blocking)
- 예시
  - JDBC를 이용해 DB에 쿼리 질의를 날린다.

### 비동기(Asynchronous) + Non-Blocking

- 호출한 함수는 호출된 함수로부터 응답을 기다리지 않는다.(Asynchronous)
- 호출된 함수는 호출한 함수가 다른 일을 하도록 내버려 둔다.(Non-Blocking)
- 예시
  - 대규모 사용자에게 푸시메시지를 전송할 때
  - 다양한 외부 API를 한 번에 호출할 때

## 동기(Synchronous) + Non-Blocking, 비동기(Asynchronous) + Blocking

![image](https://user-images.githubusercontent.com/47625368/122725774-4c3bda00-d2b0-11eb-87ce-15e4c485f447.png)

### 동기(Synchronous) + Non-Blocking

- 호출된 함수가 호출한 함수에게 제어권을 넘겨주지만(Non-Blocking)
- 호출한 함수가 다른 작업과의 동기(Synchronous)를 맞추기 위해(작업 완료 여부를 계속 신경 씀) 작업을 마쳤는지 지속적으로 확인을 한다.
- 이 과정에서 context switching이 빈번하게 발생하여 비효율적인 동작이 이뤄진다.
- 예시
  - Polling

### 비동기(Asynchronous) + Blocking

- 호출한 함수는 작업 완료 여부에 관심이 없고 다른 일을 하고 싶어한다.(Asynchronous)
- 하지만, 호출된 함수가 제어권을 넘겨주지 않기 때문에(Blocking) 다른 일을 할 수 없는 상태.
- 결국 (동기 + Blocking)과 비슷한 작업 효율이 나오기 때문에 잘 사용하지 않는 조합이다.
- 예시
  - (비동기 + Non-Blocking) 작업을 실행하고 자신의 작업을 하던 도중 호출한 작업의 결과값을 조회하려고 할 때(블로킹 메소드 실행)

### 참고자료

- https://velog.io/@wonhee010/%EB%8F%99%EA%B8%B0vs%EB%B9%84%EB%8F%99%EA%B8%B0-feat.-blocking-vs-non-blocking
- https://darkstart.tistory.com/171
- https://deveric.tistory.com/99
- https://siyoon210.tistory.com/147
- https://ssungkang.tistory.com/entry/OS-%EB%8F%99%EA%B8%B0-vs-%EB%B9%84%EB%8F%99%EA%B8%B0-%EB%B8%94%EB%A1%9C%ED%82%B9-vs-%EB%85%BC%EB%B8%94%EB%A1%9C%ED%82%B9
