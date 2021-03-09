# MEFS Go SDK based on MinIO

The MinIO Go Client SDK provides simple APIs to access any Amazon S3 compatible object storage.MEFS provides the same SDK based on MinIO

**Supported cloud storage:**

- AWS Signature Version 4

  - Amazon S3
  - MinIO

- AWS Signature Version 2
  - Google Cloud Storage (compatibility mode)
  - Openstack Swift + Swift3 middleware
  - Ceph Object Gateway
  - Riak CS

This guide will show you how to install the MinIO client SDK, connect to MinIO, and provide a walkthrough for a simple file uploader. For a complete list of APIs and examples, please take a look at the [s3-sdk-go](/docs/api/s3-sdk-go.md).

This document assumes thad you have a working [Go development environment](https://golang.org/doc/install) and [MEFS](/get-started/User-usage.md)、[MEFS account](/get-started/User-usage.md)

## Download from Github

```sh
go get -u github.com/minio/minio-go
```

## 要求

## Initialize MinIO Client

MinIO client requires the following four parameters specified to connect to an Amazon S3 compatible object storage.

| Parameter       | Description                                               |
| :-------------- | :-------------------------------------------------------- |
| endpoint        | endpoint to MEFS service                                  |
| accessKeyID     | MEFS account's address                                    |
| secretAccessKey | MEFS account's password                                   |
| secure          | Set this value to 'true' to enable secure (HTTPS) access. |

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

## API Reference

The full API Reference is available here.

- [Complete API Reference](/docs/api/s3-sdk-go.md)

## Learn more

- [Full doc](/README.md)
