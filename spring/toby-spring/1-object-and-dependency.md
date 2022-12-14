# 제 1장 오브젝트와 의존관계
## 1.4 제어의 역전

간단하게 말해서 프로그램의 제어 흐름 구조가 뒤바뀌는 것

일반적으로 프로그램의 흐름은 main() 메서드 안에서 사용할 오브젝트를 결정, 생성, 메서드 호출 등의 작업을 반복한다.

이런 구조에서 각 오브젝트는 프로그램 흐름과 오브젝트 구성 작업에 능동적으로 참여한다.

**제어의 역전이란 이런 제어 흐름의 개념을 뒤집는 것이다.**

오브젝트가 자신이 사용할 오브젝트를 스스로 선택하지도 생성하지도 않는다.

모든 제어 권한을 다른 대상에게 위임한다.

프레임워크도 제어의 역전 개념이 적용된 대표적인 기술이다. 라이브러리와 다른 점 중 하나인데 라이브러리는 사용하는 애플리케이션 흐름을 직접 제어한다. 반면 프레임워크는 애플리케이션 코드가 프레임워크에 의해 사용된다. 애플리케이션 코드는 프레임워크가 짜놓은 틀에서 수동적으로 동작해야 한다.

IoC를 적용함으로써 설계가 깔끔해지고 유연성이 증가하며 확장성이 좋아진다.

제어의 역전에서는 컨테이너와 같이 애플리케이션 컴포넌트의 생성과 관계 설정, 사용, 생명 주기 관리 등을 관장하는 존재가 필요하다.

## 1.5 스프링의 IoC

스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트를 ***빈(bean)***이라고 부른다. 빈의 생성과 관계 설정 같은 제어를 담당하는 IoC 컨테이너를 ***빈 팩토리(bean factory***)라고 부른다. 보통 빈 팩토리엑서 더 확장한 ***애플리케이션 컨텍스트(application context)***를 주로 사용한다. 두 가지는 동일하다고 봐도 되는데 빈 팩토리를 말할 때는 빈을 생성하고 관계를 설정하는 IoC 기본 기능에 초점을 맞춘 것이고, 애플리케이션 컨텍스트라고 할 때는 애플리케이션 전반에 걸친 모든 구성 요소의 제어를 담당하는 IoC 엔진이라는 의미가 더 부각된다고 보면 된다.

### 애플리케이션 컨텍스트 동작 방식

애플리케이션 컨텍스트는 `AllicationContext` 인터페이스를 구현하는데 `ApplicationContext`는 빈 팩토리가 구현하는 `BeanFactory`를 상속했으므로 일종의 빈 팩토리인 셈이다. 애플리케이션 컨텍스트는 애플리케이션 전반의 오브젝트 관리를 하는데 관리할 오브젝트 정보를 설정하는 코드를 가지고 있지 않다. 별도의 설정 정보에 그 역할을 위임하고 결과를 가져다가 사용한다.

개발자가 직접 IoC 컨테이너를 구현해서 아래처럼 사용할 수도 있을텐데

```java
public class UserDaoTest {

		private final DaoFactory daoFactory = new DaoFactory();
		
		public static void main(String[] args) throws ClassNotFoundException, SQLException {
				UserDao dao = daoFactory.getUserDao();
		}
}
```

스프링의 애플리케이션 컨텍스트를 사용했을 때는 어떤 장점이 있을까?

- **클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다.**
    - 애플리케이션이 발전하면 DaoFactory처럼 IoC를 적용한 XXXFactory 클래스가 추가될 것이다. 필요한 오브젝트를 가져오려면 어떤 Factory를 써야 하는지 클라이언트 코드에서 알아야하고 필요할 때마다 팩토리 클래스를 생성해야 하는 번거로움이 있다.
    - 애플리케이션 컨텍스트를 사용하면 팩토리 클래스가 아무리 많아져도 (`@Configuration`을 붙인 설정 클래스) 클라이언트 코드에선 알 필요가 없고 일관된 방식으로 (`@AutoWired`) 빈을 가져올 수 있다.
- **애플리케이션 컨텍스트는 종합 IoC 서비스를 제공해준다.**
    - 애플리케이션 컨텍스트는 단순히 오브젝트 생성 및 관계 설정만을 해주는 것이 아니다. 오브젝트가 만들어지는 방식, 시점, 전략을 다르게 할 수도 있고 부가적으로 자동 생성, 후처리, 조합 방식의 다변화, 인터셉팅 등의 다양한 기능을 제공한다.
    - 빈이 사용할 수 있는 기반 기술 서비스나 외부 시스템과의 연동 등을 컨테이너 차원에서 제공해 주기도 한다.
- **애플리케이션 컨텍스트는 빈을 검색하는 다양한 방법을 제공한다.**
    - `getBean()` 메서드를 쓸 수도 있고 타입만으로 검색하거나 특별한 애노테이션이 설정 되어 있는 빈을 찾을 수도 있다.

## 1.6 싱글톤 레지스트리와 오브젝트 스코프

애플리케이션 컨텍스트는 싱글톤을 저장하고 관리하는 ***싱글톤 레지스트리(singleton registry)***이기도 하다. 스프링은 기본적오르 빈 오브젝트를 모두 싱글톤으로 만든다. 디자인 패턴에서 나오는 싱글톤 패턴과 구현 방븝은 다르다.

### 서버 애플리케이션과 싱글톤

왜 스프링은 싱글톤으로 빈을 만드는 것일까? 스프링은 엔터프라이즈 시스템을 위해 고안된 기술이다. 서버 하나가 여러 요청을 동시에 받아 처리할 수 있어야 하는데 매번 클라이언트 요청이 올 때마다 로직을 담당하는 오브젝트를 새로 만든다면 어떻게 될까. 요청 한 번에 5개 오브젝트가 새로 만들어지고 초당 500개 요청이 들어온다면 한 시간에 9백만 개의 오브젝트가 만들어 진다. 즉 메모리 부담이 증가한다.

때문에 서블릿 컨테이너도 서블릿을 싱글톤으로 관리하고 여러 스레도에서 하나의 오브젝트를 공유해서 사용했다. 스프링도 빈을 싱글톤으로 관리하는데 디자인 패턴에서의 싱글톤 패턴은 사용하기 까다롭고 문제점이 있다.

### 싱글톤 패턴의 한계

- **private 생성자를 갖고 있기 때문에 상속할 수 없다.**
    - 다형성을 적용할 수 없다.
    - 객체지향적인 설계 장점을 적용할 수 없다.
- **테스트하기 힘들다.**
    - 싱글톤은 만들어지는 방식이 제한적이라 mock으로 만들기 어렵다.
    - 초기화 과정에서 생성자 등을 통해 오브젝트를 동적으로 주입하기도 힘들다.
- **서버 환경에서는 싱글톤이 하나만 만들어지는 것을 보장하지 못한다.**
    - 클래스 로더를 어떻게 구성하고 있느냐에 따라 하나 이상의 오브젝트가 만들어질 수도 있다.
    - 여러 개의 JVM이 분산돼서 설치 되는 경우에도 독립적으로 오브젝트가 생긴다.
- **싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 바람직하지 못하다.**
    - 싱글톤의 스태틱 메서드를 이용하면 언제 어디서든 싱글톤 메서드에 접근할 수 있게 된다.
    - 자연스럽게 전역 상태로 사용되기 쉽고 아무 객체에서 자유롭게 접근하고 수정, 공유할 수 있게 되는 것은 객체지향에서 권장되지 않는다.
    - 그럴 바에는 스태틱 필드와 메서드로만 구성된 클래스를 사용하는 것이 낫다.

### 싱글톤 레지스트리

위와 같은 한계점을 해결하고자 스프링이 직접 싱글톤을 지원하는데 그것이 바로 ***싱글톤 레지스트리***다. 싱글톤 패턴을 적용하지 않은 **평범한 클래스를 싱글톤으로 활용할 수 있게 해주기 때문에** 스프링이 지지하는 객체지향 설계 방식과 원칙, 디자인 패턴 등을 적용하는 데 제약이 없다.

### 싱글톤 오브젝트의 상태

**싱글톤은 멀티 스레드 환경에서 여러 스레드가 동시에 접근해서 사용할 수 있기 때문에 상태 관리에 주의를 기울여야 한다**. 때문에 스프링 빈은 무상태로 설계해야 한다.

무상태인 경우 각 요청이나 DB나 서버 리소스 정보는 어떻게 다뤄야 할까? 파라미터와 로컬 변수, 리턴 값 등을 이용하면 된다. 스프링이 한 번 초기화한 후에 수정되지 않는 읽기 전용 인스턴스는 상태로 가져도 상관 없다. (다른 싱글톤 빈 등)

### 스프링 빈의 스코프

빈이 생성되고, 존재하고, 적용되는 범위를 ***빈 스코프(bean scope)***라고 한다. 기본적으로 **스프링 빈은 싱글톤 스코프이며 강제러 제거하지 않은한 컨테이너가 존재하는 동안 계속 유지된다**.

## 1.7 의존관계 주입(DI)

### 제어의 역전과 의존관계 주입

스프링 IoC 기능의 대표적인 동작 원리는 주로 의존관계 주입이라고 불린다. 스프링이 여타 프레임워크와 차별화돼서 제공해주는 기능은 의존관계 주입이라는 용어를 사용할 때 분명하게 드러난다. 초기에는 주로 IoC 컨테이너라고 불리던 스프링이 지금은 DI 컨테이너라고 더 불리기도 한다.

### 런타임 의존관계 설정

의존이란 무슨 의미일까? 의존한다는 건 의존대상이 변하는 자신에게도 영향이 미친다는 뜻이다. 즉 A가 B에 의존하고 있다면 B가 변할 때 A가 영향을 받는다. 의존관계 주입은 다음 세가지 조건이 충족되는 작업을 말한다.

- 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. 그러기 위해서는 인터페이스에만 의존하고 있어야 한다.
- 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제 3의 존재가 결정한다.
- 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 주입해줌으로써 만들어진다.

```java
public class UserDao {
		private final ConnectionMaker connectionMaker;

		public UserDao(ConnectionMaker connectionMaker) {
				this.connectionMaker = connectionMaker;
		}
}
```

DI는 자신이 사용할 오브젝트에 대한 선택과 생성 제어권을 외부로 넘기고 자신은 수동적으로 주입받은 오브젝트를 사용한다는 점에서 IoC의 개념과 잘들어맞는다.

### 의존관계 주입의 응용

런타임 클래스에 대한 의존관계가 나타나지 않고 인터페이스를 통해 결합도가 낮은 코드를 만들었다. 변경을 통한 다양한 확장 방법에 자유롭다. 인터페이스만 구현하고 있다면 어떤 오브젝트라도 사용할 수 있다.
