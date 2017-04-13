
## Wechat代码库<br>
记录一些微信开发中常用到的代码块。<br>

* **获取code**<br>
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
* **获取微信用户的openId**<br>
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
2.0 第二步，获取openId:<br>
<br>
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
			log.error("获取openId,返回的json转换失败：" + e.getMessage());
		} catch (Exception e) {
			log.error("获取openID失败：", e);
		} 
	}

```
---
* **微信公众号平台服务器校验配置**<br>
在微信公众号开发或者微信开放平台操作，都要通过管理员对应用部署所在的服务器进行通讯校验，微信官方会要求我们上传一个校验文件到服务器，还会通过一些特殊字段的组合进行加密签名校验。<br>
<br>
```java
Controller("weinxinCheckToken")
@RequestMapping("/chenkToken")
public class WeinxinCheckToken {
	
	private Log log = LogFactory.getLog(WeinxinCheckToken.class);
	private final String token = "weixin";
	
	@RequestMapping("/wechat.html")
	public void checkToken(HttpServletRequest request, HttpServletResponse response) {
		log.info("开始签名校验");
		String signature = request.getParameter("signature");
		String timestamp = request.getParameter("timestamp");
		String nonce = request.getParameter("nonce");
		String echostr = request.getParameter("echostr");

		log.info("signature>>>" + signature);
		log.info("timestamp>>>" + timestamp);
		log.info("nonce>>>" + nonce);
		log.info("echostr>>>" + echostr);
		ArrayList<String> array = new ArrayList<String>();
		if (!"null".equals(signature)) {
			array.add(signature);
		}

		if (!"null".equals(timestamp)) {
			array.add(timestamp);
		}
		if (!"null".equals(nonce)) {
			array.add(nonce);
		}

		// 排序
		String sortString = sort(token, timestamp, nonce);
		// 加密
		String mytoken = SHA1(sortString);

		log.info("微信端发送signature>>>" + signature);
		log.info("SHA1加密后token:>>>>" + mytoken);
		log.info("MD5加密后token:>>>>" + mytoken);
		// 校验签名
		if (mytoken != null && mytoken != "" && mytoken.equals(signature)) {
			log.info("签名校验通过。");
			try {
				response.getWriter().println(echostr);
			} catch (IOException e) {
				log.error(e);
			} // 如果检验成功输出echostr，微信服务器接收到此输出，才会确认检验完成。
		} else {
			log.info("签名校验失败。");
		}
	}
  
  
  //对字段按照官方签名校验对字段进行排序
	public static String sort(String token, String timestamp, String nonce) {
		String[] strArray = { token, timestamp, nonce };
		Arrays.sort(strArray);

		StringBuilder sbuilder = new StringBuilder();
		for (String str : strArray) {
			sbuilder.append(str);
		}

		return sbuilder.toString();
	}

  //在将字段进行排序后，就可以进行签名加密
	public static String SHA1(String decript) {
	    try {
	      MessageDigest digest = MessageDigest
	          .getInstance("SHA-1");
	      digest.update(decript.getBytes());
	      byte messageDigest[] = digest.digest();
	      // Create Hex String
	      StringBuffer hexString = new StringBuffer();
	      // 字节数组转换为 十六进制 数
	      for (int i = 0; i < messageDigest.length; i++) {
	        String shaHex = Integer.toHexString(messageDigest[i] & 0xFF);
	        if (shaHex.length() < 2) {
	          hexString.append(0);
	        }
	        hexString.append(shaHex);
	      }
	      return hexString.toString();
	  
	    } catch (NoSuchAlgorithmException e) {
	      e.printStackTrace();
	    }
	    return "";
	  }

}

```
---
* **获取微信接口调用全局token凭证**<br>
因为调用微信官方的接口，会需要全局的接口调用凭证token,每两个小时就会过期，所以可以通过java的quartz定时器定时刷新。下面是调用获取微信全局token的请求方法。<br>
<br>
```
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
* **获取微信用户信息的请求方法**<br>
在三方平台，总要获取微信用户的基本信息，下面这个方法，是在基于得到全局token和已关注微信公众号的用户的openId基础下，获取微信用户的基本信息:在得到微信个人信息后，可以封装到自己平台定义的user用户对象中，以供后续使用。<br>
<br>
```java
 <br>
	//根据openId获取微信用户信息。
	public User getWechatUserInfo(String openId) {
		//获取保存在内存中的全局接口调用access_token
		String accessToken = Constants.ACCESS_TOKEN;
		log.info("全局token>>" + accessToken);
		StringBuffer url = new StringBuffer();
		url.append("https://api.weixin.qq.com/cgi-bin/user/info?")
		   .append("access_token=").append(accessToken)
		   .append("&openid=").append(openId).append("&lang=zh_CN");
		
		log.info("根据access_token与openId获取微信用户信息:>>" + url.toString());
		
		String content;
		ObjectMapper objectMapper = new ObjectMapper();
		
		try {
      /*
      * 有时候会由于网络原因首次加载失败，所以允许最多三次进行获取信息。
      */
			for(int i = 1; i <= 3; i++) {
				content = httpUtil.executeGet(url.toString());
				log.info("获取微信用户请求响应信息:>>" + content);
				try {
					Map map = objectMapper.readValue(content, Map.class);
					Object mopenId = map.get("openid");
					Object nickName = map.get("nickname");
					log.info("第" + i + "次获取openId=" + openId + "的微信用户昵称：>>" + nickName);
					if(mopenId.equals(openId) && nickName != null) {
						/*
						 * 获取微信用户基本信息成功，并将信息封装到平台用户对象中。
						 */
						User user = new User();
						user.setNickName(String.valueOf(nickName));
						user.setSex((Integer)map.get("sex"));
						user.setPhotoUrl(String.valueOf(map.get("headimgurl")));
						user.setWxOpenId(String.valueOf(mopenId));
						user.setWxUnionId(String.valueOf(map.get("unionid")));
						log.info("调用微信得到的用户信息:>>" + user.getNickName() + ",photo>>" + user.getPhotoUrl());
						return user;
					}
					log.info("第" + i + "次获取openId=" + openId + "的微信用户信息失败!!");
				} catch (Exception e) {
					log.error("获取微信基本用户信息时,json转换失败：" + e.getMessage());
					break;
				}
			}
			
			
		} catch (Exception e) {
			log.error("http请求执行错误:>>" + e.getMessage());
			e.printStackTrace();
		}
		return null;
	}

```
