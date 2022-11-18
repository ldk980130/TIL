탬플릿이란 바뀌는 성질이 다른 코드 중에서 변경이 거의 일어나지 않으며 일정한 패턴으로 유지되는 특성을 가진 부분을 자유롭게 변경되는 성질을 가진 부분으로부터 독립시켜서 효과적으로 활용할 수 있도록 하는 방법이다.

## 3.4 컨텍스트와 DI

### 기존의 JDBC 사용

```java
Connection c = null;
PreparedStatement ps = null;
ResultSet rs = null;

try {
	c = dataSource.getConnection();

	// 변하는 부분 
	ps = c.prepareStatement("select count(*) from users");

	rs = ps.executeQuery();
	rs.next();
	return rs.getInt(1);
	// ##########
} catch (SQLException e) {
	throw e;
} finally {
    if (rs != null) {
        try{
            rs.close();
        } catch (SQLException e) {
    	}
    }
    if (ps != null) {
        try {
            ps.close();
        } catch (SQLException e) {
        }
    }
    if (c != null) {
        try {
            c.close();
        } catch (SQLException e){
        }
    }
}

```

### 클래스 분리 (탬플릿 콜백 패턴 적용)

```java
public class JdbcContext {
		private DataSource dataSource;

		public void setDataSource(DataSource dataSource) {
				this.dataSource = dataSource;
		}

		public void workWithStatementStrategy(StatementStrategy stmt) throws SQLException {
				Connection c = null;
				PreparedStatement ps = null;

				try {
					c = this.dataSource.getConnection();
					ps = stmt.makePreparedStatement(c);
					ps.executeUpdate();
				} catch (SQLException e) {
					throw e;
				} finally {
					// close connection, preparedStatement...
				}
		}
```

```java
public class UserDao {
		
		private JdbcContext jdbcContext;

		public void setJdbcContext(JdbcContext jdbcContext) {
		    this.jdbcContext = jdbcContext;
		}

		public void add(final User user) throws SQLException {
		    jdbcContext.workWithWtatemetStrategy(
		        new StatementStrategy() {...} // 변하는 로직 구현
                    );
        }
}
```

### 3.4.2JdbcContext의 특별한 DI

`UserDao`는 인터페이스를 거치지 않고 코드에서 바로 `JdbcContext` 클래스를 사용하고 있다. 클래스 레벨에서 의존 관계가 결졍되는 것이다. DI 방식이긴 하지만 의존 오브젝트의 구현 클래스를 변경할 수 없다.

의존 관계 주입(DI)이라는 개념을 충실히 따르면 인터페이스를 사용해서 클래스 레벨에서는 보이지 않는 의존 관계가 런타임에 다이나믹하게 주입 되어야 한다. 하지만 스프링의 DI는 넓게 봤을 때 객체의 생성과 관계 설정에 대한 제어권을 외부로 위임했다는 IoC라는 개념을 포괄하기 때문에 `JdbcContext`는 DI의 기본을 따르고 있다고 볼 수 있다.

**JdbcContext를 스프링 빈으로 해야하는 이유**

1. 싱글톤 빈으로 관리할 수 있다. `JdbcContext`는 `DataSource`라는 인스턴수 변수는 있지만 `DataSource`는 읽기 전용이기 때문에 싱글톤이 되는 데 문제가 없다.
2. `JdbcContext`가 DI를 통해 다른 빈에 의존하고 있다. 스프링 DI를 위해서는 오브젝트와주입받는 오브젝트 양쪽 다 빈이어야 한다.

## 3.5 탬플릿과 콜백

탬플릿은 고정된 작업 흐름을 가진 코드를 재사용한다는 의미에서 붙인 이름이다. 콜백은 탬플릿 안에서 호출되는 것을 목적으로 만들어진 오브젝트를 말한다.
