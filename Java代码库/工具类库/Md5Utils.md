## MD5Utils工具类库<br>
通常在实际的业务处理中，若是有加密，解密，哈希处理要求的情况下，可以使用MD5进行处理。<br>
其中，Converts工具类参考[Converts工具类库](https://github.com/arden2600/Code-snippet/blob/master/Java%E4%BB%A3%E7%A0%81%E5%BA%93/%E5%B7%A5%E5%85%B7%E7%B1%BB%E5%BA%93/ConvertsUtils.md)<br>

* MD5加密字符串，返回加密后的16进制字符串<br>
```java
	public static String MD5EncodeToHex(String origin) throws UnsupportedEncodingException {
		return Converts.bytesToHexString(MD5Encode(origin));
	}
```
* MD5加密字符串，返回加密后的字节数组<br>
```java
	public static byte[] MD5Encode(String origin) throws UnsupportedEncodingException {
		return MD5Encode(origin.getBytes("UTF-8"));
	}
```
* MD5加密字节数组，返回加密后的字节数组<br>
```java
	public static byte[] MD5Encode(byte[] bytes) {
		MessageDigest md = null;
		try {
			md = MessageDigest.getInstance("MD5");
			return md.digest(bytes);
		} catch (NoSuchAlgorithmException e) {
			e.printStackTrace();
			return new byte[0];
		}

	}
```