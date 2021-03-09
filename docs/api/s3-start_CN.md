# 基于 MinIO 的 MEFS Go SDK

MinIO Go Client SDK 提供了简单的 API 来访问任何与 Amazon S3 兼容的对象存储服务。MEFS 提供了基于 MinIO 的 SDK。

**支持的云存储:**

- AWS Signature Version 4

  - Amazon S3
  - MinIO

- AWS Signature Version 2
  - Google Cloud Storage (兼容模式)
  - Openstack Swift + Swift3 middleware
  - Ceph Object Gateway
  - Riak CS

本文我们将学习如何安装 MinIO client SDK，连接到 MinIO，并提供一下文件上传的示例。对于完整的 API 以及示例，请参考[s3-sdk-go](/docs/api/s3-sdk-go_CN.md)。

本文假设你已经有 [Go 开发环境](https://golang.org/doc/install) 以及 [MEFS](/get-started/User-usage_CN.md)、[MEFS 账号](/get-started/User-usage_CN.md)

## 从 Github 下载

```sh
go get -u github.com/minio/minio-go
```

## 运行 MEFS

mefs-user 的 lfs 服务启动， 以及 gateway 启动

## 初始化 MinIO Client

MinIO client 需要以下 4 个参数来连接与 Amazon S3 兼容的对象存储。

| 参数            | 描述                 |
| :-------------- | :------------------- |
| endpoint        | MEFS 服务的 endpoint |
| accessKeyID     | MEFS 账户的地址      |
| secretAccessKey | MEFS 账户密码        |
| secure          | true 代表使用 HTTPS  |

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

## API 文档

完整的 API 文档在这里。

- [完整 API 文档](/docs/api/s3-sdk-go_CN.md)

## 了解更多

- [完整文档](/README.md)
