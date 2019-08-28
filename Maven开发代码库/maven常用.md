#### @@Maven常用配置

- 工程不进行打包上传私服

  ```java
      <properties>
          <maven.deploy.skip>true</maven.deploy.skip>
      </properties>
  ```

  打包maven库没有的包到本地maven库: 
  eg: 若是maven中心库没有需要的jar包(oracle),则可以通过下面的maven命令打包到本地maven库后应用即可。如安装Oracle驱动到本地库：  
  `mvn install:install-file -DgroupId=com.oracle "-DartifactId=ojdc14" "-Dversion=10.2.0.2.0" "-Dpackaging=jar" "-Dfile=D:\ojdbc14.jar"`

  maven项目中pom.xml指定项目编码格式:  

  ```xml
  <!-- 指明编译源代码时会用的字符编码,maven编译的时候默认使用GBK编码 -->
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  ```

- 

