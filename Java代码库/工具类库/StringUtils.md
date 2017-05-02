## StringUtils字符串工具类<br>
字符串在软件开发中的重要性就不在赘述啦，是灰常重要也时刻使用的数据类型或者说是对象类型。希望将我使用过的字符串工具方法记录下来，不断优化代码块，不断完善。<br>

* 根据指定符号分隔字符串<br>
```java
	public static String[] splitStr(String str,String sSplitSign) {
		String[] oSplitSignArray = new String[]{};
		if (str!=null && sSplitSign!=null &&StringUtils.isNotEmpty(str) && StringUtils.isNotEmpty(sSplitSign)) {
			oSplitSignArray = str.split(sSplitSign);
		} else {
			log.error(">>>数据有误，错误数据str为：" + str + "\tsSplitSign为：" + sSplitSign);
		}
		return oSplitSignArray;
	}

```
* 查询字符串中指定符号指定次数出现的位置<br>
```java
	public static int getCharacterPosition(String string,String sign,int n){
	     
	    Matcher slashMatcher = Pattern.compile(sign).matcher(string);
	     int mIdx = 0;
	     while(slashMatcher.find()) {
	        mIdx++;
	        //当"/"符号第n次出现的位置
	       if(mIdx == n){
	           break;
	        }
	     }
	     return slashMatcher.start();
	  }

```
* 校验字符串是否为空且是否是"null"字符串<br>
```java
	public static boolean validStr(String str){
		return (StringUtils.isNotEmpty(str) && !"null".equals(str)) ? true : false;
	}
```