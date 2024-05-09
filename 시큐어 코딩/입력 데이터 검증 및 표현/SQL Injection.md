SQL 삽입(Injection)
====

## SQL 삽입

SQL 삽입이란? 프로그램에 SQL을 삽입하여 내부 DB 서버의 데이터를 유출 및 변조 혹은 관리자 인증을 우회하는 보안 약점

예를 들어 클라이언트에게 제공하는 화면단 중 input값을 받는 곳이 있다면(쿼리,파라미터도 포함) 해당 input 태그에 sql문을 삽입하여 DB로부터 정보를 열람하거나 조작할 수 있다.

### 문제 원인

사용자로부터 입력된 값을 필터링 과정 없이 넘겨받아 동적 쿼리를 생성했기에 문제가 된다.

동적 쿼리에 사용되는 입력 데이터에 예약어 및 특수문자가 입력되지 않게 필터링 되도록 설정하여 방지할 수 있다. (*동적쿼리 = 질의어 코드를 문자열 변수에 넣어 조건에 따라 질의를 동적으로 변경하여 처리하는 방식)

### 실제 사례

#### 잘못된 코드 사례(input값 매핑)

(* orm을 쓰지않고 직접 java.sql 패키지 중 PreparedStatement클래스를 사용하는 것을 기준으로 설명)
  
```

String tableName = props.getProperty("jdbc.tableName");
String name = props.getProperty("jdbc.name");
String query = "SELECT * FROM " + tableName + " WHERE Name =" + name;
stmt = con.prepareStatement(query);
rs = stmt.executeQuery();
...

```

해당 코드는 외부로부터 tableName과 name을 받아 SQL 쿼리를 생성하고 있으며 어떠한 예외처리도 안하고 있다.

#### 안전하게 바꾸려면

```

String tableName = props.getProperty("jdbc.tableName");
String name = props.getProperty("jdbc.name");
String query = "SELECT * FROM ? WHERE Name = ?";
stmt = con.prepareStatement(query);
stmt.setString(1, tableName); stmt.setString(2, name);
rs = stmt.executeQuery();
...

```

인자부분을 setXXX 메소드로 설정하여, 외부의 입력이 쿼리문의 구조를 바꾸는 것을 방지했다.

PS) 그외에도 HttpServlet를 사용하는 경우 HttpServletRequest에서 값을 받아오는 경우 파라미터에 대한 검증 없이 해당 파라미터를 바로 동적쿼리에 할당하는 경우도 위험사례이다.
그러나 요즘에는 HttpServlet를 직접 상속받는 방식보단 SpringMVC를 활용하기때문에 이 사례는 패스한다.


안전하게 인자값을 받기 위해서는 

- 인자의 길이 제한
- SQL문에서 쓰이는 예약어의 삽입 제한
- 알파벳과 숫자를 제외한 문자의 사용을 제한 (공격 구문을 작성할때 특수문자를 사용할 가능성이 높다)


**추천 정규식**

```
[^\\p{Alnum}] | select | delete | update | insert | create | alter |drop
```

-  ^\\p{Alnum} = 알파벳과 숫자를 제외한 나머지 문자를 의미
- 그외에는 SQL 문에서 사용되는 예약어들이다.


### 최신 트렌드를 반영한 방지 기법

사실 요즘엔 jpa나 mybatis등 orm이 잘되어있기때문에 SQL Injection에 대해 생소해 하는 개발자들도 있을것이다.
그렇다면 (비교적) 최신 기술인 ORM은이러한 공격에 대해 완벽하게 대비가 되어있는걸까?

#### MyBatis를 사용할때 생길 수 있는 문제

MyBatis의 경우 