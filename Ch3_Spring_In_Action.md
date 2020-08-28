## 1. JDBC를 사용해서 데이터 읽고 쓰기
   
**1-1. JDBC ?**  
: Java Database Connectivity, 자바에서 데이터베이스로 접근하게 해주는 API이다. 쉽게 말해 자바로 데이터를 CRUD 할 수 있게 해주는 애
   
**1-2. JdbcTemplate ?**     
: JDBC를 지원해주는 클래스

1-2-1. JdbcTemplate를 사용하지 않고 DB 쿼리
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
   if (resultSet.next()){
      ingredient = new Ingredient(
         resultSet.getString("id"),
         resultSet.getString("name").
         Ingredient.Type.valueOf(resultSet.getString("type")));
}
   return ingredient;
} catch (SQLException e){
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
