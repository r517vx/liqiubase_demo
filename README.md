# Настройки проекта и подключение liquibase
Настройки в файл _application.properties_
```
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver  
spring.datasource.url=jdbc:mysql://localhost:3306/my_db?createDatabaseIfNotExist=true&allowPublicKeyRetrieval=true&useSSL=false&useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC  
spring.datasource.username=root  
spring.datasource.password=  
  
spring.jpa.show-sql=true  
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect  
spring.jpa.hibernate.ddl-auto=none (или verify)
```

Добавляем в _pom.xml_ зависимость
```
<dependency>  
	<groupId>org.liquibase</groupId>  
	<artifactId>liquibase-core</artifactId>  
</dependency>
```
