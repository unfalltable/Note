## Minio

### 文件上传

```java
private final MinioClient minioClient;
public UploadFileResultDTO uploadFile(byte[] bytes, Long userId, UploadFileParam param, String folderName){
    //上传到minio
    PutObjectArgs putObjectArgs = PutObjectArgs.builder()
        .bucket("桶名")
        .object("文件名")
        .stream("文件输入流, 文件大小, 分片大小（-1最小5m, 最大10000）")
        .contentType("content-type") //资源d
        .build();
    minioClient.putObject(putObjectArgs);
    //保存到数据库
    //可以将文件流转换为md5存入数据库
    	//先查再插入
}
//上传文件的基本信息
class UploadFileParam{
    //文件名
    private String fileName;
    //content-type
    private String contentType;
    //文件类型
    private String fileType;
    //标签
    private String tags;
    //上传人
    private String username;
    //备注
    private String remark;
}
```

### 视频上传

```java
//检查该文件在minio上是否存在
public R checkFile(String fileMd5, int index){
    //查数据库，数据存在再查minio
    //查minio
    GetObjectArgs object = GetObjectArgs.builder()....
    minioClient.getObject(object);
}
//判断分块是否存在
public R checkFile(String fileMd5){
    //截取fileMd5前两位，拼接成文件路径
    //查minio
    GetObjectArgs object = GetObjectArgs.builder()....
    minioClient.getObject(object); 
}
//开始上传
public R uploadchunk(String fileMd5, int index, String localFilePath){
    
}
//合并
```

## 阿里云