# JDBC를 사용해서 데이터 읽고 쓰기
   
## 알아둘 것
   
**1. JDBC ?**  
: Java Database Connectivity, 자바에서 데이터베이스로 접근하게 해주는 API이다. 쉽게 말해 자바로 데이터를 CRUD 할 수 있게 해주는 애
   
**2. JdbcTemplate ?**     
: JDBC를 지원해주는 클래스

**2-1. JdbcTemplate를 사용하지 않고 DB 쿼리**
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
**2-2. JdbcTemplate 이용해서 DB 쿼리하기**
```java
private JdbcTemplate jdbc;

@Override
public Ingredient findById(String id){
   return jdbc.queryForObject(								//쿼리 실행결과가 1개일때 queryForObject() 사용, 여러개면 query()사용
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
  
    
    
## 프로젝트 적용
: DesignController에서 ingredient를 정의해 주는거 말고 DB에서 가져와 볼 것이다!   
  
1. setting
- dependency 설정 : jdbctemplate 와 h2databse(내장 데이터베이스)
  
2. 도메인 수정(식별자 id 만들어줌)    
: Taco, Order 수정. 
**#main/tacos/Taco.java**
```java
...
@Data
public class Taco {
	
	private Long id;
	private Date createdAt;

...
}
```
**#main/tacos/Order.java**
```java
...
@Data
public class Order {
	
	private Long id;
	private Date placedAt;
...
}
```

3. 리퍼지터리 생성  
3-1. IngredientRepository   
:인터페이스 생성 [참고](https://wikidocs.net/217)  
**#main/data/IngredientRepository.java**
```java
package tacos.data;

import tacos.Ingredient;

public interface IngredientRepository {

	Iterable<Ingredient> findAll();

	Ingredient findById(String id);

	Ingredient save(Ingredient ingredient);

}
```
3-2. JdbcIngredientRepository  
: ingredient 데이터 가져오고 저장하는 함수들 정의.  
```java
package tacos.data;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Repository;

import java.sql.ResultSet;
import java.sql.SQLException;

import tacos.Ingredient;

@Repository										//Component annotation 중 db 관련 annotation
public class JdbcIngredientRepository implements IngredientRepository {			//3-1에서 구현한 ingredientRepository 인터페이스 구현

	private JdbcTemplate jdbc;

	@Autowired									//Autowired annotation으로 jdbcTemplate에 연결
	public JdbcIngredientRepository(JdbcTemplate jdbc) {
	  this.jdbc = jdbc;
	}

	@Override
	  public Iterable<Ingredient> findAll() {
	    return jdbc.query("select id, name, type from Ingredient",			//jdbc.query함수는 List형태로 쿼리결과를 return, 
	      this::mapRowToIngredient);						//쿼리 결과 행 개수만큼 호출됨. 한 번 호출할 때 마다 ingredient 객체 생성
	  }										//모든 객체 만들어지면 List로 return됨

	  @Override
	  public Ingredient findById(String id) {
	    return jdbc.queryForObject(
	      "select id, name, type from Ingredient where id=?",
	        this::mapRowToIngredient, id);
	  }

	  private Ingredient mapRowToIngredient(ResultSet rs, int rowNum)
	    throws SQLException {
	      return new Ingredient(
		    rs.getString("id"),
		    rs.getString("name"),
		    Ingredient.Type.valueOf(rs.getString("type")));
	  }

	  @Override
	  public Ingredient save(Ingredient ingredient) {				//쿼리 결과를 저장
	    jdbc.update(
	        "insert into Ingredient (id, name, type) values (?, ?, ?)",
	        ingredient.getId(),							//첫번째 물음표에 들어갈 값
	        ingredient.getName(),							//두번째
	        ingredient.getType().toString());					//세번째
	    return ingredient;
	  }

}
```
3-3. DesignTacoController 수정(p83)   
```
- chapter2에서 하드코딩 했던 ingredients List를 삭제해주기  
- 데이터 다루는 함수 쓰기 위해 IngredientRepository 객체 생성해주기  
- DB에서 ingredient 가져와서 List만들어주기  
```
3-4. Table 정의
```
- classpath 루트 경로에 schema.sql 생성 (src/main/resources폴더에)
```
```sql
create table if not exists Ingredient (
  id varchar(4) not null,
  name varchar(25) not null,
  type varchar(10) not null
);

create table if not exists Taco (
  id identity,
  name varchar(50) not null,
  createdAt timestamp not null
);

create table if not exists Taco_Ingredients (
  taco bigint not null,
  ingredient varchar(4) not null
);

alter table Taco_Ingredients
    add foreign key (taco) references Taco(id);
alter table Taco_Ingredients
    add foreign key (ingredient) references Ingredient(id);

create table if not exists Taco_Order (
  id identity,
  deliveryName varchar(50) not null,
  deliveryStreet varchar(50) not null,
  deliveryCity varchar(50) not null,
  deliveryState varchar(2) not null,
  deliveryZip varchar(10) not null,
  ccNumber varchar(16) not null,
  ccExpiration varchar(5) not null,
  ccCVV varchar(3) not null,
  placedAt timestamp not null
);

create table if not exists Taco_Order_Tacos (
  tacoOrder bigint not null,
  taco bigint not null
);

alter table Taco_Order_Tacos
    add foreign key (tacoOrder) references Taco_Order(id);
alter table Taco_Order_Tacos
    add foreign key (taco) references Taco(id);
```
3-5. Data 저장
```
- src/main/resource 폴더에 data.sql 파일 생성 후 insert 이용해서 데이터 추가
```
```sql
delete from Taco_Order_Tacos;
delete from Taco_Ingredients;
delete from Taco;
delete from Taco_Order;

delete from Ingredient;
insert into Ingredient (id, name, type)
    values ('FLTO', 'Flour Tortilla 토르티아', 'WRAP');
insert into Ingredient (id, name, type)
    values ('COTO', 'Corn Tortilla', 'WRAP');
insert into Ingredient (id, name, type)
    values ('GRBF', 'Ground Beef', 'PROTEIN');
insert into Ingredient (id, name, type)
    values ('CARN', 'Carnitas', 'PROTEIN');
insert into Ingredient (id, name, type)
    values ('TMTO', 'Diced Tomatoes', 'VEGGIES');
insert into Ingredient (id, name, type)
    values ('LETC', 'Lettuce', 'VEGGIES');
insert into Ingredient (id, name, type)
    values ('CHED', 'Cheddar', 'CHEESE');
insert into Ingredient (id, name, type)
    values ('JACK', 'Monterrey Jack', 'CHEESE');
insert into Ingredient (id, name, type)
    values ('SLSA', 'Salsa', 'SAUCE');
insert into Ingredient (id, name, type)
    values ('SRCR', 'Sour Cream', 'SAUCE');
```
3-5. Taco 정보와 주문 정보 저장해 주는 함수 필요
```
- TacoRepository 만들기 (같이 미리 고려해야할 것 : Taco를 만드는 식재료도 같이 Taco_Ingredients 테이블에 저장해야함!)
- OrderRepository 만들기 (같이 미리 고려해야할 것 : 주문 들어오면 어떤 타코들인지 정보도 Taco_Order_Tacos 테이블에 저장해야함!)
```
3-5-1. Taco 정보 저장해주기   
3-5-1-1. TacoRepository   
: Taco 정보 저장해 줄 함수 적은 인터페이스
```java
package tacos.data;

import tacos.Taco;

public interface TacoRepository {

	Taco save(Taco design);
	
}
```
3-5-1-2. JdbcTacoRepository   
: TacoRepository(3-5-1-1)의 save함수 구현   
```java

```
3-5-2. OrderRepository   
``java
package tacos.data;

import tacos.Order;

public interface OrderRepository {

	Order save(Order order);
	
}
```
