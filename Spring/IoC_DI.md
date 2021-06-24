## IoC(Inversion of Control)

```
객체 간의 의존 관계를 연결할 때의 제어권이 개발자가 아닌 스프링 컨테이너로 넘어가, 객체의 생성, 생명주기 관리 등을 스프링 컨테이너가 담당한다는 의미

이에 따라 개발자는 도메인의 핵심 비즈니스 로직에 더 집중할 수 있다.

DI를 통해 구현된다.
```

### 예시

A라는 클래스가 B라는 클래스의 메소드를 사용하기로 하자. 이런 관계를 A가 B에 의존한다고 한다. 왜냐하면 B의 로직에 따라 A가 영향을 받기 때문이다.

```java
public class A {

    private B b;

    public A() {
        b = new B_1();
    }

    public void useB() {
        b.method_1();
        b.method_2();
    }

}

interface B {
    void method_1();
    void method_2();
}

class B_1 {

    public void method_1() {
        System.out.println("B_1`s method_1");
    }

    public void method_2() {
        System.out.println("B_1`s method_2");
    }

}

class B_2 {

    public void method_1() {
        System.out.println("B_2`s method_1");
    }

    public void method_2() {
        System.out.println("B_2`s method_2");
    }

}
```

- 현재 A 클래스는 B 인터페이스의 어떤 구현 클래스를 사용할지를 결정할 수 있다.
- A 클래스는 자신의 로직에 대한 책임 뿐만 아니라 B 클래스의 구현에 대한 책임도 갖고 있는 셈.
- 이때 IoC를 통해, 이런 책임을 굳이 가질 필요 없이 의존관계에 대한 책임을 제 3자에게 위임할 수 있다.

Factory라는 제 3자 클래스를 만들고, 해당 클래스에서 A는 B 클래스의 정보를 읽어와 주입받는 방식으로 구현하면,

```java
public class Factory {
    public A a() {
        return new A();
    }

    public B b() {
        return new B_1();
        // return new B_2();
    }
}

public class A {

    private B b;

    public A() {
        b = new Factory().b();
    }
    ..이하 동일..
}
```

- Factory 클래스에서 B에 대한 리턴을 받아 A의 멤버변수 b에 주입하고 있다.
- 코드를 변경하고 싶다면, Factory 클래스를 변경하면 되기 때문에 더이상 A 클래스는 건드릴 필요가 없다.
- 이렇게 책임에 대한 분리가 이뤄진다.

### 장점

- 객체지향적으로 SRP(Single Responsibility Principle)을 지킬 수 있게 되었다.
- 즉, A 클래스는 B의 의존관계를 선택하는 책임에서 벗어난다.
- **B 클래스를 변경할 때 A 클래스의 변경이 필요없게 된 것.**

## DI(Dependency Injection)

```
외부에서 두 객체 간의 관계를 결정해주는 디자인 패턴으로,
인터페이스를 사이에 둬서 클래스 레벨에서는 의존관계가 고정되지 않도록 하고 런타임 시에 관계를 다이나믹에게 주입한다.

그래서 유연성을 확보하고 결합도를 낮춰줄 수 있다.
```

### DI를 위해 지켜야할 조건 3가지

1. 클래스 모델이나 코드에 **런타임 시점의 의존관계가 드러나지 않아야 한다.** 이를 위해서는 **인터페이스에만 의존**해야 한다.
2. 런타임 시점의 의존관계는 팩토리나 컨테이너 같은 **제 3의 존재(Factory)에 의해 결정**되어야 한다.
3. 의존관계는 사용할 객체에 대한 레퍼런스를 **외부에서 제공**해줌으로써 만들어진다.

### DI에 위배된 설계

```java
public class Pencil {

}
```

```java
public class Store {
  private Pencil pencil;

  public Store() {
    this.pencil = new Pencil();
  }
}
```

- **두 클래스가 강하게 결합되어 있다.**
  - 만약 Store에서 Pencil이 아닌 다른 상품을 판다면, Store 클래스의 생성자를 변경해야 한다.
- **객체들 간의 관계가 아니라 클래스 간의 관계가 맺어지고 있다.**
  - 객체들 간의 관계가 맺어졌다면, 다른 객체의 구체 클래스(Pencil인지 Food인지)를 전혀 알지 못하더라도 (해당 클래스가 인터페이스를 구현했다면) 인터페이스의 타입(Product)으로 사용할 수 있다.

### 의존성 주입(DI)을 통한 해결

- 위와 같은 문제를 해결하기 위해 **다형성이 필요**하다.
- Pencil, Food 등 여러가지 제품을 하나로 표현하기 위해서는 Product라는 인터페이스가 필요하다.

  ```java
  public interface Product {

  }
  ```

  ```java
  public class Pencil implements Product {

  }
  ```

  - Store와 Pencil이 강하게 결합되어 있는 부분을 제거하고, 이를 위해 **외부에서 상품을 주입**받아야 한다.

  ```java
  public class Store {
    private Product product;

    public Store(Product product) {
      this.product = product;
    }
  }
  ```

여기서 스프링이 DI(및 IoC) 컨테이너를 필요로 하는 이유를 알 수 있다. 우선 Store에서 Product 객체를 주입하기 위해서 **런타임 시점에 필요한 객체(Bean)를 생성**해야 하며, **의존성이 있는 두 객체를 연결하기 위해 한 객체를 다른 객체로 주입**시켜야 하기 때문이다.

예를 들어 다음과 같이 Pencil 객체를 만들고, 그 객체를 Store로 주입시켜주는 역할을 위해 스프링 컨테이너가 필요하게 된 것이다.

```java
public class BeanFactory {

  public void store() {
    // Bean 생성
    Product pencil = new Pencil();

    // 의존성 주입
    Store store = new Store(pencil);
  }

}
```

이는 동시에 제어의 역전(IoC)라고도 불리는 이유이다. 어떠한 객체를 사용할지에 대한 책임이 BeanFactory와 같은 클래스에게 넘어갔고, 자신은 수동적으로 주입받는 객체가 되기 때문이다.(실제 스프링에서는 BeanFactory를 확장한 Application Context를 사용한다.)

### 정리

- 두 객체 간의 관계라는 관심사 분리
- 두 객체 간의 결합도를 낮추고 유연성을 높임
- 테스트 작성이 용이해짐

하지만, 의존관계를 주입할 객체를 계속해서 생성하고 소멸한다면 아무리 GC가 성능이 좋더라도 부하가 가기 마련이다. 그래서 **스프링에서는 Bean들을 기본적으로 싱글톤으로 관리**한다.

### 참고자료

- https://sabarada.tistory.com/67?category=803157
- https://mangkyu.tistory.com/150?category=761302
