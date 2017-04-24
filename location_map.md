## 百度地图<br>
记录在程序中使用的地图相关的代码块。<br>
* **根据经纬度查询地理位置**<br>
有这么一种需求，根据经纬度，要获取对应的详细地理位置信息。百度地图开发中有Geocoding API来可以完成。在调这些百度地图的web API之前，还要申请一个应用，拿到一个应用的at(类似访问认证标识吧东东)。<br>
详情：<u>[Geocoding APIWeb服务API](http://lbsyun.baidu.com/index.php?title=webapi/guide/webservice-geocoding)</u> <u>[百度应用AK](https://passport.baidu.com/v2/?login&u=http%3A%2F%2Flbsyun.baidu.com%2Fapiconsole%2Fkey)</u><br>
```javascript
      ## ak=xxxxxx,xxxx代表申请的一个百度应用AK.
 $.ajax({  
        url:"http://api.map.baidu.com/geocoder/v2/?callback=renderReverse&location=23.11827687049967,113.40788939960324&output=json&pois=1&ak=xxxxxxxxxxxxx",  
        dataType:'jsonp', 
        data:'',  
        callback:'callback',
        success:function(result) {
        alert(result);
	    	alert(result.result.formatted_address); //返回一个json对象，内部有详细地理位置信息
        },
        error:function(){
        alert('geocoding 调用失败');
        }
        timeout:3000  
    	}); 

```
<br>
* **根据两点经纬度坐标计算距离**<br>
若是知道两点的经纬度坐标，那么在精确度要求不是很高的请情况下，可以使用百度javascript开发库中的BMapLibrary.GeoUtil工具类中的`#getDistance(point,point)`方法。详情：[类 BMapLib.GeoUtils](http://api.map.baidu.com/library/GeoUtils/1.2/docs/symbols/BMapLib.GeoUtils.html#.getDistance?qq-pf-to=pcqq.c2c)  [百度地图-JavaScript API v2.0](http://lbsyun.baidu.com/index.php?title=jspopular) <br>
```javascript
## 导入基础jquery或者其他前端js框架和百度地图api的js
<script src="http://api.map.baidu.com/api?v=2.0&ak=密钥" type="text/javascript"></script>
//两点经纬度
var lat1,lon1;
var lat2,lon2;
//计算距离
var distance = BMapLib.GeoUtils.getDistance(new BMap.Point(lon1,lat1),new BMap.Point(lon2,lat2));

```