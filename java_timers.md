## Java定时器<br>
记录一些java开发中常用的定时器代码段，本着拷贝即用的想法，整理代码段并记录下来。<br>
<br>
### Java Quartz<br>
quartz也是java开发中常用的定时器框架，使用也简单，在配置文件中配置，自定义任务调度执行时间。<br>
使用环境是在结合spring框架的定时器调度使用。<br>
```java
## 1.0 在项目的pom.xml中添加对quartz的依赖
<!-- 定时器 -->
<dependency>
	<groupId>org.quartz-scheduler</groupId>
	<artifactId>quartz</artifactId>
	<version>1.8.6</version>
</dependency>

====================================================

## 2.0 定义任务调度器调用的执行句柄类和执行方法
## 当时间到的时候，就调用该类的execute方法进行业务执行
@Service("accessTokenService")
public class AccessTokenService {
	private Log log = LogFactory.getLog(AccessTokenService.class);
  ...
  
  /**
	 * execute : 定时器调用的执行方法。
	 */
	public void execute() {
		getAccessToken();
	}
  
  private void getAccessToken() {
  ...
  }

}

===================================================
## 3.0 在spring主配置文件applicationContext.xml中进行调度器的配置
## applicationContext.xml
	<!-- 应用程序定时器配置 -->
	<bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
		<property name="triggers">
			<list>
				<ref bean="accessTokenTrigger" />
			</list>
		</property>
		<property name="autoStartup" value="true" />
	</bean>

	<!-- 配置定时器：每2小时刷任务调度执行一次 通过CronExpression进行对调度时间的设置 -->
	<bean id="accessTokenTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean">
		<property name="jobDetail" ref="tokenJobDetail" />
		<property name="cronExpression" value="0 0 */2 * * ?" /><!-- 每隔2个小时触发一次 -->
	</bean>
	<bean id="tokenJobDetail"
		class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
    	<!-- 容器中引用任务调度器句柄 -->
		<property name="targetObject" ref="accessTokenService" />
      <!-- 定义任务调度器句柄中具体的调用的处理方法名称 -->
		<property name="targetMethod" value="execute" />
		<property name="concurrent" value="false" />
		<!-- 是否允许任务并发执行。当值为false时，表示必须等到前一个线程处理完毕后才再启一个新的线程 -->
	</bean>


```
