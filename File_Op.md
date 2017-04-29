## File Operation<br>
记录一些开发中或者平时需要使用的文件File的操作任务。在开发中，文件是基础，我们无时无刻都在对文件进行写入，保存即写入磁盘等操作。文件也是我们记录配置文件的载体，所以对文件的操作是开发的基础。拿不到文件句柄，就读取不了文件数据，也就不能持久化的将处理后的数据写入磁盘中。<br>

1.0 删除某个指定名称目录以及其子目录中的所有文件。(eg:在拷贝文件时候，总会将svn的配置目录`.svn`同时复制，想要删除所有文件下的.svn目录)<br>
```java
	@Test
	public void delSpecifiedDir() {
		delSpecifiedDir(new File("C:\\Users\\XianSky\\Desktop\\xiangbideWXBG-1"),".svn");
	}
  
 public void delSpecifiedDir(File file,String dirName) {
		if(file.isDirectory() && file.getName().equals(dirName)) {
			deleDir(file); //删除文件目录以及子目录
		} else if (file.isDirectory()) {
			File[] listf = file.listFiles();
			for(int i = 0; i < listf.length; i++) {
				delSpecifiedDir(listf[i],dirName); //递归查询
			}
		}
	}
  
 private boolean deleDir(File dir) {
		if(dir.isDirectory()) {
			String[] children = dir.list();
			//递归删除目录中所有子目录
			for(int i =0; i < children.length; i++) {
				boolean success = deleDir(new File(dir,children[i]));
				if(!success) {
					return false;
				}
			}
		}
		//此时目录为空，可以删除
		return dir.delete();
	}

```

