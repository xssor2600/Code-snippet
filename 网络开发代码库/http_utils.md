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
