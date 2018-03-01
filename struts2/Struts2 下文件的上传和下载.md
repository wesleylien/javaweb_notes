## Struts2 下文件的上传和下载
### 上传
* 表单需要设置 `enctype="multipart/form-data"`
* 包依赖：`commons-io`、`commons-fileupload`
* Action 实现类
    ```
    public class UploadAction extends ActionSupport {

        private String savePath;

        private File upload;
        // 上传文件的文件类型
        private String uploadContentType;
        // 上传文件的文件名
        private String uploadFileName;
        // 省略 getter 和 setter
        ...
        private String getSavePath() throws Exception {
            return ServletActionContext.getServletContext().getRealPath("/WEB-INF/" + savePath);
        }

        @Override
        public String execute() throws Exception {
            FileOutputStream fos = new FileOutputStream(getSavePath() + "\\" + getUploadFileName());
            FileInputStream fis = new FileInputStream(getUpload());
            byte[] buffer = new byte[1024];
            int len = 0;
            while((len = fis.read(buffer)) > 0) {
                fos.write(buffer, 0, len);
            }
            return SUCCESS;
        }
    }
    ```
* 拦截器实现上传文件过滤
    * Struts2 中文件上传的拦截器是 `fileUpload`，只需要在 action 中配置该拦截器
        ```
        <action name="uploadPro" class="xxx">
            <interceptor-ref name="fileUpload">
                <!-- 指定允许上传的文件类型，用 , 隔开 -->
                <param name="allowedTypes">image.png,image/gif,image.jpeg</param>
                <!-- 指定文件大小，单位是 byte -->
                <param name="maximumSize">2000</param>
            </interceptor-ref>
            ...
        </action>
        ```

### 下载
* 结果类型 `stream` 用于支持文件的下载
* 指定 `stream` 结果类型，需要指定一个 `inputName` 参数，参数指定了一个输入流
    ```
    <result name="success" type="stream">
        <!-- 指定下载文件的文件类型 -->
        <param name="contentType">image/jpg</param>
        <!-- 指定下载文件的入口输入流 -->
        <param name="inputName">targetFile</param>
        <!-- 指定下载的文件名 -->
        <param name="contentDisposition">filename="hahaha.jpg"</param>
        <!-- 指定下载文件时的缓冲大小 -->
        <param name="bufferSize">4096</param>
    </result>
    ```
* 文件下载 Action 的处理方法需返回一个 InputStream
    ```
    public class DownloadAction extends ActionSupport {
        // 方法名为 getTargetFile，则 stream 结果类型的 inputName 参数值为 targetFile
        public InputStream getTargetFile() throws Exception {
            // ServletContext 提供 getResourceAsStream() 方法
            return ServletActionContext.getServletContext().getResourceAsStream(inputPath);
        }
    }
    ```
