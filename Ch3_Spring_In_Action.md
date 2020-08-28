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
   statement = connection.prepareStatement(
      "select id , name, type from Ingredient shere id = ?");
   statement.setString(1,id);
   resultSet = statement.executeQuery();
   Ingredient = ingredient = null;
   if (resultSet.next()){
      ingredient = new Ingredient(
         resultSet.getString("id"),
         resultSet.getString("name").
         Ingredient.Type.valueOf(resultSet.getString("type")));
}
   return ingredient;
} catch (SQLException e){
} finally {
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
