## 获取微信接口调用全局token凭证<br>
在做微信公众号开发时候，在项目中jsp页面或者其他页面中要调用微信JSSDK时候，必须先要根据APPID,SECRET调用获取ACCESS_TOKEN的接口，得到全局access_token,才能成功调用接口，因为微信服务器那边对每个调用接口，针对每个APPID都有次数频率限制的。这个全局access_token官方文档说明，理论上2小时过期，理想情况下，我们在项目中通过定时器定时刷新来实现中控方式对全局token进行管理。<br>
* **获取微信接口调用全局token凭证**<br>
因为调用微信官方的接口，会需要全局的接口调用凭证token,每两个小时就会过期，所以可以通过java的quartz定时器定时刷新。下面是调用获取微信全局token的请求方法。<br>
```java
  //获取微信接口调用的全局token
	private void getAccessToken() {
		StringBuffer url = new StringBuffer();
		url.append("https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential")
		   .append("&appid=").append(Constants.APPID)
		   .append("&secret=").append(Constants.APPSECRET);
		log.info("获取全局accesss_token的请求:>>" + url.toString());
		try {
			String content;
			ObjectMapper objectMapper = new ObjectMapper();
			/*
			 * 发送请求获取access_token，最多发送3次请求进行获取。
			 */
			for(int i = 1; i <= 3; i++) {
				if(httpUtil == null) {
					httpUtil = new HttpUtil();
				}
				content = httpUtil.executeGet(url.toString());
				try {
					Map map = objectMapper.readValue(content, Map.class);
					Object at = map.get("access_token");
					log.info("第" + i + "次定时器获取全局access_token:>>" + at);
					if(null != at) {
						//刷新内存中的全局ACCESS_TOKEN值。
						Constants.ACCESS_TOKEN = String.valueOf(at);
						log.info("全局access_token刷新成功!!");
						break;
					}
					log.info("全局access_token刷新失败!!");
				} catch (Exception e) {
					log.error("获取全局access_token时,json转换失败：" + e.getMessage());
					break;
				}
			}
	
		} catch (Exception e) {
			log.error("获取全局access_token失败：" + e.getMessage());
		}
		
	}


```
---