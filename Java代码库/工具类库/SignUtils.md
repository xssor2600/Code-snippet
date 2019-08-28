## 签名工具类<br>
* 签名工具类
```java
package com.chaos.open.common.util;


import com.alibaba.fastjson.JSON;
import com.outstanding.framework.core.PendingException;
import org.apache.commons.beanutils.BeanMap;

import java.util.Collections;
import java.util.LinkedList;

/**
 * 签名工具类
 **/
public class SignUtil {

    public static String getSignStr(Object o) throws PendingException {
        BeanMap beanMap = new BeanMap(o);
        LinkedList<String> keyList = new LinkedList<>();
        for (Object key : beanMap.keySet()) {
            Object value = beanMap.get(key);
            if (value != null && value != "") {
                String s = (String) key;
                if (!"sign".equals(s) && !"signData".equals(s) && !"sign_type".equals(s) && !"ipAddress".equals(s) && !"class".equals(s)) {
                    keyList.add(s);
                }
            }

        }

        Collections.sort(keyList);

        StringBuilder builder = new StringBuilder();

        for (String key : keyList) {
            builder.append(key);
            builder.append("=");
			final Object value = beanMap.get(key);
			if("biz_result".equalsIgnoreCase(String.valueOf(key))){
				String bizResult = JSON.toJSONString(value).replace("\\","");
				String newStr = bizResult.substring(1, bizResult.length() - 1);
				builder.append(newStr);
			}else{
				builder.append(value);
			}
            builder.append("&");
        }

        String signStr = builder.substring(0, builder.length() - 1);

        return signStr;

    }
}
```