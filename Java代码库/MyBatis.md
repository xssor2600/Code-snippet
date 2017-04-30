## MyBatis Relevant<br>
mybatis框架相关内容代码块整合。<br>

* MyBatis出入多个参数<br>
当在项目中dao层，使用mybatis来进行数据获取与管理，那么当dao接口方法中的入参为多个参数，那么该 如何处理?下面就列出几种方式的代码段。<br>
(1) 使用占位符来表示多个入参顺序:<br>
在配置文件中使用`#{n}`表示接收第n个入参。<br>
```java
## Dao接口中声明的多参数方法
public User selectUser(String name, String department);

## 对应mybatis的Mapper.xml映射sql文件
<select id="selectUser" resultMap="userMap">
  select * from t_user where vc_user_name = #{0} and vc_department=#{1}
</select>

```
可以看到mapper.xml中的sql参数使用占位符**#{}**来接收参数，即#{0}表示第一个参数，#{1}表示方法的第二个参数，以此类推。<br>
<br>
(2) 采用Map类型传递多个参数<br>
因为Map数据类型是KEY-VALUE键值对结构的，那么就可以将KEY作为参数，传递到xml中配置的sql语句中。<br>
```java
## Dao层接口方法，入参用传入Map类型
public User selectUser(Map paramMap);

## 对应UserMapper.xml中
<select id="selectUser" resultMap="userMap">
	select * from t_user where vc_user_name=#{userName,jdbcType=VARCHAR} 
		and vc_department=#{department,jdbcType=VARCHAR}
</select>

## service层中构造map参数调用
Map paramMap = new HashMap();
paramMap.put("userName","xxx");
paramMap.put("department","xxx");
User user = selectUser(paramMap);
```
虽然通过这种传入Map类型参数也能实现多参数查询，但是该种方法不够友好，要非常清楚map中key值是什么，才能在sql中设置使用正确，若是map中的key参数修改了，而忘记同步xml中sql的参数名字，那么就会传参失败。<br>
<br>
(3)通过注解来传递参数<br<
通过mybatis内部提供的注解@param来实现传入多个参数，这种方式比较友好，推荐。<br>
```java
## Dao接口层多个参数使用注解设置
public User selectUser(@param("userName") String userName, @param("department") String department);

## 对应UserMapper.xml中使用参数
<select id="selectUser" resultMap="userMap">
		select * from t_user where vc_user_name=#{userName,jdbcType=VARCHAR} 
		and vc_department=#{department,jdbcType=VARCHAR}
</select>

```
这样，通过在dao层接口方法参数中使用@param来绑定参数名称的话，就算该方法入参的名字修改了，也是不影响mapper.xml中sql的参数名称。最大程度的将dao接口方法与mapper.xml中的sql接收的参数进行解绑。<br>
<br>
(4)List封装sql的in语法<br>
当我们的sql中需要使用in语法查询在某个范围集合内的数据，可以将数据保存到List中，再将List作为参数传入Mapper.xml中。<br>
若是我们要查询多个用户id的数据，那么可以将id保存在List中。<br>
```java
## Dao层接口声明方法
public List<User> listUser(List<String> ids);

## UserMapper.xml中
<select id="listUser" retsultType="xx.xx.User">
	select vc_id id, vc_user_name userName,.... from t_user where vc_id in
	<!--注意这里的collection参数与dao入参变量名相同 -->
    <foreach item="item" index="index" collection="ids" open="(" separator="," close=")">
		#{item}
	</foreach>
</select>

## foreach就可以构成: select ... from t_user where vc_id in ('xx','xx',...)
```
(5)若是List参数与普通参数复合，那么可以使用Map<String,Object>类型作为参数<br>
```java
## service
List<String> ids = new ArrayList<String>();
ids.add("1");
ids.add("2");

Map<String,Object> paramMap = new HashMap<Stirng,Object>();
map.put("registerDate",DateUtils.getDate());
map.put("ids",ids);

## dao
List<User> getUsers(Map<String,Object> paramMap);

## mapper.xml
<select id="getUsers" resultType="xxx.xxx.User">
 select vc_id id, vc_user_name userName,.... from t_user where 
	vc_register_date = #{registerDate,jdbcType=VARCHAR} and vc_id in
	<foreach item="item" index="index" collection="ids" open="(" separator="," close=")">
		#{item}
	</foreach> 
</select>

```
方式挺多，都可以按照实际的需求进行搭配使用。<br>