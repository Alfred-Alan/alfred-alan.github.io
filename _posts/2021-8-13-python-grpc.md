---
layout: post
title: '如何在python中使用gprc'
description: '怎么搭建grpc结合python使用'
categories: [Python]
tags: []
image:  /assets/img/blog/grpc.png

related_posts:
  - 
---
- Table of Contents
{:toc .large-only}

### 一.安装 gRPC

```bash
# gRPC 的安装：
pip install grpcio

# 安装 ProtoBuf 相关的 python 依赖库：
pip install protobuf

# 安装 python grpc 的 ProtoBuf 编译工具：
pip install grpcio-tools
```

### 二.编写.proto文件
#### 1.创建`proto`
创建一个`./protos/hello_world.proto`

```protobuf
// file: '/protos/hello_world.proto'
// Copyright 2015 gRPC authors.
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//     http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

syntax = "proto3";

package hello_world;

option go_package = "./";

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  // Sends another greeting
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

####  <span id="compile_proto">2.编译.proto文件</span>

创建一个 ``grpc_utils``文件夹 用来存储编译后的文件

编译原型（为了可读性而添加了格式）：

```powershell
python -m grpc_tools.protoc
    -I./protos
    -I./ 
    --python_out=./grpc_utils
    --grpc_python_out=./grpc_utils
    ./protos/helloworld.proto
```

第一个 ``-I`` 代表 proto文件的路径

第二个``-I``代表当前位置 以当前位置找其他相对路径

通过编译`helloworld.proto`生成：

- `grpc_utils/hello_world_pb2.py`
- `grpc_utils/hello_world_pb2_gprc.py`

#### 4.结构

现在我的结构的这样的

```
|____protos
| |____hello_world.proto
|
|____grpc_utils
| |____hello_world_pb2.py
| |____hello_world_pb2_gprc.py
```

#### 3.注意

注意 `hello_world_pb2_gprc.py` 里面的导入有问题

加个 from . 就可以了

```python
from . import hello_world_pb2 as hello__world__pb2
```
### 三.编写server&client

#### 1.greeter_server.py

```python
# file: 'greeter_server.py'
# Copyright 2015 gRPC authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
"""The Python implementation of the GRPC helloworld.Greeter server."""

from concurrent import futures

import logging

import grpc

from grpc_utils import hello_world_pb2, hello_world_pb2_grpc


class Greeter(hello_world_pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        return hello_world_pb2.HelloReply(message='Hello, %s!' % request.name)

    def SayHelloAgain(self, request, context):
        return hello_world_pb2.HelloReply(message='Hello, %s Again!' % request.name)


def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    hello_world_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
    server.add_insecure_port('[::]:50051')
    server.start()
    server.wait_for_termination()


if __name__ == '__main__':
    logging.basicConfig()
    serve()
```

#### 2.greeter_client.py

```python
# file: 'greeter_client.py'
# Copyright 2015 gRPC authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
"""The Python implementation of the GRPC helloworld.Greeter client."""

from __future__ import print_function
import logging

import grpc

from grpc_utils import hello_world_pb2, hello_world_pb2_grpc


def run():
    # NOTE(gRPC Python Team): .close() is possible on a channel and should be
    # used in circumstances in which the with statement does not fit the needs
    # of the code.
    with grpc.insecure_channel('localhost:50051') as channel:
        stub = hello_world_pb2_grpc.GreeterStub(channel)
        response = stub.SayHello(hello_world_pb2.HelloRequest(name='you'))
        print("SayHello received: " + response.message)
        response = stub.SayHelloAgain(hello_world_pb2.HelloRequest(name='you'))
        print("SayHelloAgain received: " + response.message)


if __name__ == '__main__':
    logging.basicConfig()
    run()
```

#### 4.结构

现在我的结构的这样的

```
|____protos
| |____hello_world.proto
|
|____grpc_utils
| |____hello_world_pb2.py
| |____hello_world_pb2_gprc.py
|
|____greeter_server.py
|____greeter_client.py
```

#### 3.启动服务

运行`greeter_server` 开启服务

启动`greeter_client` 返回执行结果

```powershell
SayHello received: Hello, you!
SayHelloAgain received: Hello, you Again!

Process finished with exit code 0
```

### 四.grpc-getway允许http请求

我们想调用grpc的服务必须编写对应client的代码，我们可以使用grpc-getway来反向代理 grpc，以达到通过http请求调用。

#### 1.准备工作


使用grpc-getway 需要go环境的支持 [go官方](https://golang.org/dl/)/[国内镜像](https://gomirrors.org)

我的环境是 win10，go1.17

安装配置好环境变量`GOPATH`

![gopath](/assets/img/grpc/gopath.png)

![path](/assets/img/grpc/path.png)



`go path`里面需要这三个文件夹

#### ![gopathdir](/assets/img/grpc/gopathdir.png)

我这里使用`go module` 来管理依赖包，打开`go module` 。

```shell
set GO111MODULE=on
```

输入`go env`检验一下是否修改成功

#### 2.下载依赖包

创建`tools.go`文件

```go
// file: 'tools.go'
// +build tools

package tools

import (
	_ "github.com/golang/protobuf/protoc-gen-go"
	_ "github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway"
	_ "github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2"
	_ "google.golang.org/grpc/cmd/protoc-gen-go-grpc"
	_ "google.golang.org/protobuf/cmd/protoc-gen-go"
)
```

初始化go mod 

```powershell
go mod init hello_world
```

下载依赖包

```powershell
go mod tidy
```

安装依赖包

```powershell
go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway
go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2
go install google.golang.org/protobuf/cmd/protoc-gen-go
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc
```

安装成功后 会在 `%GOPATH%\bin`下看见对应的.exe 文件

#### 3.修改proto文件

```diff
syntax = "proto3";

+ import "google/api/annotations.proto";

package hello_world;

option go_package = "./";

// The greeting service definition.
service Greeter {
  // Sends a greeting
- rpc SayHello (HelloRequest) returns (HelloReply) {}
+ rpc SayHello (HelloRequest) returns (HelloReply) {
+    option (google.api.http) = {
+      post: "/v1/example/echo"
+      body: "*"
+    };
+ }
  // Sends another greeting
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

`annotations.proto`需要从[googleapis/google/api](https://github.com/googleapis/googleapis/tree/master/google/api)下载

我们把 `annotations.proto` 及其依赖项都要下载

![google_proto](/assets/img/grpc/google_proto.png)

##### 重新编译

执行[编译proto](#compile_proto)下的编译命令 重新编译proto文件

##### 报错

发现`hello_world_pb2.py`导包问题报错 找不到google.api

![google_api](/assets/img/grpc/google_api.png)

安装`googleapis-common-protos`解决问题

```power
pip install googleapis-common-protos
```

这个包会将[googleapis/google/api](https://github.com/googleapis/googleapis/tree/master/google/api)里面的所有proto编译成`.py`文件 直接调用就可以了


#### 4.编译go文件

先创建`./gen/go/`文件夹存储

生成 pb.go 文件

```python
python -m grpc.tools.protoc
	-I.
    --go_out ./gen/go/
    --go-grpc_out ./gen/go/
    ./protos/hello_world.proto
```

生成 pb.gw.go 文件

```python
python -m grpc.tools.protoc
	-I.
	--grpc-gateway_out=logtostderr=true:./gen/go/
    ./protos/hello_world.proto
```

#### 5.编写 HTTP 反向代理服务器

创建 `proxy.go`

```go
// file: 'proxy.proto'
package main

import (
    "context"
    "log"
    "flag"
    "net/http"

    "github.com/golang/glog"
    "github.com/grpc-ecosystem/grpc-gateway/v2/runtime"
    "google.golang.org/grpc"

    gw "github.com/hello_world/gen/go"  // Update
)

var (
    // command-line options:
    // gRPC server endpoint
    grpcServerEndpoint = flag.String("grpc-server-endpoint",  "localhost:50051", "gRPC server endpoint")
)

func run() error {
    ctx := context.Background()
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()

    // Register gRPC server endpoint
    // Note: Make sure the gRPC server is running properly and accessible
    mux := runtime.NewServeMux()
    opts := []grpc.DialOption{grpc.WithInsecure()}
    err := gw.RegisterGreeterHandlerFromEndpoint(ctx, mux,  *grpcServerEndpoint, opts)
    if err != nil {
        return err
    }

    // Start HTTP server (and proxy calls to gRPC server endpoint)
    log.Print("Greeter gRPC Server gateway start at port 8081...")
    return http.ListenAndServe(":8081", mux)
}

func main() {
    flag.Parse()
    defer glog.Flush()

    if err := run(); err != nil {
        glog.Fatal(err)
    }
}
```

`gw "github.com/hello_world/gen/go"` 这一行导入我们刚刚生成的go文件

需要在go.mod 里指向本地的目录

```diff
module hello_world

go 1.17

+ require(
+	 github.com/hello_world/gen/go v0.0.0
+ )
+ replace github.com/hello_world/gen/go => ./gen/go
```

#### 6.打包go文件

在打包之前 我们也需要对 `./gen/go`进行安装依赖

```powershell
cd ./gen/go
```

初始化go mod

```powershell
go mod init gen_go
```

下载依赖包

```powershell
go mod tidy
```



好了我们的依赖已经下载完成可以打包了

```
go build .
```

打包完成之后会看见`hello_world.exe`

启动这个可执行文件

```powershell
grpc_project>hello_world.exe
2021/09/02 15:34:39 Greeter gRPC Server gateway start at port 8081...
```

![post_grpc-getway](/assets/img/grpc/post_grpc_getway.png)

ok post发送成功

#### 4.结构	

现在的我目录是这样的

```markdown
|____gen
| |____go
| | |____go.mod
| | |____hello_world.pb.go
| | |____hello_world.pb.gw.go
| | |____hello_world_grpc.pb.go
|
|____google
| |____api
| | |____annotations.proto
| | |____field_behavior.proto
| | |____http.proto
| | |____httpbody.proto
|
|____grpc_utils
| |____hello_world_pb2.py
| |____hello_world_pb2_gprc.py
|
|____protos
| |____hello_world.proto
|
|____go.mod
|____greeter_server.py
|____greeter_client.py
|____hello_world.exe
|____proxy.go
```

