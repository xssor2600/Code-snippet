## 微信JSSDK开发配置<br>
在做微信公众号开发时候，需要在微信网页中调用微信api接口，那么就是属于JSSDK的范畴。微信网页对于JSSDK的规范也有一定的规则。<br>
除了在微信公众平台绑定域名，引入js文件外，还要进行页面权限验证的配置。下面的代码块就是分别将页面前端和服务器后端对于微信JSSDK的使用方式给列出来。详情可见:[微信JS-SDK说明文档](https://mp.weixin.qq.com/wiki/11/74ad127cc054f6b80759c40f77ec03db.html)<br>

* 页面引入js文件，设置wx.config配置: <br>
在这里使用jsp作为前端页面来显示，其实不一定就只有一种形式的页面使用或者校验方式，应该还有其他可行的途径。这里只是因为自己的开发经验与实践可行后，就将这种使用形式场景给列出来:<br>
```javascript
<%@ page language="java" import="java.util.*,com.cybbj.action.weixin.WxConfig" pageEncoding="UTF-8"%>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";

Map<String,Object>  ret = new HashMap<String,Object> ();  
ret=WxConfig.getWxConfig(request);  
request.setAttribute("appId", ret.get("appId"));  
request.setAttribute("timestamp", ret.get("timestamp"));  
request.setAttribute("nonceStr", ret.get("nonceStr"));  
request.setAttribute("signature", ret.get("signature"));  
%>
<!DOCTYPE html>
<html>
  <head>
    <base href="<%=basePath%>">
    
    <title>信息提示:</title>
    <meta name="viewport" content="initial-scale=1, maximum-scale=1">
	<meta http-equiv="pragma" content="no-cache">
	<meta http-equiv="cache-control" content="no-cache">
	<meta http-equiv="expires" content="0">    
	<script src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
	<script type="text/javascript" src="https://res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>

<script type="text/javascript">
$(function(){
wx.config({
    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来
    appId: '${appId}', // 必填，公众号的唯一标识
    timestamp:'${timestamp}' , // 必填，生成签名的时间戳
    nonceStr: '${nonceStr}', // 必填，生成签名的随机串
    signature: '${signature}',// 必填，签名，见附录1
    jsApiList: [
    	'checkJsApi',
        'openLocation',
        'getLocation',
        'onMenuShareTimeline',
        'onMenuShareAppMessage'
    ] 
	});
	
wx.ready(function () {
wx.getLocation({
    success: function (res) {
        var latitude = res.latitude; // 纬度，浮点数，范围为90 ~ -90
        var longitude = res.longitude; // 经度，浮点数，范围为180 ~ -180。
        var speed = res.speed; // 速度，以米/每秒计
        var accuracy = res.accuracy; // 位置精度
    },
    cancel: function (res) {
        alert('用户拒绝授权获取地理位置');
    }
});

//   wx.checkJsApi({
//             jsApiList: [
//                 'getLocation',
//                 'onMenuShareTimeline',
//                 'onMenuShareAppMessage'
//             ],
//             success: function (res) {
//                 alert(JSON.stringify(res));
//             }
//         });


});

});

            

</script>

</head>
  <body>
  <button onclick="">获取微信用户当前位置坐标</button>
  </body>
</html>

```
可以看到，前端jsp页面与后台有交互的，仅仅只有一处，就是在`WxConfig.getWxConfig(request)`，将request传递到后台，处理后，返回信息都封装到一个map中了。这个map中封装着appid,timestamp,nonceStr,signature等数据，用于传递到wx.config中进行校验。<br>
<br>
* 服务器后端config的校验:<br>
在前端进行校验前，就是通过WxConfig.getWxConfig()方法从后台获取校验数据，那么，后台中是如何处理这些数据呢？看看下面代码块是如何实现的把。<br>
```java
public class WxConfig {
	
	private static Log log = LogFactory.getLog(WxConfig.class);
	
	/** 
	  * 方法名：httpRequest</br> 
	  * 详述：发送http请求</br> 
	  * @param requestUrl 
	  * @param requestMethod 
	  * @param outputStr 
	  * @return 说明返回值含义 
	  * @throws 说明发生此异常的条件 
	*/  
	public static Map<String, Object> getWxConfig(HttpServletRequest request) {
		Map<String, Object> ret = new HashMap<String, Object>();
		String appId = Constants.APPID; // 必填，公众号的唯一标识
		String secret = Constants.APPSECRET;

		//设置的js调用域名
		String doname = ConfigService.getValue("DONAME", "doname");
		String context = doname + request.getContextPath();
		String handlerMappingUrl = (String) request.getAttribute("org.springframework.web.servlet.HandlerMapping.pathWithinHandlerMapping");
		
		StringBuffer requestUrl = request.getRequestURL();
//		log.info("原始请求连接SS>>:" + request.getRequestURI().toString());
		String queryString = request.getQueryString();
		log.info("查询参数>>:" + queryString);
        String fullRequestUrl = "";
        
        /*
         * 若是不通过springmvc进行请求映射的url就不进行处理 
         */
        if(handlerMappingUrl == null || "null".equals(handlerMappingUrl)) {
        	fullRequestUrl = requestUrl.toString();
        } else {	//将springmvc映射路径替换/WEB-INF/下的jsp绝对路径
        	fullRequestUrl = context + handlerMappingUrl;
        }
        
        if(queryString != null && !"null".equals(queryString)) {
   		   fullRequestUrl = fullRequestUrl +"?"+queryString;
   	   }
        
	    log.info("访问的url>>" + fullRequestUrl);
		String access_token = "";
		String jsapi_ticket = "";
		String timestamp = Long.toString(System.currentTimeMillis() / 1000); // 必填，生成签名的时间戳
		String nonceStr = UUID.randomUUID().toString(); // 必填，生成签名的随机串
		String url = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid="
				+ appId + "&secret=" + secret;

		JSONObject json = HttpClient.httpRequestReturnByJson(url, "GET", null);
		
		if (json != null) {
			// 要注意，access_token需要缓存
			access_token = json.getString("access_token");
//			access_token = Constants.ACCESS_TOKEN;

			url = "https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token="
					+ access_token + "&type=jsapi";
			json = HttpClient.httpRequestReturnByJson(url, "GET", null);
			if (json != null) {
				jsapi_ticket = json.getString("ticket");
				log.info("得到的jsapi ticket:>>" + jsapi_ticket);
			}
		}
		String signature = "";
		// 注意这里参数名必须全部小写，且必须有序
		String sign = "jsapi_ticket=" + jsapi_ticket + "&noncestr=" + nonceStr
				+ "&timestamp=" + timestamp + "&url=" + fullRequestUrl;
		try {
			MessageDigest crypt = MessageDigest.getInstance("SHA-1");
			crypt.reset();
			crypt.update(sign.getBytes("UTF-8"));
			signature = byteToHex(crypt.digest());
		} catch (NoSuchAlgorithmException e) {
			e.printStackTrace();
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		}
		ret.put("appId", appId);
		ret.put("timestamp", timestamp);
		ret.put("nonceStr", nonceStr);
		ret.put("signature", signature);
//		log.info("appId:>>" + appId);
		log.info("timestamp:>>" + timestamp);
		log.info("nonceStr:>>" + nonceStr);
		log.info("signature:>>" + signature);
		return ret;
	}
	
	 private static String byteToHex(final byte[] hash) {  
	        Formatter formatter = new Formatter();  
	        for (byte b : hash) {  
	            formatter.format("%02x", b);  
	        }  
	        String result = formatter.toString();  
	        formatter.close();  
	        return result;  
	  
	    }  

}

```
当前后端都按照流程运行完，wx.config校验就会ok,那么后面调用所有的JSSDK接口都能成功了。。。

