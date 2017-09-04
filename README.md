# myBatis框架内容

### mybatis的介绍
mybatis就是一个封装来jdbc的持久层框架，它和hibernate都属于ORM框架，但是具体的说，hibernate是一个完全的orm框架，而mybatis是一个不完全的orm框架。
Mybatis让程序员只关注sql本身，而不需要去关注如连接的创建、statement的创建等操作。Mybatis会将输入参数、输出结果进行映射。

#### - 分析jdbc的问题
```
 原生态的jdbc代码
public static void main(String[] args) {
			Connection connection = null;
			PreparedStatement preparedStatement = null;
			ResultSet resultSet = null;
			
			try {
				//1、加载数据库驱动
				Class.forName("com.mysql.jdbc.Driver");
				//2、通过驱动管理类获取数据库链接
				connection =  DriverManager.getConnection("jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8", "root", "root");
				//3、定义sql语句 ?表示占位符
			String sql = "select * from user where username = ?";
				//4、获取预处理statement
				preparedStatement = connection.prepareStatement(sql);
				//5、设置参数，第一个参数为sql语句中参数的序号（从1开始），第二个参数为设置的参数值
				preparedStatement.setString(1, "王五");
				//6、向数据库发出sql执行查询，查询出结果集
				resultSet =  preparedStatement.executeQuery();
				//7、遍历查询结果集
				while(resultSet.next()){
					User user 
					System.out.println(resultSet.getString("id")+"  "+resultSet.getString("username"));
				}
			} catch (Exception e) {
				e.printStackTrace();
			}finally{
				//8、释放资源
				if(resultSet!=null){
					try {
						resultSet.close();
					} catch (SQLException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
				if(preparedStatement!=null){
					try {
						preparedStatement.close();
					} catch (SQLException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
				if(connection!=null){
					try {
						connection.close();
					} catch (SQLException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}

			}

		}
```
#### - 问题总结
1. 在创建连接时，存在硬编码配置文件（全局配置文件）
2. 在执行statement时存在硬编码 配置文件（映射文件）
3. 频繁的开启和关闭数据库连接，会造成数据库性能下降。数据库连接池（全局配置文件
### 映射文件配置
在config目录下，创建User.xml（这种命名规范是由ibatis遗留下来）
- 配置 user.xml
```
<!-- namespace：命名空间，对statement的信息进行分类管理 -->
<!-- 注意：在mapper代理时，它具有特殊及重要的作用 -->
<mapper namespace="test">
     <!--根据用户ID查询用户信息--> 
     select：表示一个MappedStatement对象 
     id：statement的唯一标示
     #{}：表示一个占位符？
     #{id}：里面的id表示输入参数的参数名称，如果该参数是简单类型，那么#{}里面的参数名称可以任意
     parameterType：输入参数的java类型
     resultType：输出结果的所映射的java类型（单条结果所对应的java类型）
    <select id="findUsersById" parameterType="int" resultType="com.idsbg.mybatis.po.User" >
       SELECT * FROM USER WHERE id=#{id}
    </select>

    <!-- 根据用户名称模糊查询用户列表 -->
    ${}：表示一个sql的连接符 
    ${value}：里面的value表示输入参数的参数名称，如果该参数是简单类型，那么${}里面的参数名称必须是value 
    ${}这种写法存在sql注入的风险，所以要慎用！！但是在一些场景下，必须使用${}，比如排序时，动态传入排序的列名，${}会原样输出，不加解释 
    <select id="findUsersByName" parameterType="String" resultType="com.idsbg.mybatis.po.User">
        SELECT * FROM  USER  WHERE username LIKE '%${value}%'
    </select>

    <!-- 添加用户 -->
    selectKey：查询主键，在标签内需要输入查询主键的sql
    order：指定查询主键的sql和insert语句的执行顺序，相当于insert语句来说 
    LAST_INSERT_ID：该函数是mysql的函数，获取自增主键的ID，它必须配合insert语句一起使用 
    <insert id="insertUser" parameterType="com.idsbg.mybatis.po.User" >
        <selectKey keyProperty="id" resultType="int" order="AFTER">
            SELECT  LAST_INSERT_ID()
        </selectKey>
        INSERT INTO USER
        (username,birthday,sex,address)
        VALUES (#{username},#{birthday},#{sex},#{address})
    </insert>
    <delete id="deleteById" parameterType="int" >
            DELETE  FROM USER WHERE id=#{id}
    </delete>

    <!-- 自增主键之UUID -->
    <insert id="insertUser2" parameterType="com.idsbg.mybatis.po.User">
        <selectKey keyProperty="id" resultType="string" order="BEFORE">
            SELECT UUID()
        </selectKey>
        INSERT INTO USER
        (id,username,birthday,sex,address)
        VALUES (#{id},#{username},#{birthday},#{sex},#{address})
    </insert>

    <!-- 自增主键之UUID -->
    <insert id="insertUser3" parameterType="com.idsbg.mybatis.po.User">
        <selectKey keyProperty="id" resultType="int" order="BEFORE">
            SELECT seq.nextval FROM val
        </selectKey>
        INSERT INTO USER
        (id,username,birthday,sex,address)
        VALUES (#{id},#{username},#{birthday},#{sex},#{address})
    </insert>
</mapper>
```
- SqlMapConfig.xml的配置
```
 <?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 加载java的配置文件或者声明属性信息 -->
    <properties resource="db.properties">
        <property name="db.username" value="123" />
    </properties>
    <!-- 自定义别名 -->
    <typeAliases>
        <!-- 单个别名定义 -->
        <typeAlias type="com.idsbg.mybatis.po.User" alias="user"/>

        <!-- 批量别名定义（推荐） -->
        <!-- package：指定包名称来为该包下的po类声明别名，默认的别名就是类名（首字母大小写都可） -->
       <!-- <package name="com.idsbg.mybatis.po" />-->
    </typeAliases>
    <!-- 配置mybatis的环境信息，与spring整合，该信息由spring来管理 -->
    <environments default="development">
        <environment id="development">
            <!-- 配置JDBC事务控制，由mybatis进行管理 -->
            <transactionManager type="JDBC"></transactionManager>
            <!-- 配置数据源，采用mybatis连接池 -->
            <dataSource type="POOLED">
                <property name="driver" value="${db.driver}" />
                <property name="url" value="${db.url}" />
                <property name="username" value="${db.username}" />
                <property name="password" value="${db.password}" />
            </dataSource>
        </environment>
    </environments>
    <!-- 加载映射文件 -->
    <mappers>
       <!-- <mapper resource="User.xml" />-->
        <mapper resource="com/idsbg/mybatis/mapper/UserMapper.xml"/>
        <!-- 批量加载映射文件 -->
       <!-- <package name="com.idsbg.mybatis.mapper" />-->
    </mappers>

</configuration>
  ```
### 总结
- #{}和${}
- #{}表示占位符?，#{}接收简单类型的参数时，里面的名称可以任意
- ${}表示拼接符，${}接收简单类型的参数时，里面的名称必须是value
- ${}里面的值会原样输出，不加解析（如果该参数值是字符串，有不会添加引号）
- ${}存在sql注入的风险，但是有些场景下必须使用，比如排序后面会动态传入排序的列名
- parameterType和resultType:parameterType指定输入参数的java类型，parameterType只有一个，也就是说入参只有一个。resultType指定输出结果的java类型（是单条记录的java类型）
- selectOne和selectList:selectOne查询单个对象;selectList查询集合对象
### Mapper代理的开发方式(即开发mapper接口（相当于dao接口）)
Mapper代理使用的是jdk的代理策略。
- Mapper代理的开发规范
1. mapper接口的全限定名要和mapper映射文件的namespace值一致。
2. mapper接口的方法名称要和mapper映射文件的statement的id一致。
3. mapper接口的方法参数类型要和mapper映射文件的statement的parameterType的值一致，而且它的参数是一个。
4. mapper接口的方法返回值类型要和mapper映射文件的statement的resultType的值一致。
- mapper映射文件
在config下创建mapper目录然后创建UserMapper.xml（这是mybatis的命名规范，当然，也不是必须是这个名称）
sqlSession内部的数据区域本身就是一级缓存，是通过map来存储的。

- UserMapper.xml 配置文件：
 ```
<!--对 mapper接口相对应的文件进行配置-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.idsbg.mybatis.mapper.UserMapper">
    <!-- 根据用户ID查询用户信息 -->
    <select id="findById" parameterType="int" resultType="user">
        SELECT
        * FROM USER WHERE id =#{id}
    </select>

    <!-- 添加用户 -->
    <insert id="insertUser" parameterType="user">
        <selectKey keyProperty="id" resultType="int" order="AFTER">
            SELECT
            LAST_INSERT_ID()
        </selectKey>

        INSERT INTO USER
        (username,birthday,sex,address)
        VALUES(#{username},#{birthday},#{sex},#{address})
    </insert>

    <!-- 定义sql片段 -->
    <!-- sql片段内，可以定义sql语句中任何部分 -->
    <!-- sql片段内，最好不用将where和select关键字声明在内 -->
    <sql id="whereClause">
        <!-- if标签：可以对输入的参数进行判断 -->
        <!-- test:指定判断表达式 -->
        <if test="user != null">
            <if test="user.username != null and user.username != ''">
                AND username LIKE '%${user.username}%'
            </if>
            <if test="user.sex != null and user.sex != ''">
                AND sex = #{user.sex}
            </if>
        </if>

        <if test="idList != null">
            <!-- AND id IN (#{id},#{id},#{id}) -->

            <!-- collection：表示pojo中集合属性的属性名称 -->
            <!-- item:为遍历出的结果声明一个变量名称 -->
            <!-- open：遍历开始时，需要拼接的字符串 -->
            <!-- close:遍历结束时，需要拼接的字符串 -->
            <!-- separator：遍历中间需要拼接的连接符 -->
            AND id IN
            <foreach collection="idList" item="id" open="(" close=")"
                     separator=",">
                #{id}
            </foreach>
        </if>
    </sql>

    <!-- 综合查询，查询用户列表 -->
    <select id="findUserList" parameterType="com.idsbg.mybatis.po.UserQueryVO"
            resultType="user">
        SELECT * FROM user
        <!-- where标签：默认去掉后面第一个AND，如果没有参数，则把自己干掉 -->
        <where>
            <!-- 引入sql片段 -->
            <include refid="whereClause" />
        </where>
    </select>

    <!-- 综合查询用户总数 -->
    <select id="findUserCount" parameterType="com.idsbg.mybatis.po.UserQueryVO"
            resultType="int">
        SELECT count(*) FROM user
        <!-- where标签：默认去掉后面第一个AND，如果没有参数，则把自己干掉 -->
        <where>
            <!-- 引入sql片段 -->
            <include refid="whereClause" />
        </where>
    </select>
    <!-- resultMap入门 -->
    <!-- id标签：专门为查询结果中唯一列映射 -->
    <!-- result标签：映射查询结果中的普通列 -->
    <resultMap type="user" id="UserRstMap">
        <id column="id_" property="id" />
        <result column="username_" property="username" />
        <result column="sex_" property="sex" />
    </resultMap>

    <select id="findUserRstMap" parameterType="int" resultMap="UserRstMap">
        Select id id_,username username_,sex sex_ from user where id = #{id}
    </select>

</mapper>
  ```
- 加载映射文件
  ```
<!-- 加载映射文件 -->
    <mappers>
       <mapper resource="com/idsbg/mybatis/mapper/UserMapper.xml"/>
        <!-- 批量加载映射文件 -->
       <!-- <package name="com.idsbg.mybatis.mapper" />-->
    </mappers>
  ```

### 全局配置文件
```
SqlMapConfig.xml的配置内容和顺序如下（顺序不能乱）：
Properties（属性）
Settings（全局参数设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境信息集合）
	environment（单个环境信息）
		transactionManager（事物）
		dataSource（数据源）
mappers（映射器）
```

- 加载的顺序
1. 先加载properties中property标签声明的属性
2. 再加载properties标签引入的java配置文件中的属性
3. parameterType的值会和properties的属性值发生冲突。
- typeAliases
对po类进行别名的定义
```
 <typeAliases>
        <!-- 单个别名定义 -->
        <typeAlias type="com.idsbg.mybatis.po.User" alias="user"/>

        <!-- 批量别名定义（推荐） -->
        <!-- package：指定包名称来为该包下的po类声明别名，默认的别名就是类名（首字母大小写都可） -->
       <!-- <package name="com.idsbg.mybatis.po" />-->
    </typeAliases>
```
- mybatis支持的别名
```
别名	映射的类型
_byte 	byte 
_long 	long 
_short 	short 
_int 	int 
_integer 	int 
_double 	double 
_float 	float 
_boolean 	boolean 
string 	String 
byte 	Byte 
long 	Long 
short 	Short 
int 	Integer 
integer 	Integer 
double 	Double 
float 	Float 
boolean 	Boolean 
date 	Date 
decimal 	BigDecimal 
bigdecimal 	BigDecimal 
```
- 输出映射(resultType)
1. 使用resultType进行结果映射时，需要查询出的列名和映射的对象的属性名一致，才能映射成功。
2. 如果查询的列名和对象的属性名全部不一致，那么映射的对象为空。
3. 如果查询的列名和对象的属性名有一个一致，那么映射的对象不为空，但是只有映射正确那一个属性才有值。
4. 如果查询的sql的列名有别名，那么这个别名就是和属性映射的列名。
- resultMap使用要求
>使用resultMap进行结果映射时，不需要查询的列名和映射的属性名必须一致。但是需要声明一个resultMap，来对列名和属性名进行映射。

### 动态sql
在mybatis中，它提供了一些动态sql标签，可以让程序员更快的进行mybatis的开发，这些动态sql可以通过sql的可重用性。常用的动态sql标签：if标签、where标签、sql片段、foreach标签
- If标签/where标签映射文件(具体内容见 UserMapper.xml)
### mybatis与hibernate的区别及各自应用场景
- Mybatis技术特点：
1. 通过直接编写SQL语句，可以直接对SQL进行性能的优化；
2. 学习门槛低，学习成本低。只要有SQL基础，就可以学习mybatis，而且很容易上手；
3. 由于直接编写SQL语句，所以灵活多变，代码维护性更好。
4. 不能支持数据库无关性，即数据库发生变更，要写多套代码进行支持，移植性不好。
- Hibernate技术特点：
1. 标准的orm框架，程序员不需要编写SQL语句。
2. 具有良好的数据库无关性，即数据库发生变化的话，代码无需再次编写。
3. 学习门槛高，需要对数据关系模型有良好的基础，而且在设置OR映射的时候，需要考虑好性能和对象模型的权衡。
4. 程序员不能自主的去进行SQL性能优化。
- Mybatis应用场景：需求多变的互联网项目，例如电商项目。
- Hibernate应用场景：需求明确、业务固定的项目，例如OA项目、ERP项目等。



