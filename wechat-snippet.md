
## Wechat代码库<br>
记录一些微信开发中常用到的代码块。<br>

* 获取code<br>
在springmvc中开发微信，得到code。
```java
	@RequestMapping(value="/routerToMyPage.html",method=RequestMethod.GET)
	public void redirectToMyPage(HttpServletResponse response){
//		String url = "https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=snsapi_base&state=123#wechat_redirect";
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
	/*
	* 通过上个页面跳转下面请求中，可以通过request.getParameter("code")获取code值。
	*/
	@RequestMapping(value="/myPage.html",method=RequestMethod.GET)
	public ModelAndView toMyPage(HttpServletRequest request){
		ModelAndView mv = new ModelAndView("/mypage/mypage");
		//通过request.getParameter("code")拿到code值。
		String code = request.getParameter("code");
		log.info("获取的code值为>>" + code);
		if(null == code) {
			return null;
		}
		....
		mv.addObject("code", code);
		return mv;
	}
```
---
* 获取微信用户的openId<br>
微信开发要获取openId,根据官方文档要获取openId,需要先获取code。所以下面使用了另外一种方法获取code，在code方法中请求转发方式，将请求发送到openId方法中，code和openId的获取都是分别通过springmvc进行请求分派。<br>
1.0 第一步，获取code对应的请求处理:<br>
```java
	@RequestMapping("/getCode.html")
	public void getCode(HttpServletRequest request, HttpServletResponse response) {
		
		String url = "";
		StringBuffer sb = new StringBuffer();
		//服务器域名
		String doname = ConfigService.getValue("DONAME", "doname");
		//项目根目录
		String root = request.getContextPath();
		//构造url
		sb.append(doname).append(root).append("/getOpenid/openId.html");
		
		try {
		/*
		* 对微信获取code的api中redirect_uri参数url进行编码。
		*/
		url = URLEncoder.encode(sb.toString(), "UTF-8");
		//清空stringbuffer
		sb.replace(0, sb.length(), "");
		sb.append("https://open.weixin.qq.com/connect/oauth2/authorize?appid=")
		.append(Constants.APPID)
		.append("&redirect_uri=")
		.append(url)
		.append("&response_type=code&scope=snsapi_base&state=123#wechat_redirect");
		
		log.info("url:"+sb.toString());
			//response.sendRedirect(stringBuffer.toString());
			//通过请求转发形式，跳转到openId请求中
			request.getRequestDispatcher(sb.toString()).forward(request, response);
		} catch (UnsupportedEncodingException e) {
			log.error("获取code值redirect_uri参数url编码失败：", e);
		} catch (Exception e) {
			log.error("内部转发失败，获取code失败：", e);
		}
	}
```
<br>
2.0 第二步，获取openId:<br>
```java
	@RequestMapping("/openId.html")
	@ResponseBody
	public void openId(HttpServletRequest request, HttpServletResponse response) {
		
		String content = "";
		String openId = "";
		String code = request.getParameter("code");

		StringBuffer url = new StringBuffer();
		url.append("https://api.weixin.qq.com/sns/oauth2/access_token?appid=")
		.append(Constants.APPID)
		.append("&secret=")
		.append(Constants.APPSECRET)
		.append("&code=")
		.append(code)
		.append("&grant_type=authorization_code");
		log.info("------获取的code："+code);
		try {
			/*
			 * 发送一个get请求调用微信获取openId的API.
			 */
			content = httpUtil.executeGet(url.toString());
			ObjectMapper objectMapper = new ObjectMapper();
			Map map = objectMapper.readValue(content, Map.class);
			Object topenId = map.get("openid");
			if (null != topenId) {
				openId = String.valueOf(map.get("openid"));
				log.info("获取的openID：" + openId);
				request.getSession().setAttribute("openId", openId);
			}
			log.info("openId为null,未成功保存到session中");
		} catch (JsonParseException | JsonMappingException e) {
			log.error("获取openId,返回的json转换失败：");
		} catch (Exception e) {
			log.error("获取openID失败：", e);
		} 
	}

```
