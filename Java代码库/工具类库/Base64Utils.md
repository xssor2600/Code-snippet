## Base64<br>
* Base64工具类
```java

package com.outstanding.framework.utils;

import org.apache.commons.codec.binary.Base64;

import java.io.UnsupportedEncodingException;

/**
 * Base64加解密工具类
 */
public class Base64Util {

    private static final String CHARSET_NAME = "UTF-8";
    /**
     * Base64解码
     * 
     * @param str
     * @return
     */
    public static byte[] decode(String str) throws UnsupportedEncodingException {
        Base64 base64 = new Base64();
        return base64.decode(str.getBytes(CHARSET_NAME));
    }

    /**
     * Base64解码
     */
    public static String decodeString(String str) throws UnsupportedEncodingException {
        Base64 base64 = new Base64();
        return new String(base64.decode(str.getBytes(CHARSET_NAME)),CHARSET_NAME);
    }

    /**
     * Base64编码
     */
    public static String encode(byte[] b) throws UnsupportedEncodingException {
        Base64 base64 = new Base64();
        return new String(base64.encode(b),CHARSET_NAME);
    }

    /**
     * Base64编码
     */
    public static byte[] encodeByte(byte[] b) {
        Base64 base64 = new Base64();
        return base64.encode(b);
    }
}

```

