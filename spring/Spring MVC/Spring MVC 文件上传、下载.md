## 文件上传
```
<form action="upload" enctype="multipart/form-data" method="post">
  <input type="file" name="file"/><br/>
  <input type="submit" value="upload"/>
</form>
```

### Spring 配置
Spring MVC 专门提供了 `CommonsMultipartResolver` 组件用于文件上传

使用 xml 配置方式为：
```
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
```
@Bean
public MultipartResolver multipartResolver() {
  CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver();
  multipartResolver.setMaxUploadSize(1000000);
  return multipartResolver;
}
```

### 单文件上传
```
// 将普通的 HttpServletRequest 对象转换为 MultipartHttpServletRequest 对象
MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest) request;
// 通过 getFile() 方法获取从表单中上传过来的 file，参数 "file" 必须与 form 表单中 input 标签的 name 是一致的
CommonsMultipartFile file = (CommonsMultipartFile) multipartRequest.getFile("file");

FileCopyUtils.copy(file.getBytes(), uploadFile);
```
或者直接作为参数
```
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
```
MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest) request;

Map<String, MultipartFile> fileMap = multipartRequest.getFileMap();

for(Map.Entry<String, MultipartFile> entity:fileMap.entrySet()) {
    MultipartFile mf = entity.getValue();
    fileName = mf.getOriginalFilename();
    File uploadFile = new File(path + fileName);
    FileCopyUtils.copy(mf.getBytes(), uploadFile);
}
```

## 文件下载
response 设置编码格式( ContentType )为 text/html;charset=utf-8

response 设置 header 中 Content-disposition 属性值为 attachment; filename=文件名

response 设置 header 中 Content-Length 属性，值为文件的大小
```
response.setContentType("text/html;charset=utf-8");
request.setCharacterEncoding("UTF-8");
BufferedInputStream bis = null;
BufferedOutputStream bos = null;

String ctxPath = request.getSession().getServletContext().getRealPath("/") + "\\" + "image\\";
String downloadPath = ctxPath + fileName;

try {
    long fileLength = new File(downloadPath).length();
    response.setContentType("application/x-msdownload;");
    response.setHeader("content-disposition", "attchment;filename=" + new String(fileName.getByte("utf-8"), "ISO8859-1"));
    bis = new BufferedInputStream(new FileInputStream(downloadPath));
    bos = new BufferedOutputStream(response.getOutputStream());
    byte[] buff = new byte[2048];
    int bytesRead;
    while(-1 != (bytesRead = bis.read(buff, 0, buff.length))) {
        bos.write(buff, 0, bytesRead);
    }

} catch(Exception e) {
    e.printStackTrace();
} finally {
    if (bis != null) {
        bis.close();
    }
    if (bos != null) {
        bos.close();
    }
}
return null;
```
