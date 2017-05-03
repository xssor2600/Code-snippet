## Converts工具类库<br>
在开发过程中，总会遇到不同数据类型的转换问题，例如十六进制字符串转字节数组，字节数组转换成十六进制字符串等等，这个Converts工具类就是集合了这些转换功能的工具方法集合。<br>
工具类的全局基础字段:<br>
```java
public final static char[] BToA = "0123456789abcdef".toCharArray();
```
* 把16进制字符串转换成字节数组<br>
```java
	public static byte[] hexStringToByte(String hex) {
		int len = (hex.length() / 2);
		byte[] result = new byte[len];
		char[] achar = hex.toCharArray();
		for (int i = 0; i < len; i++) {
			int pos = i * 2;
			result[i] = (byte) (toByte(achar[pos]) << 4 | toByte(achar[pos + 1]));
		}
		return result;
	}

	private static byte toByte(char c) {
		byte b = (byte) "0123456789ABCDEF".indexOf(c);
		return b;
	}
```
* 把字节数组转换成16进制字符串<br>
```java
	public static final String bytesToHexString(byte[] bArray) {
		StringBuffer sb = new StringBuffer(bArray.length);
		String sTemp;
		for (int i = 0; i < bArray.length; i++) {
			sTemp = Integer.toHexString(0xFF & bArray[i]);
			if (sTemp.length() < 2)
				sb.append(0);
			sb.append(sTemp.toUpperCase());
		}
		return sb.toString();
	}
```
* 把字节数组转换为对象<br>
```java
	public static final Object bytesToObject(byte[] bytes) throws IOException,
			ClassNotFoundException {
		ByteArrayInputStream in = new ByteArrayInputStream(bytes);
		ObjectInputStream oi = new ObjectInputStream(in);
		Object o = oi.readObject();
		oi.close();
		return o;
	}
```
* 把可序列化对象转换成字节数组<br>
```java
	public static final byte[] objectToBytes(Serializable s) throws IOException {
		ByteArrayOutputStream out = new ByteArrayOutputStream();
		ObjectOutputStream ot = new ObjectOutputStream(out);
		ot.writeObject(s);
		ot.flush();
		ot.close();
		return out.toByteArray();
	}

	public static final String objectToHexString(Serializable s)
			throws IOException {
		return bytesToHexString(objectToBytes(s));
	}

	public static final Object hexStringToObject(String hex)
			throws IOException, ClassNotFoundException {
		return bytesToObject(hexStringToByte(hex));
	}
```
* BCD码转为10进制串(阿拉伯数据)<br>
```java
public static String bcd2Str(byte[] bytes) {
		StringBuffer temp = new StringBuffer(bytes.length * 2);
		for (int i = 0; i < bytes.length; i++) {
			temp.append((byte) ((bytes[i] & 0xf0) >>> 4));
			temp.append((byte) (bytes[i] & 0x0f));
		}
		return temp.toString().substring(0, 1).equalsIgnoreCase("0") ? temp
				.toString().substring(1) : temp.toString();
	}

```
* 10进制串转为BCD码<br>
```java
	public static byte[] str2Bcd(String asc) {
		int len = asc.length();
		int mod = len % 2;
		if (mod != 0) {
			asc = "0" + asc;
			len = asc.length();
		}
		byte abt[] = new byte[len];
		if (len >= 2) {
			len = len / 2;
		}
		byte bbt[] = new byte[len];
		abt = asc.getBytes();
		int j, k;
		for (int p = 0; p < asc.length() / 2; p++) {
			if ((abt[2 * p] >= '0') && (abt[2 * p] <= '9')) {
				j = abt[2 * p] - '0';
			} else if ((abt[2 * p] >= 'a') && (abt[2 * p] <= 'z')) {
				j = abt[2 * p] - 'a' + 0x0a;
			} else {
				j = abt[2 * p] - 'A' + 0x0a;
			}
			if ((abt[2 * p + 1] >= '0') && (abt[2 * p + 1] <= '9')) {
				k = abt[2 * p + 1] - '0';
			} else if ((abt[2 * p + 1] >= 'a') && (abt[2 * p + 1] <= 'z')) {
				k = abt[2 * p + 1] - 'a' + 0x0a;
			} else {
				k = abt[2 * p + 1] - 'A' + 0x0a;
			}
			int a = (j << 4) + k;
			byte b = (byte) a;
			bbt[p] = b;
		}
		return bbt;
	}

	public static String BCD2ASC(byte[] bytes) {
		StringBuffer temp = new StringBuffer(bytes.length * 2);
		for (int i = 0; i < bytes.length; i++) {
			int h = ((bytes[i] & 0xf0) >>> 4);
			int l = (bytes[i] & 0x0f);
			temp.append(BToA[h]).append(BToA[l]);
		}
		return temp.toString();
	}
```
* 将两个ASCII字符合成一个字节<br>
如："EF"--> 0xEF<br>
```java
	public static byte uniteBytes(byte src0, byte src1) {
		byte _b0 = Byte.decode("0x" + new String(new byte[] { src0 })).byteValue();
		_b0 = (byte) (_b0 << 4);
		byte _b1 = Byte.decode("0x" + new String(new byte[] { src1 })).byteValue();
		byte ret = (byte) (_b0 ^ _b1);
		return ret;
	}
```
* 将指定字符串src，以每两个字符分割转换为16进制形式<br>
如："2B44EFD9" –> byte[]{0x2B, 0×44, 0xEF, 0xD9}<br>
```java
	public static byte[] HexString2Bytes(String src) {
		byte[] ret = new byte[src.length() / 2];
		byte[] tmp = src.getBytes();
		for (int i = 0; i < tmp.length / 2; i++) {
			ret[i] = uniteBytes(tmp[i * 2], tmp[i * 2 + 1]);
		}
		return ret;
	}
```
* byte数组转long<br>
```java
	public static long bytesToLong(byte[] bytes) {
		 ByteBuffer buffer = ByteBuffer.allocate(8);
		 buffer.put(bytes, 0, bytes.length);
		 buffer.flip();// need flip
		 return buffer.getLong();
	}
	
	public static byte[] int2bytes(int num)
	{
	   byte[] b=new byte[4];
	   for(int i=0;i<4;i++){
		   b[i]=(byte)(num>>>(24-i*8));
	   }
	   return b;
	}
	
	public static int bytes2int(byte[] b)
	{
		int mask=0xff;
		int temp=0;
		int res=0;
		for(int i=0;i<4;i++){
	        res<<=8;
	        temp=b[i]&mask;
	        res|=temp;
		}
		return res;
	} 
	public static boolean isEqualArray(byte[] a, byte[] b) {
		if (a.length != b.length) {
			return false;
		}
		for (int i = 0; i < a.length; ++i) {
			if (a[i] != b[i])
				return false;
		}
		return true;
	}
```