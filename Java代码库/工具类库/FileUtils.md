## File操作工具类库<br>
文件操作工具类。<br>
* 文件拷贝:<br>
	* 基本的底层文件字节流拷贝FileStream:<br>
	```java
    private static void copyFileUsingFileStream(File source,File dest) throws IOException{
        try(InputStream input = new FileInputStream(source);OutputStream output = new FileOutputStream(dest)){
            //保存从输入流中读取的字节数组
            byte[] bytes = new byte[input.available()];
            int readByte = 0;
            while((readByte = input.read(bytes)) != -1){
                output.write(bytes,0, readByte);
            }
        }catch(Exception e) {
            e.printStackTrace();
        }

    }
	```
	* 使用NIO的文件渠道FileChannel:<br> 
	```java
	private static void copyFileUsingFileChannels(File source,File dest) throws Exception{
        try(FileChannel inputChannel = new FileInputStream(source).getChannel();
            FileChannel outputChannel = new FileOutputStream(dest).getChannel()){
            //inputChannel.transferTo(0,inputChannel.size(),outputChannel);
            outputChannel.transferFrom(inputChannel,0,inputChannel.size());
        }catch (Exception e){
                e.printStackTrace();
        }
    }
	```
	* 使用Apache-common-io工具类:<br>
需要添加common-io对应的maven依赖。
	```java
    private static void copyFileUsingApacheCommonIO(File source,File dest) throws Exception{
        //底层也是由FileChannel实现的
        FileUtils.copyFile(source,dest);
    }
	```
	* jdk7自带的Files文件操作类：<br> 
  也就是先将两个数组中的相同的元素去除，最后返回两者去除交集元素后的并集。<br>
	```java
    private static void copyFileUsingJava7Files(File source,File dest) throws Exception{
        Files.copy(source.toPath(),dest.toPath());
    }

	```