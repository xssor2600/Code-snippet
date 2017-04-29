## Java生态圈内容总集:<br>
java相关的所有其他内容，进行代码库罗列和总结整理。<br>

### maven<br>
maven的相关内容>_<...<br>
* maven编译设置包括web项目lib目录下jar包。<br>
由于现在java的包库依赖很多时候都是由maven管理，但是也有不少三方其他自定封装的jar包在maven库中并没有定义，就不能在项目的pom.xml中引入依赖。这时候我们在编译的时候就会缺少jar而编译失败。所以，我们的目的的要设置在maven的build命令过程中，会搜索web项目下的lib文件夹下所有的jar库包，以至于编译可以成功通过。当然，若是公司有自己的私库，那么可以直接将jar包上传，在公司内网中的项目可以直接引入依赖。下面的代码库就是用于实现这个目的。参考:[ 既使用maven编译，又使用lib下的Jar包](http://blog.csdn.net/catoop/article/details/48489365)<br>
```xml
## pom.xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
        <source>1.7</source>
        <target>1.7</target>
        <encoding>UTF-8</encoding>
        <compilerArgs> 
            <arg>-verbose</arg>
            <arg>-Xlint:unchecked</arg>
            <arg>-Xlint:deprecation</arg>
            <arg>-bootclasspath</arg>
            <arg>${env.JAVA_HOME}/jre/lib/rt.jar</arg>
            <arg>-extdirs</arg> 
            <arg>${project.basedir}/src/main/webapp/WEB-INF/lib</arg>
        </compilerArgs> 
    </configuration>
</plugin>


```
