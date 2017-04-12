
## Wechat代码库<br>
记录一些微信开发中常用到的代码块。<br>

* 获取code<br>
在springmvc中开发微信，得到code。
```java
	@RequestMapping(value="/routerToMyPage.html",method=RequestMethod.GET)
	public void redirectToMyPage(HttpServletResponse response){
//		String url = "https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=snsapi_base&state=123#wechat_redirect";
////		String url = "https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=https://IP/xiangbideWXBG/my/myPage.html&response_type=code&scope=snsapi_base&state=123#wechat_redirect";
//		return "redirect:"+url;
		StringBuffer sb = new StringBuffer(300);
		sb.append("https://open.weixin.qq.com/connect/oauth2/authorize?appid=");
//		String appId = ConfigService.getValue("", "APPID");
		String appId = "";
		sb.append(appId);
		String url;
		try {
			url = URLEncoder.encode("https://IP/xiangbideWXBG/my/myPage.html", "utf-8");
			sb.append("&redirect_uri=").append(url);
			sb.append("&response_type=code&scope=snsapi_base&state=123#wechat_redirect");
//			mv.setViewName("redirect:" + sb.toString());
			log.info("url>>" + sb.toString());
			response.sendRedirect(sb.toString());
		} catch (Exception e) {
			log.error("重定向url编码失败：>>" + e.getMessage());
			e.printStackTrace();
		}
	}
```
