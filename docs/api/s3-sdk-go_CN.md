# MinIO Go Client API文档

## 初使化MinIO Client对象。

##  MinIO

```go
package main

import (
    "fmt"

    "github.com/minio/minio-go"
)

func main() {
    endpoint := "127.0.0.1:5080"
    accessKeyID := "0x6A6AEB9840C8a42A9d8Ff55cf2c213F9b812ED0A"
    secretAccessKey := "123456789"
    useSSL := false

    _, err := minio.New(endpoint, accessKeyID, secretAccessKey, useSSL)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println("OK")
}
```


| 操作存储桶                                 | 操作对象                                  |
| :---                                     | :---                                     |
| [`MakeBucket`](#MakeBucket)                       | [`GetObject`](#GetObject)                           |
| [`ListBuckets`](#ListBuckets)                     | [`PutObject`](#PutObject)                           |
| [`BucketExists`](#BucketExists)                   |                                                     |
| [`RemoveBucket`](#RemoveBucket)                   | [`StatObject`](#StatObject)                         |
| [`ListObjects`](#ListObjects)                     | [`RemoveObject`](#RemoveObject)                     |
| [`ListObjectsV2`](#ListObjectsV2)                 | [`RemoveObjects`](#RemoveObjects)                   |
|                                                   | [`FPutObject`](#FPutObject)                         |
|                                                   | [`FGetObject`](#FGetObject)                         |
|                                                   | [`PutObjectWithContext`](#PutObjectWithContext)     |
|                                                   | [`GetObjectWithContext`](#GetObjectWithContext)     |
|                                                   | [`FPutObjectWithContext`](#FPutObjectWithContext)   |
|                                                   | [`FGetObjectWithContext`](#FGetObjectWithContext)   |
## 1. 构造函数
<a name="MinIO"></a>

### New(endpoint, accessKeyID, secretAccessKey string, ssl bool) (*Client, error)
初使化一个新的client对象。

__参数__

|参数   | 类型   |描述   |
|:---|:---| :---|
|`endpoint`   | _string_  |MEFS服务endpoint   |
|`accessKeyID`  |_string_   |MEFS账户的地址 |
|`secretAccessKey`  | _string_  |MEFS账户密码 |
|`ssl`   | _bool_  |true代表使用HTTPS |

## 2. 操作存储桶

<a name="MakeBucket"></a>
### MakeBucket(bucketName, location string) error
创建一个存储桶。

__参数__

| 参数  | 类型  | 描述  |
|---|---|---|
|`bucketName`  | _string_  | 存储桶名称 |
| `location`  |  _string_ | **选项字段|


__示例__


```go
err = minioClient.MakeBucket("mybucket", "")
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println("Successfully created mybucket.")
```

<a name="ListBuckets"></a>
### ListBuckets() ([]BucketInfo, error)
列出所有的存储桶。

| 参数  | 类型   | 描述  |
|---|---|---|
|`bucketList`  | _[]minio.BucketInfo_  | 所有存储桶的list。 |


__minio.BucketInfo__

| 参数  | 类型   | 描述  |
|---|---|---|
|`bucket.Name`  | _string_  | 存储桶名称 |
|`bucket.CreationDate`  | _time.Time_  | 存储桶的创建时间 |


__示例__


```go
buckets, err := minioClient.ListBuckets()
if err != nil {
    fmt.Println(err)
    return
}
for _, bucket := range buckets {
    fmt.Println(bucket)
}
```

<a name="BucketExists"></a>
### BucketExists(bucketName string) (found bool, err error)
检查存储桶是否存在。

__参数__


|参数   |类型   |描述   |
|:---|:---| :---|
|`bucketName`  | _string_  |存储桶名称 |


__返回值__

|参数   |类型   |描述   |
|:---|:---| :---|
|`found`  | _bool_ | 存储桶是否存在  |
|`err` | _error_  | 标准Error  |


__示例__


```go
found, err := minioClient.BucketExists("mybucket")
if err != nil {
    fmt.Println(err)
    return
}
if found {
    fmt.Println("Bucket found")
}
```

<a name="RemoveBucket"></a>
### RemoveBucket(bucketName string) error
删除一个存储桶，存储桶必须为空才能被成功删除。

__参数__


|参数   |类型   |描述   |
|:---|:---| :---|
|`bucketName`  | _string_  |存储桶名称   |

__示例__


```go
err = minioClient.RemoveBucket("mybucket")
if err != nil {
    fmt.Println(err)
    return
}
```

<a name="ListObjects"></a>
### ListObjects(bucketName, prefix string, recursive bool, doneCh chan struct{}) <-chan ObjectInfo
列举存储桶里的对象。

__参数__


|参数   |类型   |描述   |
|:---|:---| :---|
|`bucketName` | _string_  |存储桶名称   |
|`objectPrefix` |_string_   | 要列举的对象前缀 |
|`recursive`  | _bool_  |`true`代表递归查找，`false`代表类似文件夹查找，以'/'分隔，不查子文件夹。  |
|`doneCh`  | _chan struct{}_ | 在该channel上结束ListObjects iterator的一个message。 |


__返回值__

|参数   |类型   |描述   |
|:---|:---| :---|
|`objectInfo`  | _chan minio.ObjectInfo_ |存储桶中所有对象的read channel，对象的格式如下： |

__minio.ObjectInfo__

|属性   |类型   |描述   |
|:---|:---| :---|
|`objectInfo.Key`  | _string_ |对象的名称 |
|`objectInfo.Size`  | _int64_ |对象的大小 |
|`objectInfo.ETag`  | _string_ |对象的MD5校验码 |
|`objectInfo.LastModified`  | _time.Time_ |对象的最后修改时间 |


```go
// Create a done channel to control 'ListObjects' go routine.
doneCh := make(chan struct{})

// Indicate to our routine to exit cleanly upon return.
defer close(doneCh)

isRecursive := true
objectCh := minioClient.ListObjects("mybucket", "myprefix", isRecursive, doneCh)
for object := range objectCh {
    if object.Err != nil {
        fmt.Println(object.Err)
        return
    }
    fmt.Println(object)
}
```


<a name="ListObjectsV2"></a>
### ListObjectsV2(bucketName, prefix string, recursive bool, doneCh chan struct{}) <-chan ObjectInfo
使用listing API v2版本列举存储桶中的对象。

__参数__


|参数   |类型   |描述   |
|:---|:---| :---|
|`bucketName`  | _string_  |存储桶名称 |
| `objectPrefix` |_string_   | 要列举的对象前缀 |
| `recursive`  | _bool_  |`true`代表递归查找，`false`代表类似文件夹查找，以'/'分隔，不查子文件夹。  |
|`doneCh`  | _chan struct{}_ | 在该channel上结束ListObjects iterator的一个message。  |


__返回值__

|参数   |类型   |描述   |
|:---|:---| :---|
|`objectInfo`  | _chan minio.ObjectInfo_ |存储桶中所有对象的read channel |


```go
// Create a done channel to control 'ListObjectsV2' go routine.
doneCh := make(chan struct{})

// Indicate to our routine to exit cleanly upon return.
defer close(doneCh)

isRecursive := true
objectCh := minioClient.ListObjectsV2("mybucket", "myprefix", isRecursive, doneCh)
for object := range objectCh {
    if object.Err != nil {
        fmt.Println(object.Err)
        return
    }
    fmt.Println(object)
}
```

## 3. 操作对象

<a name="GetObject"></a>
### GetObject(bucketName, objectName string, opts GetObjectOptions) (*Object, error)
返回对象数据的流，error是读流时经常抛的那些错。


__参数__


|参数   |类型   |描述   |
|:---|:---| :---|
|`bucketName`  | _string_  |存储桶名称  |
|`objectName` | _string_  |对象的名称  |
|`opts` | _minio.GetObjectOptions_ | GET请求的一些额外参数，像encryption，If-Match |


__minio.GetObjectOptions__

|参数 | 类型 | 描述 |
|:---|:---|:---|
| `opts.Materials` | _encrypt.Materials_ | `encrypt`包提供的对流加密的接口，(更多信息，请看https://godoc.org/github.com/minio/minio-go/v6) |

__返回值__


|参数   |类型   |描述   |
|:---|:---| :---|
|`object`  | _*minio.Object_ |_minio.Object_代表了一个object reader。它实现了io.Reader, io.Seeker, io.ReaderAt and io.Closer接口。 |


__示例__


```go
object, err := minioClient.GetObject("testbucket2", "myobject", minio.GetObjectOptions{})
if err != nil {
    fmt.Println(err)
    return
}
localFile, err := os.Create("/tmp/local-file.jpg")
if err != nil {
    fmt.Println(err)
    return
}
if _, err = io.Copy(localFile, object); err != nil {
    fmt.Println(err)
    return
}
```

<a name="FGetObject"></a>
### FGetObject(bucketName, objectName, filePath string, opts GetObjectOptions) error
下载并将文件保存到本地文件系统。

__参数__


|参数   |类型   |描述   |
|:---|:---| :---|
|`bucketName`  | _string_  |存储桶名称 |
|`objectName` | _string_  |对象的名称  |
|`filePath` | _string_  |下载后保存的路径 |
|`opts` | _minio.GetObjectOptions_ | GET请求的一些额外参数，像encryption，If-Match |


__示例__


```go
err = minioClient.FGetObject("mybucket", "myobject", "/tmp/myobject", minio.GetObjectOptions{})
if err != nil {
    fmt.Println(err)
    return
}
```
<a name="GetObjectWithContext"></a>
### GetObjectWithContext(ctx context.Context, bucketName, objectName string, opts GetObjectOptions) (*Object, error)
和GetObject操作是一样的，不过传入了取消请求的context。

__参数__


|参数   |类型   |描述   |
|:---|:---| :---|
|`ctx`  | _context.Context_  |请求上下文（Request context） |
|`bucketName`  | _string_  |存储桶名称  |
|`objectName` | _string_  |对象的名称  |
|`opts` | _minio.GetObjectOptions_ |  GET请求的一些额外参数，像encryption，If-Match |


__返回值__


|参数   |类型   |描述   |
|:---|:---| :---|
|`object`  | _*minio.Object_ |_minio.Object_代表了一个object reader。它实现了io.Reader, io.Seeker, io.ReaderAt and io.Closer接口。 |


__示例__


```go
ctx, cancel := context.WithTimeout(context.Background(), 100 * time.Second)
defer cancel()

object, err := minioClient.GetObjectWithContext(ctx, "mybucket", "myobject", minio.GetObjectOptions{})
if err != nil {
    fmt.Println(err)
    return
}

localFile, err := os.Create("/tmp/local-file.jpg")
if err != nil {
    fmt.Println(err)
    return
}

if _, err = io.Copy(localFile, object); err != nil {
    fmt.Println(err)
    return
}
```

<a name="FGetObjectWithContext"></a>
### FGetObjectWithContext(ctx context.Context, bucketName, objectName, filePath string, opts GetObjectOptions) error
和FGetObject操作是一样的，不过允许取消请求。

__参数__


|参数   |类型   |描述   |
|:---|:---| :---|
|`ctx`  | _context.Context_  |请求上下文 |
|`bucketName`  | _string_  |存储桶名称 |
|`objectName` | _string_  |对象的名称  |
|`filePath` | _string_  |下载后保存的路径 |
|`opts` | _minio.GetObjectOptions_ | GET请求的一些额外参数，像encryption，If-Match  |


__示例__


```go
ctx, cancel := context.WithTimeout(context.Background(), 100 * time.Second)
defer cancel()

err = minioClient.FGetObjectWithContext(ctx, "mybucket", "myobject", "/tmp/myobject", minio.GetObjectOptions{})
if err != nil {
    fmt.Println(err)
    return
}
```

<a name="PutObject"></a>
### PutObject(bucketName, objectName string, reader io.Reader, objectSize int64,opts PutObjectOptions) (n int, err error)
当对象小于128MiB时，直接在一次PUT请求里进行上传。当大于128MiB时，根据文件的实际大小，PutObject会自动地将对象进行拆分成128MiB一块或更大一些进行上传。对象的最大大小是5TB。

__参数__


|参数   |类型   |描述   |
|:---|:---| :---|
|`bucketName`  | _string_  |存储桶名称  |
|`objectName` | _string_  |对象的名称   |
|`reader` | _io.Reader_  |任意实现了io.Reader的GO类型 |
|`objectSize`| _int64_ |上传的对象的大小，-1代表未知。 |
|`opts` | _minio.PutObjectOptions_  |  允许用户设置可选的自定义元数据，内容标题，加密密钥和用于分段上传操作的线程数量。 |

__minio.PutObjectOptions__

|属性 | 类型 | 描述 |
|:--- |:--- | :--- |
| `opts.UserMetadata` | _map[string]string_ | 用户元数据的Map|
| `opts.Progress` | _io.Reader_ | 获取上传进度的Reader |
| `opts.ContentType` | _string_ | 对象的Content type， 例如"application/text" |
| `opts.ContentEncoding` | _string_ | 对象的Content encoding，例如"gzip" |
| `opts.ContentDisposition` | _string_ | 对象的Content disposition, "inline" |
| `opts.ContentLanguage` | _string_ | 对象的Content Language,  "French" |
| `opts.CacheControl` | _string_ | 指定针对请求和响应的缓存机制，例如"max-age=600"|
| `opts.Mode` | _*minio.RetentionMode_ | 设置保留模式, e.g "COMPLIANCE" |
| `opts.RetainUntilDate` | _*time.Time_ | 保留模式的有效期|
| `opts.ServerSideEncryption` | _encrypt.ServerSide_ | `encrypt`包提供的对流加密的接口，(更多信息，请看https://godoc.org/github.com/minio/minio-go/v6) |
| `opts.StorageClass` | _string_ | 指定对象的存储类 MinIO服务器支持的值有 `REDUCED_REDUNDANCY` 和 `STANDARD` |
| `opts.WebsiteRedirectLocation` | _string_ | 指定对象重定向到同一个桶的其他的对象或者外部的URL|

__示例__


```go
file, err := os.Open("my-testfile")
if err != nil {
    fmt.Println(err)
    return
}
defer file.Close()

fileStat, err := file.Stat()
if err != nil {
    fmt.Println(err)
    return
}

n, err := minioClient.PutObject("mybucket", "myobject", file, fileStat.Size(), minio.PutObjectOptions{ContentType:"application/octet-stream"})
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println("Successfully uploaded bytes: ", n)
```

API方法在minio-go SDK版本v3.0.3中提供的PutObjectWithSize，PutObjectWithMetadata，PutObjectStreaming和PutObjectWithProgress被替换为接受指向PutObjectOptions struct的指针的新的PutObject调用变体。

<a name="PutObjectWithContext"></a>
### PutObjectWithContext(ctx context.Context, bucketName, objectName string, reader io.Reader, objectSize int64, opts PutObjectOptions) (n int, err error)
和PutObject是一样的，不过允许取消请求。

__参数__


|参数   |类型   |描述   |
|:---|:---| :---|
|`ctx`  | _context.Context_  |请求上下文 |
|`bucketName`  | _string_  |存储桶名称  |
|`objectName` | _string_  |对象的名称   |
|`reader` | _io.Reader_  |任何实现io.Reader的Go类型 |
|`objectSize`| _int64_ | 上传的对象的大小，-1代表未知 |
|`opts` | _minio.PutObjectOptions_  |允许用户设置可选的自定义元数据，content-type，content-encoding，content-disposition以及cache-control headers，传递加密模块以加密对象，并可选地设置multipart put操作的线程数量。|


__示例__


```go
ctx, cancel := context.WithTimeout(context.Background(), 10 * time.Second)
defer cancel()

file, err := os.Open("my-testfile")
if err != nil {
    fmt.Println(err)
    return
}
defer file.Close()

fileStat, err := file.Stat()
if err != nil {
    fmt.Println(err)
    return
}

n, err := minioClient.PutObjectWithContext(ctx, "my-bucketname", "my-objectname", file, fileStat.Size(), minio.PutObjectOptions{
	ContentType: "application/octet-stream",
})
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println("Successfully uploaded bytes: ", n)
```

<a name="CopyObject"></a>
### FPutObject(bucketName, objectName, filePath, opts PutObjectOptions) (length int64, err error)
将filePath对应的文件内容上传到一个对象中。

当对象小于128MiB时，FPutObject直接在一次PUT请求里进行上传。当大于128MiB时，根据文件的实际大小，FPutObject会自动地将对象进行拆分成128MiB一块或更大一些进行上传。对象的最大大小是5TB。

__参数__


|参数   |类型   |描述   |
|:---|:---| :---|
|`bucketName`  | _string_  |存储桶名称  |
|`objectName` | _string_  |对象的名称 |
|`filePath` | _string_  |要上传的文件的路径 |
|`opts` | _minio.PutObjectOptions_  |允许用户设置可选的自定义元数据，content-type，content-encoding，content-disposition以及cache-control headers，传递加密模块以加密对象，并可选地设置multipart put操作的线程数量。 |


__示例__


```go
n, err := minioClient.FPutObject("my-bucketname", "my-objectname", "my-filename.csv", minio.PutObjectOptions{
	ContentType: "application/csv",
});
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println("Successfully uploaded bytes: ", n)
```

<a name="FPutObjectWithContext"></a>
### FPutObjectWithContext(ctx context.Context, bucketName, objectName, filePath, opts PutObjectOptions) (length int64, err error)
和FPutObject操作是一样的，不过允许取消请求。

__参数__


|参数   |类型   |描述   |
|:---|:---| :---|
|`ctx`  | _context.Context_  |请求上下文  |
|`bucketName`  | _string_  |存储桶名称  |
|`objectName` | _string_  |对象的名称 |
|`filePath` | _string_  |要上传的文件的路径 |
|`opts` | _minio.PutObjectOptions_  |允许用户设置可选的自定义元数据，content-type，content-encoding，content-disposition以及cache-control headers，传递加密模块以加密对象，并可选地设置multipart put操作的线程数量。 |

__示例__


```go
ctx, cancel := context.WithTimeout(context.Background(), 100 * time.Second)
defer cancel()

n, err := minioClient.FPutObjectWithContext(ctx, "mybucket", "myobject.csv", "/tmp/otherobject.csv", minio.PutObjectOptions{ContentType:"application/csv"})
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println("Successfully uploaded bytes: ", n)
```

<a name="StatObject"></a>
### StatObject(bucketName, objectName string, opts StatObjectOptions) (ObjectInfo, error)
获取对象的元数据。

__参数__


|参数   |类型   |描述   |
|:---|:---| :---|
|`bucketName`  | _string_  |存储桶名称  |
|`objectName` | _string_  |对象的名称   |
|`opts` | _minio.StatObjectOptions_ | GET info/stat请求的一些额外参数，像encryption，If-Match |


__返回值__

|参数   |类型   |描述   |
|:---|:---| :---|
|`objInfo`  | _minio.ObjectInfo_  |对象stat信息 |


__minio.ObjectInfo__

|属性   |类型   |描述   |
|:---|:---| :---|
|`objInfo.LastModified`  | _time.Time_  |对象的最后修改时间 |
|`objInfo.ETag` | _string_ |对象的MD5校验码|
|`objInfo.ContentType` | _string_ |对象的Content type|
|`objInfo.Size` | _int64_ |对象的大小|


__示例__


```go
objInfo, err := minioClient.StatObject("mybucket", "myobject", minio.StatObjectOptions{})
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println(objInfo)
```

<a name="RemoveObject"></a>
### RemoveObject(bucketName, objectName string) error
删除一个对象。

__参数__


|参数   |类型   |描述   |
|:---|:---| :---|
|`bucketName`  | _string_  |存储桶名称  |
|`objectName` | _string_  |对象的名称 |


```go
err = minioClient.RemoveObject("mybucket", "myobject")
if err != nil {
    fmt.Println(err)
    return
}
```

<a name="RemoveObjects"></a>
### RemoveObjects(bucketName string, objectsCh chan string) (errorCh <-chan RemoveObjectError)

从一个input channel里删除一个对象集合。一次发送到服务端的删除请求最多可删除1000个对象。通过error channel返回的错误信息。

__参数__

|参数   |类型   |描述   |
|:---|:---| :---|
|`bucketName`  | _string_  |存储桶名称  |
|`objectsCh` | _chan string_  | 要删除的对象的channel   |


__返回值__

|参数   |类型   |描述   |
|:---|:---| :---|
|`errorCh` | _<-chan minio.RemoveObjectError_  | 删除时观察到的错误的Receive-only channel。 |


```go
objectsCh := make(chan string)

// Send object names that are needed to be removed to objectsCh
go func() {
	defer close(objectsCh)
	// List all objects from a bucket-name with a matching prefix.
	for object := range minioClient.ListObjects("my-bucketname", "my-prefixname", true, nil) {
		if object.Err != nil {
			log.Fatalln(object.Err)
		}
		objectsCh <- object.Key
	}
}()

for rErr := range minioClient.RemoveObjects("mybucket", objectsCh) {
    fmt.Println("Error detected during deletion: ", rErr)
}
```