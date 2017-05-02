## DateUtils日期工具类<br>
日期也是在软件开发中经常使用的数据项，在软件开发领域，对于日期的形式需求也多种多样。整理这个日期工具类，希望可以将大多数日期使用方法总结起来。<br>

* 得到 yyyy-MM-dd HH:mm:ss 格式当前时间<br>
将当前系统时间，转换成上述格式的字符串。<br>
```java
	public static String getDateTimeStr() {
		Date dNowTime = Calendar.getInstance().getTime();
		SimpleDateFormat sdformat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		return sdformat.format(dNowTime);
	}

```
* 得到 yyyyMMdd格式当前时间<br>
 得到上述格式时间字符串，也就是将时间仅仅细化到天。<br>
```java
		Date dNowTime = Calendar.getInstance().getTime();
		SimpleDateFormat sdformat = new SimpleDateFormat("yyyyMMdd");
		return sdformat.format(dNowTime);

```