## UUIDGenerate生成器<br>
若是程序中需要使用UUID来对对象或者记录进行标识，那么UUID生成器也就有了作用。<br>
当然了，要使用这个生成器，那么需要在pom.xml中引入依赖包:<br>
```xml
		<dependency>
			<groupId>com.fasterxml.uuid</groupId>
			<artifactId>java-uuid-generator</artifactId>
			<version>3.1.4</version>
		</dependency>

```
<br>
* 生成UUID唯一标识字符串<br>
```java
	public String genericIdentity() {
		return Generators.timeBasedGenerator().generate().toString().replace("-", "");
	}
```