## 文件上传
首先在页面文件定义一个 form 组件，enctype 为 `multipart/form-data`，method 为 `post`
``` xml
<form action="upload" enctype="multipart/form-data" method="post">
  <input type="file" name="file"/><br/>
  <input type="submit" value="upload"/>
</form>
```

### Spring 配置
Spring MVC 专门提供了 `CommonsMultipartResolver` 类用于文件上传，在容器中进行注册

使用 xml 配置方式为：
``` xml
<!-- id 必须为 multipartResolver，同理还有 localeResolver / themeResolver -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!-- 默认编码 -->
	<property name="defaultEncoding" value="utf-8" />
	<!-- 文件最大限制，单位是byte -->
	<property name="maxUploadSize" value="10485760000" />
	<!-- 低于这个大小的文件暂存内存中 -->
	<property name="maxInMemorySize" value="4096" />
</bean>
```

使用 Java 配置为：
``` java
@Bean
public MultipartResolver multipartResolver() {
  CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver();
  multipartResolver.setMaxUploadSize(1000000);
  return multipartResolver;
}
```

对于 Spring Boot 来说，我们不需要配置文件上传的解析类了，因为 Spring Boot 已经帮我们注册好了

### 单文件上传
在处理请求的方法中，我们可以在传入的实参 `MultipartHttpServletRequest` 对象中获取一个 `MultipartFile` 对象
``` java
// 将普通的 HttpServletRequest 对象转换为 MultipartHttpServletRequest 对象
MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest) request;
// 通过 getFile() 方法获取从表单中上传过来的 file，参数 "file" 必须与 form 表单中 input 标签的 name 是一致的
CommonsMultipartFile file = (CommonsMultipartFile) multipartRequest.getFile("file");

FileCopyUtils.copy(file.getBytes(), uploadFile);
```

或者直接将 `MultipartFile` 对象作为参数传入
``` java
@RequestMapping(...)
public @ResponseBody String upload(MultipartFile file) {
  try {
    FileUtils.writeByteArrayToFile(new File(dirPath + file.getOriginalFilename()), file.getBytes());
    return "ok";
  } catch (IOException e) {
    e.printStackTrace();
    return "wrong";
  }
}
```

### 多文件上传
处理多文件上传与单文件上传不同点仅仅是 `MultipartHttpServletRequest` 对象的方法不同，`getFile` 方法用于获取一个 `MultipartFile` 对象，`getFileMap` 方法获取一组关于 `MultipartFile` 的键值对
``` java
MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest) request;

Map<String, MultipartFile> fileMap = multipartRequest.getFileMap();

for(Map.Entry<String, MultipartFile> entity : fileMap.entrySet()) {
    MultipartFile mf = entity.getValue();
    fileName = mf.getOriginalFilename();
    File uploadFile = new File(path + fileName);
    FileCopyUtils.copy(mf.getBytes(), uploadFile);
}
```

## 文件下载
我们知道对于 Java Web 来说，无论是返回页面、JSON 字符串还是文件，本质上还是 `HttpServletResponse` 的 I/O 流输出。因此我们只需要将 `HttpServletResponse` 设置为文件的输出要求，并通过 `OutputStream` 输出即可：
* 设置编码格式 (`ContentType`) 为 `text/html;charset=utf-8` 或 `application/octet-stream` 或 `application/x-msdownload`
* 设置 `Content-Length` 属性，值为文件的大小
* 设置 header 中 `Content-Disposition` 属性值为 `attachment; filename=文件名`

``` java
@RequestMapping(value="...", method=RequestMethod.GET)
public void downloadFile(HttpServletRequest request, HttpServletResponse response) {

    // 假设获取本地 File 对象为 fileLocal
    response.setContentType("application/octet-stream");
    response.setContentLength((int) fileLocal.length());
    response.setHeader("Content-Length", "attchment;filename=" + new String(fileName.getByte("utf-8"), "ISO8859-1"));

    FileInputStream fis = null;
    OutputStream os = null;

    try {
        fis = new FileInputStream(file);
        os = response.getOutputStream();

        IOUtils.copy(fis, os);

        response.flushBuffer();
    } catch(Exception e) {
        e.printStackTrace();
    } finally {
        if (fis != null) {
            try {
                fis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (os != null) {
            try {
                os.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

或者利用 Spring MVC 的 `ResponseEntity` 类，处理请求方法返回一个 `ResponseEntity` 对象
``` java
@RequestMapping(value = "...", method = RequestMethod.GET)
public ResponseEntity<InputStreamResource> downloadFile() throws IOException {
    
    // 假设 filePath 为本地文件路径
    FileSystemResource file = new FileSystemResource(filePath);
    HttpHeaders headers = new HttpHeaders();
    headers.add("Cache-Control", "no-cache, no-store, must-revalidate");
    headers.add("Content-Disposition", String.format("attachment; filename=\"%s\"", file.getFilename()));
    headers.add("Pragma", "no-cache");
    headers.add("Expires", "0");

    return ResponseEntity
            .ok()
            .headers(headers)
            .contentLength(file.contentLength())
            .contentType(MediaType.parseMediaType("application/octet-stream"))
            .body(new InputStreamResource(file.getInputStream()));
}
```
