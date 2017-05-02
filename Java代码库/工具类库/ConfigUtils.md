## ConfigUtils配置文件工具类<br>
在开发中，为了将某些容易改变，变动的数据项与代码解耦，防止硬编码对日后维护修改的成本加大。所以，通常也常常使用键值对KEY-VALUE形式的properties文件对一些数据项进行配置，那么这个工具ConfigUtils就是为了读取这些配置项服务的。<br>

config.properties就是项目的配置项文件，并且放置在`src/main/resources`目录下，当通过maven编译发布后，该文件会在项目的classes目录下。<br>
* 读取并解析配置文件config.properties<br>
```java
public final class ConfigUtils {

    /** logger. */
	private static Log log  =  LogFactory.getLog(ConfigUtils.class);

    /**
     * 内部使用Map, key是section的名称，value是该section包含的Properties.
     */
    private static Map<String, Properties> sectionMap =
            new HashMap<String, Properties>();

    /** 配置文件名称. */
    private static final String CONFIG_FILE_NAME = "config.properties";

    /* 初始化区域，读取config.properties的内容并构建内部的Map. */
    static {
		String sTempPath = System.getProperty("classesPath");
		
		String sPath = sTempPath+CONFIG_FILE_NAME;
		log.info(">>path: " + sPath);
        BufferedReader reader = null;
        try {
        	reader = new BufferedReader(new FileReader(sPath));
            /* 分行读取配置文件内容并解析之 */
            String line = null;
            String currentSectionName = null;
            Properties currentProperties = null;
            while ((line = reader.readLine()) != null) {
                line = line.trim();
                if (line.length() == 0 || line.charAt(0) == '#') {
                    /* 忽略空行与注释行 */
                    continue;
                }
                if (line.matches("\\[.*\\]")) {
                    /* 本行是个section定义行，创建新的section并加入map*/
                    currentSectionName = line.replaceFirst("\\[(.*)\\]", "$1");
                    currentProperties =  new Properties();
                    sectionMap.put(currentSectionName, currentProperties);
                } else if (line.matches(".*=.*")) {
                    /* 本行是个property定义行，读取property的名称与值并记录*/
                    if (currentProperties != null) {
                        int i = line.indexOf('=');
                        currentProperties.setProperty(
                                line.substring(0, i).trim(),
                                line.substring(i + 1).trim());
                    }
                }
            }
            reader.close();
//            logAllProperties();
        } catch (FileNotFoundException e) {
            /* 未找到config.properties */
        	log.error("Unable to find property file.");
        } catch (IOException e) {
            /* 无法读取config.properties的内*/
        	log.error("Unable to read content of property file.");
        }
    }

    /**
     * 返回指定section与property在config.properties中的配置
     * @param sectionName section name
     * @param propertyName property name
     * @return 返回sectionName与propertyName应的配置值，如果没有相关定义，返回null值
     */
    public static String getValue(
            final String sectionName, final String propertyName) {

        String value = null; // 返回null
        Properties properties =
                (Properties) ConfigService.sectionMap.get(sectionName);
        if (properties != null) {
            value = properties.getProperty(propertyName);
        }
        return value;
    }

    /** 私有构造方法避免实例化本类. */
    private ConfigUtils() {
    }
}


```
假设config.properties中配置这一个项目全局编码:<br>
```xml
#########################################################################
# all encode
#########################################################################
[ENCODECONFIG]
#编码配置
encode=UTF-8
```
那么，如何使用这个ConfigUtils呢，很简单，就像下面代码块一样:<br>
```java
## 第一个参数是config.properties中[]中名称，第二个值则为key值
String encode = ConfigUtils.getValue("ENCODECONFIG","encode");
```