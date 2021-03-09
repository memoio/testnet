# MinIO Go Client API Reference 

## Initialize MinIO Client object.

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


| Bucket operations                                 | Object operations                                   | Encrypted Object operations                 | Presigned operations                          | Bucket Policy/Notification Operations                         | Client custom settings                                |
| :---                                              | :---                                                | :---                                        | :---                                          | :---                                                          | :---                                                  |
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
## 1. Constructor
<a name="MinIO"></a>

### New(endpoint, accessKeyID, secretAccessKey string, ssl bool) (*Client, error)
Initializes a new client object.

__Parameters__

|Param   |Type   |Description   |
|:---|:---| :---|
|`endpoint`   | _string_  |MEFS endpoint   |
|`accessKeyID`  |_string_   |Address for MEFS account|
|`secretAccessKey`  | _string_  |Secret key for MEFS account |
|`ssl`   | _bool_  | If 'true' API requests will be secure (HTTPS), and insecure (HTTP) otherwise  |

## 2. Bucket operations

<a name="MakeBucket"></a>
### MakeBucket(bucketName, location string) error
Creates a new bucket.

__Parameters__

| Param  | Type  | Description  |
|---|---|---|
|`bucketName`  | _string_  | Name of the bucket |
| `location`  |  _string_ | ** Option|



__Example__


```go
err = minioClient.MakeBucket("mybucket", "us-east-1")
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println("Successfully created mybucket.")
```

<a name="ListBuckets"></a>
### ListBuckets() ([]BucketInfo, error)
Lists all buckets.

| Param  | Type  | Description  |
|---|---|---|
|`bucketList`  | _[]minio.BucketInfo_  | Lists of all buckets |


__minio.BucketInfo__

| Field  | Type  | Description  |
|---|---|---|
|`bucket.Name`  | _string_  | Name of the bucket |
|`bucket.CreationDate`  | _time.Time_  | Date of bucket creation |


__Example__


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
Checks if a bucket exists.

__Parameters__


|Param   |Type   |Description   |
|:---|:---| :---|
|`bucketName`  | _string_  |Name of the bucket |


__Return Values__

|Param   |Type   |Description   |
|:---|:---| :---|
|`found`  | _bool_ | Indicates whether bucket exists or not  |
|`err` | _error_  | Standard Error  |


__Example__


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
Removes a bucket, bucket should be empty to be successfully removed.

__Parameters__


|Param   |Type   |Description   |
|:---|:---| :---|
|`bucketName`  | _string_  |Name of the bucket   |

__Example__


```go
err = minioClient.RemoveBucket("mybucket")
if err != nil {
    fmt.Println(err)
    return
}
```

<a name="ListObjects"></a>
### ListObjects(bucketName, prefix string, recursive bool, doneCh chan struct{}) <-chan ObjectInfo
Lists objects in a bucket.

__Parameters__


|Param   |Type   |Description   |
|:---|:---| :---|
|`bucketName` | _string_  |Name of the bucket   |
|`objectPrefix` |_string_   | Prefix of objects to be listed |
|`recursive`  | _bool_  |`true` indicates recursive style listing and `false` indicates directory style listing delimited by '/'.  |
|`doneCh`  | _chan struct{}_ | A message on this channel ends the ListObjects iterator.  |


__Return Value__

|Param   |Type   |Description   |
|:---|:---| :---|
|`objectInfo`  | _chan minio.ObjectInfo_ |Read channel for all objects in the bucket, the object is of the format listed below: |

__minio.ObjectInfo__

|Field   |Type   |Description   |
|:---|:---| :---|
|`objectInfo.Key`  | _string_ |Name of the object |
|`objectInfo.Size`  | _int64_ |Size of the object |
|`objectInfo.ETag`  | _string_ |MD5 checksum of the object |
|`objectInfo.LastModified`  | _time.Time_ |Time when object was last modified |


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
Lists objects in a bucket using the recommended listing API v2

__Parameters__


|Param   |Type   |Description   |
|:---|:---| :---|
|`bucketName`  | _string_  |Name of the bucket |
| `objectPrefix` |_string_   | Prefix of objects to be listed |
| `recursive`  | _bool_  |`true` indicates recursive style listing and `false` indicates directory style listing delimited by '/'.  |
|`doneCh`  | _chan struct{}_ | A message on this channel ends the ListObjectsV2 iterator.  |


__Return Value__

|Param   |Type   |Description   |
|:---|:---| :---|
|`objectInfo`  | _chan minio.ObjectInfo_ |Read channel for all the objects in the bucket, the object is of the format listed below: |


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
## 3. Object operations

<a name="GetObject"></a>
### GetObject(bucketName, objectName string, opts GetObjectOptions) (*Object, error)
Returns a stream of the object data. Most of the common errors occur when reading the stream.


__Parameters__


|Param   |Type   |Description   |
|:---|:---| :---|
|`bucketName`  | _string_  |Name of the bucket  |
|`objectName` | _string_  |Name of the object  |
|`opts` | _minio.GetObjectOptions_ | Options for GET requests specifying additional options like encryption, If-Match |


__minio.GetObjectOptions__

|Field | Type | Description |
|:---|:---|:---|
| `opts.ServerSideEncryption` | _encrypt.ServerSide_ | Interface provided by `encrypt` package to specify server-side-encryption. (For more information see https://godoc.org/github.com/minio/minio-go/v6) |

__Return Value__


|Param   |Type   |Description   |
|:---|:---| :---|
|`object`  | _*minio.Object_ |_minio.Object_ represents object reader. It implements io.Reader, io.Seeker, io.ReaderAt and io.Closer interfaces. |


__Example__


```go
object, err := minioClient.GetObject("mybucket", "myobject", minio.GetObjectOptions{})
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
Downloads and saves the object as a file in the local filesystem.

__Parameters__


|Param   |Type   |Description   |
|:---|:---| :---|
|`bucketName`  | _string_  |Name of the bucket |
|`objectName` | _string_  |Name of the object  |
|`filePath` | _string_  |Path to download object to |
|`opts` | _minio.GetObjectOptions_ | Options for GET requests specifying additional options like encryption, If-Match |


__Example__


```go
err = minioClient.FGetObject("mybucket", "myobject", "/tmp/myobject", minio.GetObjectOptions{})
if err != nil {
    fmt.Println(err)
    return
}
```
<a name="GetObjectWithContext"></a>
### GetObjectWithContext(ctx context.Context, bucketName, objectName string, opts GetObjectOptions) (*Object, error)
Identical to GetObject operation, but accepts a context for request cancellation.

__Parameters__


|Param   |Type   |Description   |
|:---|:---| :---|
|`ctx`  | _context.Context_  |Request context  |
|`bucketName`  | _string_  |Name of the bucket  |
|`objectName` | _string_  |Name of the object  |
|`opts` | _minio.GetObjectOptions_ | Options for GET requests specifying additional options like encryption, If-Match |


__Return Value__


|Param   |Type   |Description   |
|:---|:---| :---|
|`object`  | _*minio.Object_ |_minio.Object_ represents object reader. It implements io.Reader, io.Seeker, io.ReaderAt and io.Closer interfaces. |


__Example__


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
Identical to FGetObject operation, but allows request cancellation.

__Parameters__


|Param   |Type   |Description   |
|:---|:---| :---|
|`ctx`  | _context.Context_  |Request context |
|`bucketName`  | _string_  |Name of the bucket |
|`objectName` | _string_  |Name of the object  |
|`filePath` | _string_  |Path to download object to |
|`opts` | _minio.GetObjectOptions_ | Options for GET requests specifying additional options like encryption, If-Match |


__Example__


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
Uploads objects that are less than 128MiB in a single PUT operation. For objects that are greater than 128MiB in size, PutObject seamlessly uploads the object as parts of 128MiB or more depending on the actual file size. The max upload size for an object is 5TB.

__Parameters__


|Param   |Type   |Description   |
|:---|:---| :---|
|`bucketName`  | _string_  |Name of the bucket  |
|`objectName` | _string_  |Name of the object   |
|`reader` | _io.Reader_  |Any Go type that implements io.Reader |
|`objectSize`| _int64_ |Size of the object being uploaded. Pass -1 if stream size is unknown |
|`opts` | _minio.PutObjectOptions_  | Allows user to set optional custom metadata, content headers, encryption keys and number of threads for multipart upload operation. |

__minio.PutObjectOptions__

|Field | Type | Description |
|:--- |:--- | :--- |
| `opts.UserMetadata` | _map[string]string_ | Map of user metadata|
| `opts.Progress` | _io.Reader_ | Reader to fetch progress of an upload |
| `opts.ContentType` | _string_ | Content type of object, e.g "application/text" |
| `opts.ContentEncoding` | _string_ | Content encoding of object, e.g "gzip" |
| `opts.ContentDisposition` | _string_ | Content disposition of object, "inline" |
| `opts.ContentLanguage` | _string_ | Content language of object, e.g "French" |
| `opts.CacheControl` | _string_ | Used to specify directives for caching mechanisms in both requests and responses e.g "max-age=600"|
| `opts.Mode` | _*minio.RetentionMode_ | Retention mode to be set, e.g "COMPLIANCE" |
| `opts.RetainUntilDate` | _*time.Time_ | Time until which the retention applied is valid|
| `opts.ServerSideEncryption` | _encrypt.ServerSide_ | Interface provided by `encrypt` package to specify server-side-encryption. (For more information see https://godoc.org/github.com/minio/minio-go/v6) |
| `opts.StorageClass` | _string_ | Specify storage class for the object. Supported values for MinIO server are `REDUCED_REDUNDANCY` and `STANDARD` |
| `opts.WebsiteRedirectLocation` | _string_ | Specify a redirect for the object, to another object in the same bucket or to a external URL. |

__Example__


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

API methods PutObjectWithSize, PutObjectWithMetadata, PutObjectStreaming, and PutObjectWithProgress available in minio-go SDK release v3.0.3 are replaced by the new PutObject call variant that accepts a pointer to PutObjectOptions struct.

<a name="PutObjectWithContext"></a>
### PutObjectWithContext(ctx context.Context, bucketName, objectName string, reader io.Reader, objectSize int64, opts PutObjectOptions) (n int, err error)
Identical to PutObject operation, but allows request cancellation.

__Parameters__


|Param   |Type   |Description   |
|:---|:---| :---|
|`ctx`  | _context.Context_  |Request context |
|`bucketName`  | _string_  |Name of the bucket  |
|`objectName` | _string_  |Name of the object   |
|`reader` | _io.Reader_  |Any Go type that implements io.Reader |
|`objectSize`| _int64_ | size of the object being uploaded. Pass -1 if stream size is unknown |
|`opts` | _minio.PutObjectOptions_  |Pointer to struct that allows user to set optional custom metadata, content-type, content-encoding, content-disposition, content-language and cache-control headers, pass encryption module for encrypting objects, and optionally configure number of threads for multipart put operation. |


__Example__


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

<a name="FPutObject"></a>
### FPutObject(bucketName, objectName, filePath, opts PutObjectOptions) (length int64, err error)
Uploads contents from a file to objectName.

FPutObject uploads objects that are less than 128MiB in a single PUT operation. For objects that are greater than the 128MiB in size, FPutObject seamlessly uploads the object in chunks of 128MiB or more depending on the actual file size. The max upload size for an object is 5TB.

__Parameters__


|Param   |Type   |Description   |
|:---|:---| :---|
|`bucketName`  | _string_  |Name of the bucket  |
|`objectName` | _string_  |Name of the object |
|`filePath` | _string_  |Path to file to be uploaded |
|`opts` | _minio.PutObjectOptions_  |Pointer to struct that allows user to set optional custom metadata, content-type, content-encoding, content-disposition, content-language and cache-control headers, pass encryption module for encrypting objects, and optionally configure number of threads for multipart put operation.  |


__Example__


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
Identical to FPutObject operation, but allows request cancellation.

__Parameters__


|Param   |Type   |Description   |
|:---|:---| :---|
|`ctx`  | _context.Context_  |Request context  |
|`bucketName`  | _string_  |Name of the bucket  |
|`objectName` | _string_  |Name of the object |
|`filePath` | _string_  |Path to file to be uploaded |
|`opts` | _minio.PutObjectOptions_  |Pointer to struct that allows user to set optional custom metadata, content-type, content-encoding,content-disposition and cache-control headers, pass encryption module for encrypting objects, and optionally configure number of threads for multipart put operation. |

__Example__


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
Fetch metadata of an object.

__Parameters__


|Param   |Type   |Description   |
|:---|:---| :---|
|`bucketName`  | _string_  |Name of the bucket  |
|`objectName` | _string_  |Name of the object   |
|`opts` | _minio.StatObjectOptions_ | Options for GET info/stat requests specifying additional options like encryption, If-Match |


__Return Value__

|Param   |Type   |Description   |
|:---|:---| :---|
|`objInfo`  | _minio.ObjectInfo_  |Object stat information |


__minio.ObjectInfo__

|Field   |Type   |Description   |
|:---|:---| :---|
|`objInfo.LastModified`  | _time.Time_  |Time when object was last modified |
|`objInfo.ETag` | _string_ |MD5 checksum of the object|
|`objInfo.ContentType` | _string_ |Content type of the object|
|`objInfo.Size` | _int64_ |Size of the object|


__Example__


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
Removes an object.

__Parameters__


|Param   |Type   |Description   |
|:---|:---| :---|
|`bucketName`  | _string_  |Name of the bucket  |
|`objectName` | _string_  |Name of the object |


```go
err = minioClient.RemoveObject("mybucket", "myobject")
if err != nil {
    fmt.Println(err)
    return
}
```

<a name="RemoveObjects"></a>
### RemoveObjects(bucketName string, objectsCh chan string) (errorCh <-chan RemoveObjectError)
Removes a list of objects obtained from an input channel. The call sends a delete request to the server up to 1000 objects at a time. The errors observed are sent over the error channel.

__Parameters__

|Param   |Type   |Description   |
|:---|:---| :---|
|`bucketName`  | _string_  |Name of the bucket  |
|`objectsCh` | _chan string_  | Channel of objects to be removed   |


__Return Values__

|Param   |Type   |Description   |
|:---|:---| :---|
|`errorCh` | _<-chan minio.RemoveObjectError_  | Receive-only channel of errors observed during deletion.  |


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