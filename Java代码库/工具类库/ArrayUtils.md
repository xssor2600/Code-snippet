## Array数组工具类库<br>
在我们开发中，数组这容器是少不了的。所以，对数组的一些经常的使用，操作都可以封装到数组工具类中，作为共用代码块使用。若是在后面发现更好的方法，则可以实时替换，因为在github上面真的太方便了。<br>

* 判断数组中是否包含某个值:<br>
```java
	public static<T> boolean  arrayContains(T[] array,T content){
		boolean flag = false;
		for(int i=0;i<array.length;i++){
			if(content.equals(array[i])||content==array[i]){
				flag=true;
				break;
			}
		}
		return flag;
	}
```
* 判断当前一个小的数组是否包含在另外一个大的数组中:<br> 
 目的也就是说看看小的数组是否是大数组的一个子集。<br>
```java
	public static<T> boolean arrayContainsArray(T[] origArray,T[] targArray){
		boolean flag = true;
			for(int i=0;i<targArray.length;i++){
				if(!Arrays.asList(origArray).contains(targArray[i])){
					flag = false;
					return flag;
				}
		}
		return flag;
	}
```
* 将一个List集合转换为特定符号分隔的字符串:<br>
```java
	public static<T> String listToString(List<T> list,String sign){
        if (list==null || list.size()==0) {
            return null;
        }
        StringBuilder result=new StringBuilder();
        boolean flag=false;
        for (T string : list) {
            if (flag) {
                result.append(sign);
            }else {
                flag=true;
            }
            result.append(string);
        }
        return result.toString();
    }
```
* 除去一个小数组和大数组中的交集，返回两个数组的并集：<br> 
  也就是先将两个数组中的相同的元素去除，最后返回两者去除交集元素后的并集。<br>
```java
	@SuppressWarnings("unchecked")
	public static <T> T[] filterRepeatElements(final T[] mixArray,final T[] maxArray){
	    if(mixArray == null || mixArray.length == 0) {
	        return maxArray;
	    }
	    final int len = Array.getLength(maxArray)-Array.getLength(mixArray);
	    final Class<?> type = maxArray.getClass().getComponentType();
	    final Set<T> temSet1 = new HashSet<T>();
	    final Set<T> temSet2 = new HashSet<T>();
	    Collections.addAll(temSet1, mixArray);
	    Collections.addAll(temSet2, maxArray);
	    temSet2.removeAll(temSet1);
	    T[] retArray = (T[]) Array.newInstance(type, len);
	    int i = 0;
	    for(Iterator<T> it = temSet2.iterator();it.hasNext();){
	        retArray[i] = it.next();
	        i++;
	    }
	    return retArray;
	}

```
* 查找处整型数组中最小的元素:<br>
```java
	public static int minInt(Integer[] target) {
		// 返回数组最小值
		if(target==null || target.length==0){
			return -1;
		}
		int x;
		Integer temp[] = new Integer[target.length];
		System.arraycopy(target, 0, temp, 0, target.length);
		x = temp[0];
		for (int i = 1; i < temp.length; i++) {
			if (temp[i] < x) {
				x = temp[i];
			}
		}
		return x;
	}

```