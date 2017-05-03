## QRCODE二维码生成器<br>
记录在做二维码项目开发过程中，生成二维码的工具方法。<br>

(1) 先添加二维码生成maven依赖<br>
```java
		<dependency>
			<groupId>com.google.zxing</groupId>
			<artifactId>core</artifactId>
			<version>3.2.1</version>
		</dependency>
```

(2) 二维码生成器<br>
下面这个工具类中，生成的二维码中间是没有图片的。<br>
```java
public class GenerateCode {
	
	private Log log = LogFactory.getLog(GenerateCode.class);
	private static final int BLACK = 0xff000000;
	private static final int WHITE = 0xFFFFFFFF;

	 * @param args
	 */
	public static void main(String[] args) {
		GenerateCode test = new GenerateCode();

		String filePostfix="png";
		File file = new File("d://test_QR_CODE."+filePostfix);
		try {
			test.encode(new String("hello 中国,I'm Hongten.welcome to my zone".getBytes(),"ISO-8859-1"), 
				file,filePostfix, BarcodeFormat.QR_CODE, 430, 165, null);
		} catch (Exception e) {
			e.printStackTrace();
		}
		
		//test.decode(file);
	}

	/**
	 *  生成QRCode二维码<br> 
	 *  在编码时需要将com.google.zxing.qrcode.encoder.Encoder.java中的<br>
	 *  static final String DEFAULT_BYTE_MODE_ENCODING = "ISO8859-1";<br>
	 *  修改为UTF-8，否则中文编译后解析不了<br>
	 * @param contents 二维码的内容
	 * @param file 二维码保存的路径，如：C://test_QR_CODE.png
	 * @param filePostfix 生成二维码图片的格式：png,jpeg,gif等格式
	 * @param format qrcode码的生成格式
	 * @param width 图片宽度
	 * @param height 图片高度
	 * @param hints
	 */
	public void encode(String contents, File file,String filePostfix, BarcodeFormat format, int width, int height, Map<EncodeHintType, ?> hints) {
		try {
			BitMatrix bitMatrix = new MultiFormatWriter().encode(contents, format, width, height);
			writeToFile(bitMatrix, filePostfix, file);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	/**
	 * 生成二维码图片<br>
	 * 
	 * @param matrix
	 * @param format
	 *            图片格式
	 * @param file
	 *            生成二维码图片位置
	 * @throws IOException
	 */
	public static void writeToFile(BitMatrix matrix, String format, File file) throws IOException {
		BufferedImage image = toBufferedImage(matrix);
		ImageIO.write(image, format, file);
	}

	/**
	 * 生成二维码内容<br>
	 * 
	 * @param matrix
	 * @return
	 */
	public static BufferedImage toBufferedImage(BitMatrix matrix) {
		int width = matrix.getWidth();
		int height = matrix.getHeight();
		BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_ARGB);
		for (int x = 0; x < width; x++) {
			for (int y = 0; y < height; y++) {
				image.setRGB(x, y, matrix.get(x, y) == true ? BLACK : WHITE);
			}
		}
		return image;
	}


}
```
(3) 使用测试例子<br>
```java
	public static void main(String[] args) {
		GenerateCode test = new GenerateCode();

		String filePostfix="png";
		File file = new File("d://test_QR_CODE."+filePostfix);
		try {
			test.encode(new String("hello 中国,I'm Hongten.welcome to my zone".getBytes(),"ISO-8859-1"), file,filePostfix, BarcodeFormat.QR_CODE, 430, 165, null);
		} catch (Exception e) {
			e.printStackTrace();
		}
		
		//test.decode(file);
	}
```