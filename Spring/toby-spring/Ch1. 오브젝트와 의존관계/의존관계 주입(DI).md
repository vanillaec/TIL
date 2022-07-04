## 제어의 역전과 의존관계 주입

- DaoFactory처럼 객체를 생성하고 관계를 맺어주는 등의 작업을 담당하는 기능을 일반화한 것이 스프링의 IoC 컨테이너이다. 스프링이 제공하는 IoC방식의 핵심은 의존 관계 주입(DI)이다.
    
    의존성 주입이란 오브젝트 레퍼런스를 외부로부터 제공(주입)받고 이를 통해 여타 오브젝트와 다이나믹하게 의존관계가 만들어지는 것을 의미한다.
    

## 의존관계

두 개의 클래스 또는 모듈이 의존관게에 있다는 할 때는 항상 방향성이 존재한다. 다음에서는 A가 B에 의존하고 있음을 의미한다.

![의존관계 도식도](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a31bae9a-d3db-4a94-b97f-e7ea7de5f905/123.png)

의존관계 도식도

A는 B에게 의존하고 있기 때문에 B에서 특정 메소드가 추가 혹은 수정되면 A에게도 영향을 미친다. 

### UserDao의 의존관계

UserDao의 의존관계를 살펴보면 ConnectionMaker라는 인터페이스에 의존하고 있음을 알 수 있다. 하지만 UserDao의 입장에서 DConnectionMaker라는 구현체의 존재를 알 수 없기 때문에 DConnectionMaker의 메소드가 추가, 수정되어도 UserDao에게 영향을 주지 않는다. 이처럼 인터페이스에 대해서만 의존관계를 만들어두면 인터페이스 구현 클래스와의 관계는 느슨해지면서 변화에 영향을 덜 받는 상태가 된다.

![인터페이스를 통한 느슨한 결합을 갖는 의존관계](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1478059a-3fbe-4c99-8800-2098bca5ddff/5555.png)

인터페이스를 통한 느슨한 결합을 갖는 의존관계

이처럼 UML 관계에서 나오는 의존관계를 설계 모델의 관점에서 이야기하는 것이다. 하지만 의존관계에는 런타임 시에 오브젝트 사이에서 만들어지는 의존관계도 존재한다.

### 런타임 시 발생하는 의존관계

- 인터페이스를 통해 설계 시점에 느슨한 의존관계를 갖는 경우에는 UserDao의 오브젝트가 런타임 시에 사용할 오브젝트가 어떤 클래스로 만든 것인지 미리 알 수가 없다. 의존관계 주입은 이렇게 구체적인 의존 오브젝트와 그것을 사용할 주체, 보통 클라이언트라고 부르는 오브젝트를 런타임 시에 연결해주는 작업을 말한다.
    
    프로그램이 시작되고 UserDao 오브젝트가 만들어지고 나서 런타임 시에 의존관계를 맺는 대상, 즉 실제 사용대상인 오브젝트를 의존 오브젝트라고 말한다.
    

🤔 UserDao 오브젝트를 생성한다는게 무슨 말임? 그니까 그냥 빈 생성자를 통해 UserDao를 생성해두고 나중에 private final ConnectionMaker con; 얘를 필요할 때 동적으로 매꿔준다 이말인가.

### UserDao를 통해 알아보는 의존관계 주입

UserDao를 개선하기 전에는 다음과 같이 DConnectionMaker()라는 구현 클래스를 알고 있었다.

```java
public UserDao() {
	connectionMaker = new DConnectionMaker();
}
```

이를 UserDao() 생성자를 통해 ConnectionMaker의 구현체를 주입받도록 함으로써, 즉 제 3자(DaoFactory)에게 ConnectionMaker의 구현체를 결정하고, 레퍼런스를 넘겨주도록 함으로써 이를 해결했다.

```java
public class UserDao {

    private ConnectionMaker connectionMaker;

    public UserDao(ConnectionMaker connectionMaker) {
        this.connectionMaker = connectionMaker;
    }
}
```

DaoFacotry는 여기서 두 오브젝트 사이의 런타임 의존관계를 설정해주는 의존관계 주입 작업을 주도하는 존재이며, 동시에 IoC방식으로 오브젝트의 생성과 초기화, 제공 등의 작업을 수행하는 컨테이너이다. 이렇게 의존관계 주입을 담당하는 컨테이너를 DI 컨테이너라고 한다. DI 컨테이너는 UserDao를 만드는 시점에서 생성자의 파라미터로 이미 만들어진 DConnectionMaker의 오브젝트를 전달한다. 

## 의존관계 검색과 주입

### 의존관계 검색

- 스프링이 제공하는 IoC방법에는 의존관계 주입뿐만이 아니라 의존관계 검색이라는 기능도 제공한다. 이를 예시를 통해 알아보자.
    
    의존관계를 맺는 방법이 외부로부터의 주입이 아니라 스스로 검색을 이용한다.
    

다음과 같이 UserDao의 생성자 리스트를 만들었다고 가정하자.

```java
public UserDao() {
	DaoFactory daoFactory = new DaoFactory();
	this.connectionMaker - daoFactory.connectionMaker();
}
```

이렇게 해도 UserDao는 어떤 ConnectionMaker를 사용하는지 알지 못한다. 따라서 IoC 개념을 잘 따르고 있으며, 그 혜택을 받는다고 할 수 있다. 하지만 그 적용 방법이 스프링이 제공하는 방식과 다르다. 즉, 외부로부터의 주입이 아니라 스스로 IoC 컨테이너인 DaoFactory에게 요청하는 것이다. 스프링에서는 미리 정해놓은 이름을 전달해서 그 이름에 해당하는 오브젝트를 찾아온다. 

다음은 의존관계 검색 방식으로 ConnectionMaker 오브젝트를 가져오는 코드이다. 위에서와 달리 connectionMaker는 getBean() 메소드를 통한 검색 방법을 통해 ConnectionMaker 오브젝트를 가져온다.

```java
public UserDao() {
	AnnotationConfigApplicationContext context =
		new AnnotationConfigApplicationContext(DaoFactory.class);
	this.connectionMaker = context.getBean("connectionMaker", ConnectionMaker.class);
}
```

의존관계 검색과 주입 방식중에 어떤 것이 더 나을까? 코드를 보면 의존관계 검색에서는 오브젝트 팩토리 클래스나, 스프링 API가 나타난다. 따라서 대개는 의존관계 주입 방식을 사용하는 편이 낫다. 그런데 의존관게 검색 방식을 사용해야만 할 때가 있다. (스태틱 메소드인 main(), 서버에서 main()과 같은 기동 메소드는 없지만, 사용자의 요청을 받을 때 마다 main() 메소드와 비슷한 역할을 하는 서블릿)

### 의존관계 검색과 주입의 차이점

의존관계 검색 방식에서는 검색하는 오브젝트는 자신이 스프링의 빈일 필요가 없다. 반면에 의존관계 주입에서는 UserDao와 ConnectionMaker 사이에 DI가 적용되려면 UserDao도 반드시 컨테이너가 만드는 빈 오브젝트여야 한다. (UserDao에 ConnectionMaker 오브젝트를 주입해주려면 UserDao에 대한 생성과 초기화 권한을 갖고 있어야 하고, 그러려면 UserDao는 IoC 방식으로 컨테이너에서 생성되는 오브젝트, 즉 빈이여야 한다.)