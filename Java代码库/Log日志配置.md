## Log日志配置<br>
Log在开发和测试过程中都占据着重要作用。关于日志相关的代码段也整理起来，以便自己使用和学习。<br>

* Log日志文件夹初始化<br>
  通常我们都会在配置了日志相关的配置文件后，会统一在指定位置创建文件夹来保存日志信息。这里，可以在web.xml中添加监听器，监听web服务在启动时候，便会在项目根目录下，创建一个名为log的文件夹，用于存储项目中打印的日志信息。<br>
  (1) 添加日志maven依赖包<br>
```xml
		<!-- https://mvnrepository.com/artifact/log4j/log4j -->
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>1.2.17</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/commons-logging/commons-logging -->
		<dependency>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
			<version>1.0</version>
		</dependency>
```
  (2) 在web.xml中添加日志生成监听器<br>
```xml
	<!-- 日志文件根路径设置 -->
	<listener>
		<listener-class>com.xiansky.utils.LogBasePath</listener-class>
	</listener>
```
  (3) 创建监听器任务类LogBasePath
```java
public class LogBasePath implements ServletContextListener {

	public void contextDestroyed(ServletContextEvent servletContextEvent) {
		System.clearProperty("logBasePath");
		System.clearProperty("classesPath");
	}

	public void contextInitialized(ServletContextEvent servletContextEvent) {		
		System.clearProperty("logBasePath");
		System.clearProperty("classesPath");
		String sTempPath = System.getProperty("user.dir");
		String sTemppath2 = "";
		//含有才处理
		if (StringUtils.contains(sTempPath, "bin")) {
			sTemppath2 = sTempPath.substring(0, sTempPath.indexOf("bin"));
		} else {
			sTemppath2 = sTempPath;
		}
		String sProjectName = servletContextEvent.getServletContext().getContextPath();
		//设置classes路径环境
		StringBuffer sClassesPath = new StringBuffer(50);
		sClassesPath.append(sTemppath2.replace("\\", "/")).append("webapps").append(sProjectName)
			.append("/WEB-INF/classes/");
		System.setProperty("classesPath", sClassesPath.toString());
		
		//设置log4j路径		
		StringBuffer sLogPath = new StringBuffer(50);
		sLogPath.append(sTemppath2.replace("\\", "/")).append("webapps").append(sProjectName).append("/log/");
		System.setProperty("logBasePath", sLogPath.toString());
		sLogPath.delete(0, sLogPath.length());
		
		//加载log4j
		System.out.println(">>>ClassesPath: " + sClassesPath.toString());
		StringBuffer sLog4jPath = new StringBuffer();
		sLog4jPath.append(sClassesPath.toString()).append("log4j.xml");
		DOMConfigurator.configure(sLog4jPath.toString());
		sLog4jPath.delete(0, sLog4jPath.length());
		sClassesPath.delete(0, sClassesPath.length());

	}

}

```
这样，在我们启动部署在服务器上的web应用时候，就会在项目根目录下创建日志Log文件夹，用于存放项目所有的打印输出，记录的日志信息，便于我们查看和排错。<br>

 (4) 配置日志log4j.xml文件<br>
在上述配置了日志文件夹后，也会在当前系统变量中添加名为`logBasePath`的变量，这样，我们在配置文件就可以直接引用这个变量，这个变量中存放就是我们项目根目录下的log文件夹的绝对路径。这样一来，就可以分别定义debug,info,error,warn等级别的日志输出磁盘位置。<br>
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE log4j:configuration SYSTEM "http://toolkit.alibaba-inc.com/dtd/log4j/log4j.dtd">
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
	<!-- ===================================================================== -->
	<!-- 以下是appender的定义，定义日志输出的目的地、输出方式及过滤级别 -->
	<!-- ===================================================================== -->
	<!-- 将信息输到控制台 -->	
	<appender name="stdout" class="org.apache.log4j.ConsoleAppender">
		<layout class="org.apache.log4j.PatternLayout">
			<param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss:SSS} %p %t [%L] %c - %m%n" />
		</layout>
	</appender>
	
	<!-- 未定义拦截信息 -->
	<appender name="other-log" class="org.apache.log4j.DailyRollingFileAppender">
		<param name="file" value="${logBasePath}/other-log.log" />
		<param name="append" value="true" />
		<param name="encoding" value="UTF-8" />
		<layout class="org.apache.log4j.PatternLayout">
			<param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss:SSS} %p %t [%L] %c - %m%n" />
		</layout>
	</appender>

	<!-- 所有定义debug,info,warn,error信息日志 -->
	<appender name="all-log" class="org.apache.log4j.DailyRollingFileAppender">
		<param name="file" value="${logBasePath}/all.log" />
		<param name="append" value="true" />
		<param name="encoding" value="UTF-8" />
		<layout class="org.apache.log4j.PatternLayout">
			<param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss:SSS} %p %t [%L] %c - %m%n" />
		</layout>		
	</appender>

	<!-- 定义debug信息日志 -->
	<appender name="debug-log" class="org.apache.log4j.DailyRollingFileAppender">
		<param name="file" value="${logBasePath}/debug.log" />
		<param name="append" value="true" />
		<param name="encoding" value="UTF-8" />
		<layout class="org.apache.log4j.PatternLayout">
			<param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss:SSS} %p %t [%L] %c - %m%n" />
		</layout>		
		<filter class="org.apache.log4j.varia.LevelRangeFilter">
			<param name="LevelMax" value="DEBUG"></param>
			<param name="LevelMin" value="DEBUG"></param>
		</filter>
	</appender>
	<!-- 定义info信息日志 -->
	<appender name="info-log" class="org.apache.log4j.DailyRollingFileAppender">
		<param name="file" value="${logBasePath}/info.log" />
		<param name="append" value="true" />
		<param name="encoding" value="UTF-8" />
		<layout class="org.apache.log4j.PatternLayout">
			<param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss:SSS} %p %t [%L] %c - %m%n" />
		</layout>
		<filter class="org.apache.log4j.varia.LevelRangeFilter">
			<param name="LevelMax" value="INFO"></param>
			<param name="LevelMin" value="INFO"></param>
		</filter>
	</appender>
	<!-- 定义warn信息日志 -->
	<appender name="warn-log" class="org.apache.log4j.DailyRollingFileAppender">
		<param name="file" value="${logBasePath}/warn.log" />
		<param name="append" value="true" />
		<param name="encoding" value="UTF-8" />
		<layout class="org.apache.log4j.PatternLayout">
			<param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss:SSS} %p %t [%L] %c - %m%n" />
		</layout>
		<filter class="org.apache.log4j.varia.LevelRangeFilter">
			<param name="LevelMax" value="WARN"></param>
			<param name="LevelMin" value="WARN"></param>
		</filter>
	</appender>
	<!-- 定义error信息日志 -->
	<appender name="error-log" class="org.apache.log4j.DailyRollingFileAppender">
		<param name="file" value="${logBasePath}/error.log" />
		<param name="append" value="true" />
		<param name="encoding" value="UTF-8" />
		<layout class="org.apache.log4j.PatternLayout">
			<param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss:SSS} %p %t [%L] %c - %m%n" />
		</layout>
		<filter class="org.apache.log4j.varia.LevelRangeFilter">
			<param name="LevelMax" value="ERROR"></param>
			<param name="LevelMin" value="ERROR"></param>
		</filter>
	</appender>

	<!-- ===================================================================== -->
	<!-- 日志写出器：每一个logger可以有多个输出目的地和输出方式 -->
	<!-- ===================================================================== -->
	<logger name="com.cybbj" additivity="false">
		<level value="ALL"/>
		<appender-ref ref="stdout" />
		<appender-ref ref="all-log"/>
		<appender-ref ref="debug-log"/>
		<appender-ref ref="info-log"/>
		<appender-ref ref="warn-log"/>
		<appender-ref ref="error-log"/>
	</logger>

	<!-- ===================================================================== -->
	<!-- Root logger 所有logger的基类，没有定义的logger将会使用root logger -->
	<!-- ===================================================================== -->
	<root>
		<level value="info" />
		<appender-ref ref="stdout" />
		<appender-ref ref="other-log" />		
	</root>
</log4j:configuration>

```
