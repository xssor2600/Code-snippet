## Http 请求封装类<br>
由于在web开发中会经常需要在服务器端进行http请求的发送与处理，所以这里罗列了一些之前在开发中遇到和使用的封装好的http请求方法，针对get,post请求，返回类型等等。<br>
```java
## 发送http请求，返回字符串封装成json对象。

	/**
	 * httpRequestReturnByJson : 发送一个http请求，返回json对象。
	 * @param requestUrl
	 * @param requestMethod
	 * @param outputStr
	 * @return
	 */
	   public static JSONObject httpRequestReturnByJson(String requestUrl,String requestMethod, String outputStr) {  
	        JSONObject jsonObject = null;  
	        StringBuffer buffer = new StringBuffer();  
	        try {  
	            TrustManager[] tm = { new X509TrustManager() {
					public void checkClientTrusted(X509Certificate[] chain,
							String authType) throws CertificateException {
					}
					public void checkServerTrusted(X509Certificate[] chain,
							String authType) throws CertificateException {
					}
					public X509Certificate[] getAcceptedIssuers() {
						return null;
					}
	            } };  
	            SSLContext sslContext = SSLContext.getInstance("SSL", "SunJSSE");  
	            sslContext.init(null, tm, new java.security.SecureRandom());  
	            SSLSocketFactory ssf = sslContext.getSocketFactory();  
	            URL url = new URL(requestUrl);  
	            HttpsURLConnection httpUrlConn = (HttpsURLConnection) url.openConnection();  
	            httpUrlConn.setSSLSocketFactory(ssf);  
	            httpUrlConn.setDoOutput(true);  
	            httpUrlConn.setDoInput(true);  
	            httpUrlConn.setUseCaches(false);  
	            httpUrlConn.setRequestMethod(requestMethod);  
	            if ("GET".equalsIgnoreCase(requestMethod))  
	                httpUrlConn.connect();  
	            if (null != outputStr) {  
	                OutputStream outputStream = httpUrlConn.getOutputStream();  
	                outputStream.write(outputStr.getBytes("UTF-8"));  
	                outputStream.close();  
	            }  
	            InputStream inputStream = httpUrlConn.getInputStream();  
	            InputStreamReader inputStreamReader = new InputStreamReader(inputStream, "utf-8");  
	            BufferedReader bufferedReader = new BufferedReader(inputStreamReader);  
	            String str = null;  
	            while ((str = bufferedReader.readLine()) != null) {  
	                buffer.append(str);  
	            }  
	            bufferedReader.close();  
	            inputStreamReader.close();  
	            inputStream.close();  
	            inputStream = null;  
	            httpUrlConn.disconnect();  
//	            jsonObject = JSONObject.fromObject(buffer.toString());  
	            jsonObject = JSONObject.parseObject(buffer.toString());
	        } catch (ConnectException ce) {  
	            ce.printStackTrace();  
	        } catch (Exception e) {  
	            e.printStackTrace();  
	        }  
	        return jsonObject;  
	    }  


## 以json格式数据，进行post请求提交
	   /**
	    * postJsonDataRequest : 以json格式数据post提交请求
	    * @param requestUrl
	    * @param jsonData
	    * @return
	    */
	   public static String postJsonDataRequest(String requestUrl,JSONObject jsonData) {
		   String content = "";
		   try {
			URL url = new URL(requestUrl);
			 // 建立http连接
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            // 设置允许输出
            conn.setDoOutput(true);

            conn.setDoInput(true);

            // 设置不用缓存
            conn.setUseCaches(false);
            // 设置传递方式
            conn.setRequestMethod("POST");
            // 设置维持长连接
            conn.setRequestProperty("Connection", "Keep-Alive");
            // 设置文件字符集:
            conn.setRequestProperty("Charset", "UTF-8");
            //转换为字节数组
            byte[] data = (jsonData.toString()).getBytes();
            // 设置文件长度
            conn.setRequestProperty("Content-Length", String.valueOf(data.length));

            // 设置文件类型:
            conn.setRequestProperty("contentType", "application/json");
            
            // 开始连接请求
            conn.connect();
            OutputStream  out = conn.getOutputStream();     
            // 写入请求的字符串
            out.write((jsonData.toString()).getBytes());
            out.flush();
            out.close();
            // 请求返回的状态
            if (conn.getResponseCode() == 200) {
               log.info("post提交json数据响应成功");
                // 请求返回的数据
                InputStream in = conn.getInputStream();
                byte[] data1 = new byte[in.available()];
                in.read(data1);
                // 转成字符串
                content = new String(data1);
               log.info("提交处理响应字符串:>>" + content);
            } else {
               log.info("post提交json数据响应失败");
            }

		} catch (MalformedURLException e) {
			e.printStackTrace();
		} catch (ProtocolException e) {
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		   return content;
	   }
```
封装Httpclient进行请求发送的工具类库。包括GET,POST等请求：<br>
* 添加httpClient的maven依赖<br>
```xml
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
			<version>4.5.2</version>
		</dependency>
```
* 使用httpclient发送GET请求<br>
```java
	public static String requestGet(String url) throws IOException {
		String content = "";
		BufferedReader bufferedReader = null;
		//创建httpclient
		CloseableHttpClient httpclient = HttpClients.createDefault();
		
		try {

			//创建httpget
			HttpGet httpGet = new HttpGet(new URI(url));
			//执行get方法
			CloseableHttpResponse response = httpclient.execute(httpGet);
			try {
				
				bufferedReader = new BufferedReader(new InputStreamReader(response.getEntity().getContent()));
				StringBuffer sb = new StringBuffer("");
				String line = "";
				String NL = System.getProperty("line.separator");
				while ((line = bufferedReader.readLine()) != null) {
					sb.append(line + NL);
				}
				content = sb.toString();
			} catch (Exception e) {
				log.error("获取get请求响应结果失败：", e);
			}finally {
				response.close();
			}
			
		} catch (Exception e) {
			log.error("get请求发送失败：", e);
		}finally {
			httpclient.close();
			if(bufferedReader != null) {
				bufferedReader.close();
			}
		}
		log.info("content内容：---"+ content);
		return content;
	}
```
* 使用httpclient发送POST请求,以表单的形式<br>
```java
	public static String requestPost(String url, String formParam) throws IOException {
		String content = "";
		//创建httpclient
		CloseableHttpClient httpClient = HttpClients.createDefault();
		try {
			//创建post请求
			HttpPost httpPost = new HttpPost(new URI(url));
			
			httpPost.setEntity(new StringEntity(formParam, Charset.forName("UTF-8")));
			
			//执行post请求
			CloseableHttpResponse response = httpClient.execute(httpPost);
			try {
				HttpEntity entity = response.getEntity();
				content = EntityUtils.toString(entity, "UTF-8");
			} catch (Exception e) {
				log.error("获取post请求响应数据失败：", e);
			}finally {
				response.close();
			}
		} catch (Exception e) {
			log.error("发送post请求失败：", e);
		}finally {
			httpClient.close();
		}
		log.info("获取的post数据："+content);
		return content;
	}
```
这个以表单形式，进行post提交的使用方式如下:<br>
```java
		Map<String, Object> formMap = new HashMap<String, Object>();
		Map<String, Object> action_info = new HashMap<String, Object>();
		Map<String, Object> scene = new HashMap<String, Object>();
		scene.put("scene_id", articleId);
		action_info.put("scene", scene);
		formMap.put("expire_seconds", expire_seconds);
		formMap.put("action_name", action_name);
		formMap.put("action_info", action_info);

		String formParam = JSON.toJSONString(formMap);
		String responContent = HttpClient.requestPost(url, formParam);
```
