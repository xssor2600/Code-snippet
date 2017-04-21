## 百度地图<br>
记录在程序中使用的地图相关的代码块。<br>
* **根据经纬度查询地理位置**<br>
有这么一种需求，根据经纬度，要获取对应的详细地理位置信息。百度地图开发中有Geocoding API来可以完成。在调这些百度地图的web API之前，还要申请一个应用，拿到一个应用的at(类似访问认证标识吧东东)。<br>
详情：[Geocoding APIWeb服务API](http://lbsyun.baidu.com/index.php?title=webapi/guide/webservice-geocoding)<br>
[百度应用AK](https://passport.baidu.com/v2/?login&u=http%3A%2F%2Flbsyun.baidu.com%2Fapiconsole%2Fkey)<br>
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
