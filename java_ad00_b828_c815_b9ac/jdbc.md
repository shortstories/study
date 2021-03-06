# JDBC

자바에서 RDB에 접근하기 위해서 사용하는 API. Java Database Connectivity의 줄임말이다. 자바 프로그램이 특정 DB에 종속되지 않고 연결성을 유지할 수 있도록 도와준다.

## 용도

1. 데이터베이스와의 Connection 만들기
2. SQL문의 생성
3. SQL문의 실행
4. 결과를 확인하고 수정

## 구조

1. JDBC API
   * 어플리케이션과 JDBC Driver Manager 사이에서 사용되는 API
2. JDBC Driver API
   * JDBC Driver Manager과 Driver 사이에서 사용되는 API

일반적으로 JDBC는 Driver manager을 통해서 특정 데이터베이스에 알맞는 드라이버를 고르고, 그 드라이버를 통해서 데이터베이스와 연결을 맺는다. Driver manager는 다양한 종류의 데이터베이스를 동시에 연결할 수 있게 도와주기도 한다.

```text
                                   +-----+      +------------------------+
                                   |     |      |                        |
                            +----> |     +------+                        |
                            |      +-----+      +------------------------+
                            |
                            |
+-------------+       +-----+      +-----+      +------------------------+
|             |       |     |      |     |      |                        |
|      +------+       | D M +----> |     +------+                        |
|      |      |       | r a |      +-----+      +------------------------+
|      | JDBC +-------+ i n |
|      |      |       | v a |      +-----+      +------------------------+
|      |      |       | e g +----> |     |      |                        |
|      +------+       | r e |      |     +------+                        |
|             |       |   r |      +-----+      +------------------------+
+-------------+       +-----+
                            |      +-----+      +------------------------+
                            |      |     |      |                        |
                            +----> |     +------+                        |
                                   +-----+      +------------------------+

                                   Drivers         Databases
```

## 구성

* DriverManager : 데이터베이스 드라이버들의 리스트를 관리함. 어플리케이션에서 데이터베이스 Connection을 요청하면 적절한 subprotocol을 가지는 드라이버를 선택하여 맞춰주는 역할 수행. 여러 드라이버가 한 subprotocol에 해당된다면 가장 먼저 나오는 드라이버로 선택하여 Connection 구축.
* Driver : 데이터베이스 서버와 어플리케이션 간 커뮤니케이션을 담당하는 인터페이스. 직접 Driver 객체를 만지게 될 일은 거의 없음. 보통은 DriverManager을 통해서 Driver들을 관리하게 됨. 
* Connection : 데이터베이스에 접촉하기 위한 메소드들을 가지고 있는 인터페이스. 이 인터페이스를 통해서 만들어진 객체는 커뮤니케이션 context 그 자체를 표현하고 있음. 데이터베이스와 이루어지는 모든 커뮤니케이션은 connection 오브젝트를 통해서 이루어지게 됨.
* Statement : 데이터베이스에 SQL문을 보내기 위해 사용하는 인터페이스. 어떤 인터페이스들은 stored procedure들을 실행하기 위해 별도의 패러미터들을 받기도 함.
* ResultSet : Statement 객체를 사용해서 SQL문을 실행한 다음, 그 결과를 가지고 있는 객체. Iterator처럼 동작하곤 한다.
* SQLException : 데이터베이스에서 발생하는 모든 에러를 처리하는 Exception.

## 사용법

### Driver 선택

1. JDBC-ODBC Bridge Driver : JDBC가 그저 ODBC의 브릿지인 것 처럼 동작하는 드라이버. 과거에는 ODBC 드라이버만 제공하는 데이터베이스가 많았기에 쓸 일이 많았지만 지금은 다른 대안도 많기 때문에 고려해볼 상황이 거의 없다.
2. JDBC-Native API : JDBC가 native C/C++ API를 호출하게 동작하는 드라이버. 근본적으로는 ODBC 브릿지랑 크게 다를 것이 없다. 다만 성능 면에서 좀 더 나아지긴 했다. 마찬가지로 쓸 일이 거의 없다.
3. JDBC-Net pure Java : 일종의 프록시 역할을 하는 미들웨어 서버를 하나 두고, JDBC 클라이언트는 네트워크 소켓을 열어 그 미들웨어 서버와 통신하게 하는 방식. 실제로 데이터베이스와 통신하여 조작을 수행하는 것은 이 미들웨어 서버이다. 클라이언트쪽에 별도의 설정을 할 필요가 없다. 또한 유연하고 확장하기 쉬운 방식이다.
4. Pure Java : 소켓 커넥션을 통해서 자바 기반 드라이버가 직접 데이터베이스와 통신하는 방식이다. 가장 성능이 좋으며 보통 벤더가 직접 제공하는 방식이기도 하다. 클라이언트나 서버에 별도의 프로그램을 설치할 필요가 없고, 유연성이 좋으며, 동적으로 드라이버들을 다운받는 것도 가능하다.

보통 4를 쓰는 것이 가장 좋다. 그런데 만약 어플리케이션이 여러 데이터베이스에 동시에 접근해야한다면 3도 고려해봄직 하다. 2는 3이나 4가 아직 지원되지 않는 경우에만 사용하는 것이 바람직하며, 1은 되도록 쓰지 않는 것이 좋다.

