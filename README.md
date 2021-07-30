# Настройки проекта и подключение liquibase
*Нужно быть внимательными Hibernate и liquibase генерят немного разные названия колонок. При накатывании changeset колонка дропнется вместе со всеми данными*

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
Теперь необходимо добавить сам Liquibase скрипт, который будет создавать нужную нам таблицу. 
Создаем в папке **/src/main/resources/db/changelog** файл с именем **db.changelog-master.yaml** и добавляем в него следующее содержимое
<details>
	<summary>Open spoiler</summary>
	
```java
databaseChangeLog:
  - logicalFilePath: db/changelog/db.changelog-lesson1.yaml
  - changeSet:
      id: 1
      author: your_liquibase_username
      changes:
        - createTable:
            tableName: users
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
                    nullable: false
              - column:
                  name: username
                  type: varchar(50)
                  constraints:
                    unique: true
                    nullable: false
              - column:
                  name: password
                  type: varchar(512)
                  constraints:
                    nullable: false
              - column:
                  name: first_name
                  type: varchar(50)
              - column:
                  name: last_name
                  type: varchar(50)
              - column:
                  name: email
                  type: varchar(50)	
```
</details>
Это если создавать БД с чистого листа и changeset-ами.
Но задача стоит немного иначе.
Уже есть схема БД созданная Hibernate-ом по Entity 
Нужно создавать changeset по измененным Entity, а потом получившийся diff - changeset накатывать на БД с помощью liquibase

Для этого добваляем maven-plugin
<details>
	<summary>секция build файла pom.xml</summary>
	
```
	<build>
		<resources>
			<resource>
				<directory>src/main/resources</directory>
				<filtering>true</filtering>
				<includes>
					<include>*.properties</include>
				</includes>
			</resource>
			<resource>
				<directory>src/main/resources</directory>
				<filtering>false</filtering>
				<includes>
					<include>**/*.*</include>
				</includes>
			</resource>
		</resources>

		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>

			<plugin>
				<groupId>org.liquibase</groupId>
				<artifactId>liquibase-maven-plugin</artifactId>
				<version>3.5.5</version>
				<configuration>
					<propertyFile>${project.build.outputDirectory}/liquibase-maven-plugin.properties</propertyFile>
					<systemProperties>
						<user.name>your_liquibase_username</user.name>
					</systemProperties>
					<logging>info</logging>
				</configuration>
				<dependencies>
					<dependency>
						<groupId>org.liquibase.ext</groupId>
						<artifactId>liquibase-hibernate5</artifactId>
						<version>3.6</version>
					</dependency>
					<dependency>
						<groupId>org.springframework.boot</groupId>
						<artifactId>spring-boot-starter-data-jpa</artifactId>
						<version>2.1.5.RELEASE</version>
					</dependency>
					<dependency>
						<groupId>javax.validation</groupId>
						<artifactId>validation-api</artifactId>
						<version>2.0.1.Final</version>
					</dependency>
					<dependency>
						<groupId>org.javassist</groupId>
						<artifactId>javassist</artifactId>
						<version>3.24.0-GA</version>
					</dependency>
					<dependency>
						<groupId>org.yaml</groupId>
						<artifactId>snakeyaml</artifactId>
						<version>1.12</version>
					</dependency>
				</dependencies>
			</plugin>
		</plugins>
	</build>
```
</details>

<details>
	<summary>Файл pom.xml целиком</summary>
	
```
	<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.5.3</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>liquibase_demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>liquibase_demo</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>11</java.version>
		<timestamp>${maven.build.timestamp}</timestamp>
		<maven.build.timestamp.format>yyyyMMdd-HHmmssSSS</maven.build.timestamp.format>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<version>1.18.20</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.liquibase</groupId>
			<artifactId>liquibase-core</artifactId>
		</dependency>

	</dependencies>

	<build>
		<resources>
			<resource>
				<directory>src/main/resources</directory>
				<filtering>true</filtering>
				<includes>
					<include>*.properties</include>
				</includes>
			</resource>
			<resource>
				<directory>src/main/resources</directory>
				<filtering>false</filtering>
				<includes>
					<include>**/*.*</include>
				</includes>
			</resource>
		</resources>

		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>

			<plugin>
				<groupId>org.liquibase</groupId>
				<artifactId>liquibase-maven-plugin</artifactId>
				<version>3.5.5</version>
				<configuration>
					<propertyFile>${project.build.outputDirectory}/liquibase-maven-plugin.properties</propertyFile>
					<systemProperties>
						<user.name>your_liquibase_username</user.name>
					</systemProperties>
					<logging>info</logging>
				</configuration>
				<dependencies>
					<dependency>
						<groupId>org.liquibase.ext</groupId>
						<artifactId>liquibase-hibernate5</artifactId>
						<version>3.6</version>
					</dependency>
					<dependency>
						<groupId>org.springframework.boot</groupId>
						<artifactId>spring-boot-starter-data-jpa</artifactId>
						<version>2.1.5.RELEASE</version>
					</dependency>
					<dependency>
						<groupId>javax.validation</groupId>
						<artifactId>validation-api</artifactId>
						<version>2.0.1.Final</version>
					</dependency>
					<dependency>
						<groupId>org.javassist</groupId>
						<artifactId>javassist</artifactId>
						<version>3.24.0-GA</version>
					</dependency>
					<dependency>
						<groupId>org.yaml</groupId>
						<artifactId>snakeyaml</artifactId>
						<version>1.12</version>
					</dependency>
				</dependencies>
			</plugin>
		</plugins>
	</build>

</project>
```
</details>

Убедимся что есть директория **src/main/resources/db/changelog**

В **resources** нужен файл **liquibase-maven-plugin.properties** с настройками плагина.

```
changeLogFile= @project.basedir@/src/main/resources/db/changelog/db.changelog-master.yaml
url= jdbc:mysql://localhost:3306/geek_db?createDatabaseIfNotExist=true&allowPublicKeyRetrieval=true&useSSL=false&useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
username= root
password=
driver= com.mysql.cj.jdbc.Driver
referenceUrl=hibernate:spring:com.example.liquibase_demo.model?dialect=org.hibernate.dialect.MySQLDialect
diffChangeLogFile= @project.basedir@/src/main/resources/db/changelog/db.changelog-@timestamp@.yaml
ignoreClasspathPrefix= true
```

Обращаем внимание на правильность пакетного пути до наших Entity в параметре referenceUrl
В файле **db.changelog-master.yaml** перед первым запуском не должно быть changeset-ов, чтобы не убить существующую схему БД.
Запускаем:
- mvn clean
- install
- liquibase:diff

Это удобно вызывать из меню mavem справа вверху в idea.

Если Entity и схема БД у нас разные (мы добавили/исправили Entity) в папке **src/main/resources/db/changelog** у нас появится diff-файл с changeset-ом
Этот changeset после проверки необходимо накатить на БД. Для этого инклюдим его в **db.changelog-master.yaml**
Например:
```
- include:
      file: db.changelog-20210729-054643592.yaml
      relativeToChangelogFile: true
```
Перезапускаем приложение изменения должны накатиться на БД.

В итоге получаем такую схему:
1. Изменяем/добавляем Entity
2. Получаем diff-файл с changeset-ом
3. Инклюдим cangeset в **db.changelog-master.yaml** и перезапускаем приложение
4. Изменения накатываются на БД

Links:

[https://habr.com/ru/post/460377/](https://habr.com/ru/post/460377/)

[https://habr.com/ru/post/460907/](https://habr.com/ru/post/460907/)
