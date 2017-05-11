## File Operation<br>
记录一些开发中或者平时需要使用的文件File的操作任务。在开发中，文件是基础，我们无时无刻都在对文件进行写入，保存即写入磁盘等操作。文件也是我们记录配置文件的载体，所以对文件的操作是开发的基础。拿不到文件句柄，就读取不了文件数据，也就不能持久化的将处理后的数据写入磁盘中。<br>

当然了，若是在maven工程中操作文件，那么也已经有了不少现成的文件操作工具库。例如apache的`commons-io`等。那么，如果使用轮子有利有弊，优势则是轮子是经过优化测试后的工具，使用放心；劣势则是封装太好，不能更深的进行业务自定义开发。那么可以根据实际情况来操作了。<br>

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
2.0 删除某个指定文件夹下面的指定文件名的文件。<br>
若是在maven项目中，引用了Apache的commons-io工具库，那么可以直接使用**FileUtils**进行操作。如调用方法`FileUtils.deleteQuietly(File file)`进行完成这个任务。<br>
若是我们使用原生JDK进行操作，也是个不错的选择:<br>
```java
	/**
	 * delSpecificDirectoryFile : 删除指定目录下，指定文件名的文件。仅仅只在该层目录下，不递归查询子文件夹。
	 * 
	 * @param diretoryPath
	 *            指定文件夹
	 * @param fileName
	 *            指定要删除文件名称
	 * @return
	 */
	public static boolean delSpecificDirectoryFile(String diretoryPath, String fileName) {
		if (diretoryPath == null) {
			log.info("删除文件所在的目录不能为null");
			return false;
		}
		File dir = new File(diretoryPath);
		log.info("要删除文件目录为:" + diretoryPath + "下的文件，名字为:" + fileName);
		if (dir.isDirectory() && dir.exists()) {
			File[] files = dir.listFiles();
			for (File file : files) {
				if (file.isFile() && file.getName().equals(fileName)) {
					File tempFile = new File(diretoryPath + fileName);
					if (tempFile.isFile()) {
						try {
							return tempFile.delete();
						} catch (Exception ignored) {
							return false;
						}
					}
				}
			}
		}
		return false;
	}
```

