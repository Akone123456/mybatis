# MyBatis用法

## Mybatis初步使用

**导入mybatis依赖**

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.7</version>
</dependency>
```

**配置mybatis-confing.xml文件**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 引入db.properties文件中属性的值 -->
    <properties resource="db.properties"></properties>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <!-- 需要配置相关的Mapper文件 -->
    <mappers>
        <mapper resource="StudentMapper.xml"/>
    </mappers>
</configuration>
```

**配置db.properties文件**

```properties
username=root
password=root
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/test?serverTimezone=GMT
```

**引入mysql依赖和junit单元测试依赖**

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.27</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

**实体类**

```java
package com.xzit.pojo;

public class Student {
    private  Integer id;
    private  String name;
    private  Integer age;
    private  Integer weight;

    public Student() {
    }

    public Student(Integer id, String name, Integer age, Integer weight) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.weight = weight;
    }

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

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Integer getWeight() {
        return weight;
    }

    public void setWeight(Integer weight) {
        this.weight = weight;
    }

    @Override
    public String toString() {
        return "student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", weight=" + weight +
                '}';
    }
}
```

**Dao层:StudentDao**

```java
package com.xzit.dao;

import com.xzit.pojo.Student;
import org.apache.ibatis.annotations.Select;

public interface StudentDao {


    //查询
    public Student selectById (Integer id);
    //新增
    public Integer save(Student student);
    //删除
    public Integer delete(Integer id);
    //修改
    public Integer update(Student student);
}

```

**映射的SQL语句:StudentDao.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xzit.dao.StudentDao">
    <!--查询-->
    <select id="selectById" resultType="com.xzit.pojo.Student">
        select * from student where id=#{id}
    </select>
	<!--新增-->
    <insert id="save" >
        insert into student(name,age)
        values(#{name},#{age})
    </insert>
	<!--修改-->
    <update id="update">
        update student set age=#{age} where id=#{id}
    </update>
	<!--删除-->
    <delete id="delete">
        delete  from student where id=#{id}
    </delete>
</mapper>
```

**测试**

```java
import com.xzit.dao.StudentDao;
import com.xzit.pojo.Student;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;

public class TestStudent {
    SqlSessionFactory sqlSessionFactory=null;

    /*
    @Before：初始化方法   对于每一个测试方法都要执行一次
     */
    @Before
    public void init() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        sqlSessionFactory=new SqlSessionFactoryBuilder().build(inputStream);
    }

    /*
    查询
     */
    @Test
    public void test01()  {

        SqlSession session = sqlSessionFactory.openSession();
        StudentDao mapper = session.getMapper(StudentDao.class);
        Student student = mapper.selectById(3);
        System.out.println(student);
    }

    /*
    新增
     */
    @Test
    public void test02()  {

        SqlSession session = sqlSessionFactory.openSession(true);
        StudentDao mapper = session.getMapper(StudentDao.class);
        Student student = new Student();
        student.setId(4);
        student.setAge(20);
        Integer save = mapper.save(student);
        System.out.println(save);
        session.commit();
    }

    /*
    修改
     */
    @Test
    public void test03()  {

        SqlSession session = sqlSessionFactory.openSession(true);
        StudentDao mapper = session.getMapper(StudentDao.class);
        Student student = new Student();
        student.setId(1);
        student.setAge(100);
        Integer save = mapper.update(student);
        System.out.println(save);
        session.commit();
    }
    /*
    删除
     */
    @Test
    public void test04()  {

        SqlSession session = sqlSessionFactory.openSession(true);
        StudentDao mapper = session.getMapper(StudentDao.class);
        Integer delete = mapper.delete(5);
        System.out.println(delete);
        session.commit();
    }



}
```
**使用SqlSessionFactory必须手动提交sql**

```xml
session.commit();
<!--或者 设置autocommit=true -->
sqlSessionFactory.openSession(true);
```

**注意: 注解@和xml文件不能一起使用，只能使用其中一个**

```java
@Select("select * from student where id =#{id}")
public Student selectById (Integer id);
```

## 日志使用

**引入log4j日志依赖**

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

**添加log4j.properties文件**

修改的两处

* log4j.rootLogger    如果需要查看语句错误地方，改成DEBUG模式即可
* log4j.logger.com.xzit.dao.StudentMapper    日志配置在项目的Dao层



```properties
# 全局日志配置 
# 此处需要更改  如果需要查看语句错误改成DEBUG
log4j.rootLogger=INFO, stdout
# MyBatis 日志配置  
# 此处需要更改，指向项目的dao层
log4j.logger.com.xzit.dao.StudentMapper=TRACE
# 控制台输出
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```
## Mybatis-confing配置文件属性

**属性(properties)**

##### 

```xml
<!--
    当需要引入外部的配置文件的时候，可以使用这样的方式
    resource:表示从当前项目的类路径中进行加载，如果用的是idea指的是resource资源目录下的配置文件
	url:可以从当前文件系统的磁盘目录查找配置，也可以从网络上的资源进行引入
-->
    <properties resource="db.properties" ></properties>

```

##### 

**设置(Setting)**

```xml
<!--可以影响mybatis运行时的行为，包含N多个属性，需要什么引入什么-->

<settings>
        <!--开启驼峰标识验证-->
    	<!--即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>



```

**类型别名(typeAliases)**

```xml
<!--typeAliases:表示在引入实体类的名称时候，可以使用别名，而不需要写完全限定名-->
<typeAliases>
        <!--每一个具体的类都需要单独来写，如果有100个类呢？-->
        <!--当这样配置时，Emp 可以用在任何使用 com.mashibing.bean.Emp 的地方。 -->
-        <typeAlias type="com.mashibing.bean.Emp" alias="Emp"></typeAlias>
    
        <!--可以指定具体的包来保证实体类不需要写完全限定名-->
        <!--在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。 比如 domain.blog.Author 的别名为 author；-->
        <!--在使用注解的情况下，比如@Alias("author")，则别名为其注解值-->
        <package name="com.mashibing.bean"/>
</typeAliases>

```

**环境配置(environments)**

```xml
 <!--在项目开发过程中，会包含开发环境，测试环境，生产环境，有可能会使用不同的数据源进行连接操作，
    在此配置文件中可以指定多个环境
    default:表示选择哪个环境作为运行时环境
  -->
<environments default="development">
        <!--配置具体的环境属性
        id:表示当前环境的名称
        -->
      <environment id="development">
            <!--事务管理器，每一种数据源都需要配置具体的事务管理器
            type:表示事务管理器的类型
            jdbc:使用jdbc原生的事务控制
            managed:什么都没做
            -->
          <transactionManager type="JDBC"/>
            <!--配置具体的数据源的类型
            type:表示数据源的类型
            pooled:使用数据库连接池
            unpooled：每次都打开和关闭一个链接
            -->
          <dataSource type="POOLED">
                <!--链接数据的时候需要添加的必备的参数，一般是四个，如果是连接池的话，可以设置连接最大个数等相关信息-->
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
          </dataSource>
      </environment>
</environments>

```

**映射器(mappers)**

```xml
 <!--是来将mapper映射文件引入到配置文件中，方便程序启动的时候进行加载
    每次在进行填写的时候需要注意，写完xml映射之后一定要添加到mybatis-config文件中

    resource:从项目的类路径下加载对应的映射文件
    url:从本地磁盘目录或者网络中引入映射文件
    class:可以直接引入类的完全限定名，可以使用注解的方式进行使用,
            如果不想以注解的方式引入呢？
                如果想要class的方式引入配置文件，可以将xml文件添加到具体的类的同级目录下
              1、      如果是maven的项目的话，需要添加如下配置，因为maven默认只会编译java文件，需要把xml文件也添加到指定目录中
                    <build>
                        <resources>
                            <resource>
                                <directory>src/main/java</directory>
                                <includes>
                                    <include>**/*.xml</include>
                                </includes>
                            </resource>
                        </resources>
                    </build>
              2、在resource资源目录下，创建跟dao层一样的同级目录即可，将配置文件放到指定的目录
    -->
    <mappers>
<!--        <mapper resource="EmpDao.xml" />-->
<!--        <mapper resource="UserDao.xml"/>-->
<!--        <mapper class="com.mashibing.dao.UserDaoAnnotation"></mapper>-->
<!--        <mapper class="com.mashibing.dao.UserDao"></mapper>-->
        <!--如果需要引入多个配置文件，可以直接定义包的名称
        resource目录下配置的映射文件必须要具体相同的目录
        -->
        <package name="com.mashibing.dao"/>
    </mappers>

```

## **XML映射器**

**insert,delete和update**