## 1. JDBC를 사용해서 데이터 읽고 쓰기
   
### 알아둘 것
   
**1-1. JDBC ?**  
: Java Database Connectivity, 자바에서 데이터베이스로 접근하게 해주는 API이다. 쉽게 말해 자바로 데이터를 CRUD 할 수 있게 해주는 애
   
**1-2. JdbcTemplate ?**     
: JDBC를 지원해주는 클래스

**1-2-1. JdbcTemplate를 사용하지 않고 DB 쿼리**
[참고](https://happynewmind.tistory.com/55)
```java
@Override
public Ingredient findById(String id){
   Connection connection = null;
   PreparedStatement statement = null;
   ResultSet resultSet = null;
try{
   connection = dataSource.getConnection();
   statement = connection.prepareStatement(                          //prepareStatement 생성
      "select id , name, type from Ingredient shere id = ?");        //물음표 값 나중에 넣겠다. 쿼리문 재사용할거다!
   statement.setString(1,id);                                        //쿼리문의 첫번째 물음표에 id를 넣겠다!
   resultSet = statement.executeQuery();                               //쿼리 실행
   Ingredient = ingredient = null;
   if (resultSet.next()){                                            //다음 레코드 읽기(여기서는 1번째 레코드 읽는것,처음에 resultSet은 0번째를 가리키고 있음)
      ingredient = new Ingredient(
         resultSet.getString("id"),                                  //첫번째 레코드의 id값가져와서 Ingredient 객체에 넣어줌
         resultSet.getString("name").
         Ingredient.Type.valueOf(resultSet.getString("type")));
}
   return ingredient;
} catch (SQLException e){                                            //DB 연결,쿼리문, 결과 오류 잡기
} finally {                                                          //prepareStatement 닫기
   if (resultSet != null){
      try {
         resultSet.close();
      } catch (SQLException e) {}
   }
   if(statement != null){
      try { 
      statement.close();
      } catch(SQLException e){}
   }
   if (connection != null) {
      try{
      connection.close();
      } catch (SQLException e) {}
      }
   }
   return null;
}
```
**1-2-2. JdbcTemplate 이용해서 DB 쿼리하기**
```java
private JdbcTemplate jdbc;

@Override
public Ingredient findById(String id){
   return jdbc.queryForObject(
      "select id, name, type from Ingredient shere id =?"
      this::mapRowToIngredient, id);
}

private Ingredient mapRowToIngredient(ResultSet rs, int rowNum)
   throws SQLException{
 return new Ingredient(
         rs.getString("id"),
         rs.getString("name"),
         Ingredient.Type.valueOf(rs.getString("type")));
}
```

**JdbcTemplate을 썼을 때**
```
- 쿼리문, 연결 담는 객체 생성 안했다.
- 결과 담는 객체는 있음! (mapRowToIngredient함수가 만들어줌)
- 예외 처리하는 catch도 없다.
```
  
    
    
### 프로젝트 적용
1. setting
- dependency 설정 : jdbctemplate 와 h2databse
  
2. 도메인 수정
: Taco, Order 수정
**main/tacos/Taco.java**
```java
'''
@Data
public class Taco {
	
	private Long id;
	private Date createdAt;

'''
}
```
**main/tacos/Order.java**
```java
'''
@Data
public class Order {
	
	private Long id;
	private Date placedAt;
'''
}
```


