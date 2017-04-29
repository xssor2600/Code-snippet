## File Upload<br>
记录在开发中使用到的文件上传代码段，包括不同框架，不同方式的上传方式。所谓上传，就是将客户端本地的文件的字节流传输到服务器，服务器根据自定义规则生成文件名，将这些接收到的字节流，存储在什么地方，方便存取而已。<br>

* **jersey上传文件**<br>
在使用restful规范的api开发过程中，会使用jersey来进行请求路由控制。也总会遇到文件上传的需要，下面记录jersey上传文件的一些简单代码块。参考：[jersey 文件上传-使用两种不同的方式](http://blog.csdn.net/wk313753744/article/details/46235895)<br>
(1) 引入jersey文件上传需要的库包:<br>
```
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
(2)表单提交文件页面:<br>
注意添加表单提交类型:`enctype="multipart/form-data"`,重点的还有页面的每个标签的name字段的字符串值，在后台程序中就是根据这个name来获取form表单中的各项数据。<br>
```
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