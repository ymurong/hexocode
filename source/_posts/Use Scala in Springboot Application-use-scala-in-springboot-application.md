---
title: Use Scala in Springboot Application
date:  2019-10-15 12:00:00
tags:
- scala
---
> A great advantage of scala is that one could mix scala and java so that we coud easily integrate some new features developped in scala into some existing projet developped in Java, as they both run in JVM. Here is a simple guide of how to integrate scala features into an old springboot project. 


### A typical springboot application pom
A existing springboot projet pom file normally looks like this:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>example-boot-scala</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>example-boot-scala</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.12.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!--Add SpringData Dependancy-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>

		<!--Add Mysql Dependancy-->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>


</project>
```

### Scala Dependancy
Now we need to add a dependancy and a plugin to this pom file

```
<dependency>
	<groupId>org.scala-lang</groupId>
	<artifactId>scala-library</artifactId>
	<version>2.11.8</version>
</dependency>
```

```
<plugin>
    <groupId>net.alchim31.maven</groupId>
    <artifactId>scala-maven-plugin</artifactId>
    <version>3.2.1</version>
    <executions>
        <execution>
            <id>compile-scala</id>
            <phase>compile</phase>
            <goals>
                <goal>add-source</goal>
                <goal>compile</goal>
            </goals>
        </execution>
        <execution>
            <id>test-compile-scala</id>
            <phase>test-compile</phase>
            <goals>
                <goal>add-source</goal>
                <goal>testCompile</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <recompileMode>incremental</recompileMode>
        <scalaVersion>${scala.version}</scalaVersion>
        <args>
            <arg>-deprecation</arg>
        </args>
        <jvmArgs>
            <jvmArg>-Xms64m</jvmArg>
            <jvmArg>-Xmx1024m</jvmArg>
        </jvmArgs>
    </configuration>
</plugin>

```

### Scala Sources
Then we need to add a scala sources directory just like the java sources directory, ex: src/main/scala, so that all the controllers, services, repositories you developped will be included by springboot. What is cooler is that you could actually import java class into scala sources so that you could easily reuse some common sources.

### Comparison Java vs Scala in Springboot

#### Domain
Java

```
@Entity
@Table
public class MetaDatabase {

    @Id
    @GeneratedValue
    private Integer id;

    private String name;

    private String location;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getLocation() {
        return location;
    }

    public void setLocation(String location) {
        this.location = location;
    }
}
```

Scala
```
@Entity
@Table
class MetaTable {

  @Id
  @GeneratedValue
  @BeanProperty
  var id:Integer = _

  @BeanProperty
  var name:String = _

  @BeanProperty
  var tableType:String = _

  @BeanProperty
  var dbId:Integer = _
}

```

#### Repository
Java 
```
public interface MetaDatabaseRepository extends CrudRepository<MetaDatabase,Integer>{

}
```
Scala
```scala
trait MetaTableRepository extends CrudRepository[MetaTable, Integer]{

}
```

#### Service
Java
```
@Service
public class MetaDatabaseService {

    @Autowired
    private MetaDatabaseRepository metaDatabaseRepository;

    @Transactional
    public void save(MetaDatabase metaDatabase) {
        metaDatabaseRepository.save(metaDatabase);
    }

    public Iterable<MetaDatabase> query(){
        return metaDatabaseRepository.findAll();
    }

}
```

Scala
```
@Service
class MetaTableService @Autowired()(metaTableRepository:MetaTableRepository){

  @Transactional
  def save(metaTable:MetaTable) = {
    metaTableRepository.save(metaTable)
  }

  def query() = {
    metaTableRepository.findAll()
  }

}
```

#### RestController

Java
```
@RestController
@RequestMapping("/meta/database")
public class MetaDatabaseController {

    @Autowired
    private MetaDatabaseService metaDatabaseService;

    @RequestMapping(value = "/", method = RequestMethod.POST)
    public ResultVO save(@ModelAttribute MetaDatabase metaDatabase) {
        metaDatabaseService.save(metaDatabase);
        return ResultVOUtil.success();
    }

    @RequestMapping(value = "/", method = RequestMethod.GET)
    public ResultVO query() {
        return ResultVOUtil.success(metaDatabaseService.query());
    }
}
```

Scala
```
@RestController
@RequestMapping(Array("/meta/table"))
class MetaTableController @Autowired()(metaTableService: MetaTableService) {

  @RequestMapping(value = Array("/"), method = Array(RequestMethod.POST))
  @ResponseBody
  def save(@ModelAttribute metaTable:MetaTable) = {
    metaTableService.save(metaTable)
    ResultVOUtil.success()  // here scala calls java code
  }

  @RequestMapping(value = Array("/"), method = Array(RequestMethod.GET))
  @ResponseBody
  def query() = {
    ResultVOUtil.success(metaTableService.query())
  }

}
```


In summary, from these comparisons, we could see that adding scala features into a pre-existing java springboot project is not only feasible but also convenient without needing to do a lot of refactoring which could lead to potential risk. 

Enjoy using scala !


