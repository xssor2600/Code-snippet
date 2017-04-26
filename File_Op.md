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
<br>
2.0 **jersey上传文件**<br>
在使用restful规范的api开发过程中，会使用jersey来进行请求路由控制。也总会遇到文件上传的需要，下面记录jersey上传文件的一些简单代码块。<br>
(1)引入jersey文件上传需要的库包
```xml
	<dependency>
		    <groupId>org.glassfish.jersey.media</groupId>
		    <artifactId>jersey-media-multipart</artifactId>
		    <version>2.23.1</version>
		</dependency>
		
		<!-- https://mvnrepository.com/artifact/org.jvnet.mimepull/mimepull -->
        <dependency>
            <groupId>org.jvnet.mimepull</groupId>
            <artifactId>mimepull</artifactId>
            <version>1.9.4</version>
        </dependency>
	<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>1.3.1</version>
		</dependency>
```
(2)表单提交文件页面<br>
注意添加表单提交类型:`enctype="multipart/form-data"`,重点的还有页面的每个标签的name字段的字符串值，在后台程序中就是根据这个name来获取form表单中的各项数据。
```javascript
## fileUpload.jsp
	<form action="${pageContext.request.contextPath}/contentcreate/users/uploadphoto" method="post" enctype="multipart/form-data">
		 文件 :<input type="file" name="PICTURESTREAM"/><br />
		 <input type = "hidden" name="USERID" value="##">
		<input type="submit" value="上传" />
	</form>

```
(3)jersey的controller层:<br>
用于控制请求uri的方法映射。在上述表单中的action字段中就是映射到后台处理文件上传的方法中。<br>
```java
	@POST
	@Path("/uploadphoto")
    @Consumes(MediaType.MULTIPART_FORM_DATA)  
    @Produces(MediaType.APPLICATION_JSON + ";charset=utf-8")
	public Map<String,Object> uploadAvatar(FormDataMultiPart formDataMultiPart, @Context HttpServletRequest request){
		Map<String,Object> retMap = userService.uploadPicture(formDataMultiPart,request);
		return retMap;
	}
```
(4)实际文件业务处理service<br>
在实际的service层对文件的保存位置，文件名称，文件时效等等进行详细的处理。这里要注意的是，文件目录取的是本地或者服务器中的绝对路径，返回的图片url则是部署在服务器上的项目中的路径<br>
```java
##在服务器创建目录路径之后，根据图片字节流，通过FileUtils工具类将图片字节流写入到文件夹中，然后返回可以直接访问的该图片的url地址
public Map<String, Object> uploadPicture(FormDataMultiPart formDataMultiPart, HttpServletRequest request) {
		String retMess = "头像图片上传失败";
		Map<String,Object> retMap = new HashMap<String,Object>();
		//得到提交内容的图片文件二进制流，field值与页面的name值对应
		FormDataBodyPart pictureBP = formDataMultiPart.getField("PICTURESTREAM");
		FormDataBodyPart userIdBP = formDataMultiPart.getField("USERID");
		
		if(null == pictureBP || null == userIdBP) {
			retMap.put("RESULTCODE", ErrorContants.REQUESTDATANULL);
			retMap.put("RESULTMESS", retMess);
			retMap.put("PICTUREADRRE", "");	
			return retMap;
		}
		//在每个页面的multipart/form-data提交类型中，每个表单提交字段都是用FormDataBodyPart表示
    //这里取出流至关重要
		InputStream pictureInputStream = pictureBP.getValueAs(InputStream.class);
		String userId = userIdBP.getValueAs(String.class);
		
		if(null == userId || null == pictureInputStream) {
			retMap.put("RESULTCODE", ErrorContants.PARAMNULL);
			retMap.put("RESULTMESS", retMess);
			retMap.put("PICTUREADRRE", "");	
			return retMap;
		}
	 //用户身份校验
		boolean loginFlag = loginVerification.verification(userId);
		if(!loginFlag) {
			retMap.put("RESULTCODE", ErrorContants.NOTLOGIN);
			retMap.put("RESULTMESS", retMess);
			retMap.put("PICTUREADRRE", "");	
			return retMap;
		}
		
		StringBuffer avatarPath = new StringBuffer();
		//服务器当前绝对路径
		StringBuffer localPath = new StringBuffer();
		StringBuffer picturePath = new StringBuffer();
		String fileName = new UUIDGenerate().genericIdentity() + ".png";
		String root = request.getContextPath();
		String realPath = request.getServletContext().getRealPath("/");
		log.info("当前绝对路径：>>" + realPath);
		String datePath1 = DateUtil.getDateTimeStr().replace("-", File.separator);
		String datepath2 = datePath1.substring(0,datePath1.indexOf(" "));

		
    String domain = ConfigService.getValue("DONAME", "doman");
//		String picturePath = realPath+ File.separator +"avatars"+ File.separator + datepath2 + File.separator;
		//图片的相对路径
		picturePath.append(File.separator).append("avatars").append(File.separator).append(datepath2).append(File.separator);
		localPath.append(realPath).append(picturePath.toString());
		//在服务器上创建头像图片目录
		File file = new File(localPath.toString());
		if(!file.exists() && file.isDirectory()) {
			file.mkdir();
		}
		
		try {
			FileUtils.copyInputStreamToFile(pictureInputStream, new File(file,fileName));
		} catch (IOException e) {
			e.printStackTrace();
			log.error("保存文件到路径:>>" + localPath.toString() +",失败：",e);
			retMap.put("RESULTCODE", ErrorContants.RUNFALSE);
			retMap.put("RESULTMESS", "头像图片上传失败");
			retMap.put("PICTUREADRRE", "");
			return retMap;
			
		}  finally {
			if(pictureInputStream != null)
				try {
					pictureInputStream.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
		}
		avatarPath.append(domain).append(root).append(picturePath.toString()).append(fileName);
		retMap.put("RESULTCODE", ErrorContants.SUCCESS);
		retMap.put("RESULTMESS", "头像图片上传成功");
		retMap.put("PICTUREADRRE", avatarPath.toString());
		return retMap;
	}
```
