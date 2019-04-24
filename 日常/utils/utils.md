[TOC]



## 字符集转换

``` java
 public static String change2UTF8(String source,String encoding)
 {
  String result;
  try
  {
    ByteBuffer byteBuffer = ByteBuffer.wrap(source.getBytes(encoding));
    CharBuffer charBuffer = Charset.forName(encoding).decode(byteBuffer);
    ByteBuffer utf8Byte = Charset.forName(encoding).encode(charBuffer);
    result = new String(utf8Byte.array());
  }catch(IOException e)
  {
    return source;
  }
  return result;
 }
```





## 文件上传

- 文件上传是会将文件放在一个临时文件夹中，但是系统会经常清理临时文件夹，导致再上传文件时报错：

```properties
Could not parse multipart servlet request; nested exception is java.io.IOException: The temporary upload location [/tmp/tomcat.7333297176951596407.9000/work/Tomcat/localhost/ROOT] is not valid。
```

- 可以创建如下配置类解决这一问题-改变临时文件的储存路径

```java
@Configuration
public class MultipartConfig {

    /**
     * 文件上传临时路径
     */
    @Bean
    MultipartConfigElement multipartConfigElement() {
        MultipartConfigFactory factory = new MultipartConfigFactory();
        String location = System.getProperty("user.dir") + "/data/tmp";
        File tmpFile = new File(location);
        if (!tmpFile.exists()) {
            tmpFile.mkdirs();
        }
        factory.setLocation(location);
        return factory.createMultipartConfig();
    }
}
```



