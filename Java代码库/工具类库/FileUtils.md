## File操作工具类库<br>
文件操作工具类。<br>
* 文件拷贝:<br>
	* 基本的底层文件字节流拷贝FileStream:<br>
	```java
    private static void copyFileUsingFileStream(File source,File dest) throws IOException{
        try(InputStream input = new FileInputStream(source);OutputStream output = new FileOutputStream(dest)){
            //保存从输入流中读取的字节数组
            byte[] bytes = new byte[input.available()];
            int readByte = 0; //每次从输入流中读取存入数组的字节个数
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
* 使用字节流IO操作要注意问题：<br>
 从输入流中读取字节数据的时候，通常会会使用一下几个方法:<br>
```java
int read() throws IOExceptions; //返回读取字节
int read(byte[]) throws IOExceptions; //返回已读取字节数
int read(byte[],int off,int len) throws IOExceptions; 返回已读取字节数
```
常规使用方式，就是通过一个定长的字节数组byte[]，作为字节数据存放容器，从输入流读取数据，再将存放字节流的数组中数据写入
到其他输出或者进行其他处理。常规流程就如上述文件拷贝中`FileStream`的方式。<br>
```java
public static byte[] readStream(InputStream inStream) throws Exception{
   try(ByteArrayOutputStream outStream = new ByteArrayOutputStream()){
   byte[] data = new byte[1024]; //存放从输入流中读取到的字节数据
   int len = -1; //每次从输入流中读取写入字节数组的字节数量
   while((len = inStream.read(data)) != -1){
		outStream.write(data,0,len); //每次循环读取len个字节，写入len个字节
   }
   it(inStream != null)
   inStream.close();
   return outStream.toByteArray();
   }catch(IOException e){}
}
```
这种方式会存在隐患，就是会出现输入流中还没有读取到数据(eg:网络影响，磁盘，大量数据),就对改输入流进行数据提取就会阻塞，<br>
读取不到任何数据。<br>

更近一步的调整，就是在创建字节数组长度的时候，可以使用输入流的`available()`方法，这个方法的意思是返回此输入流下一个方法调用可以不受阻塞地从此输入流读取（或跳过）的估计字节数。因为有时候在网络应用中，流并不是一次性传输完的。<br>
```java
int count = in.available();  
byte[] b = new byte[count];  
inStream.read(b);  
```
可是在进行网络操作时往往出错，因为你**调用available()方法时，对发发送的数据可能还没有到达**，你得到的count是0。<br>
改动如下： 判断流中可读的字节数是否为0。<br>
```
   int count = 0;  
    while (count == 0) {  
        count = inStream.available();  
    }  
    byte[] b = new byte[count];  
    inStream.read(b); 
```
若是要确保控制一定能将输入流中所有字节数据读取出来，可以使用每次输入流读取字节数和输入流中总字节数来对比判断：<br>
```java
int count = 100;  
byte[] b = new byte[count];  
int readCount = 0; // 已经成功读取的字节的个数  
while (readCount < count) {  
    readCount += inStream.read(b, readCount, count - readCount);  
} 
```
这样每次都能读取100个字节。参考: [java InputStream读取数据问题](http://cuisuqiang.iteye.com/blog/1434416)