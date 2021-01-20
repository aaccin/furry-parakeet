---
title: gRPC系列(一) - Protobuf
tags: [go, grpc]
---

[来源](https://www.lixueduan.com/post/grpc/01-protobuf/)

本文主要记录了 Protobuf 的基本使用。包括 编译器 protoc 、Go Plugins 安装及 .proto文件定义、编译等。

## 1.概述

Protocol buffers 是一种语言无关、平台无关的可扩展机制或者说是数据交换格式，用于序列化结构化数据。与 XML、JSON 相比，Protocol buffers 序列化后的码流更小、速度更快、操作更简单。

Protocol buffers are a language-neutral, platform-neutral extensible mechanism for serializing structured data.

## 2. Protocol Compiler

protoc 用于编译 protocolbuf (.proto文件) 和 protobuf 运行时。

protoc 是 C++ 写的，比较简单的安装方式是直接下载编译好的二进制文件。

release 版本下载地址如下

[下载](https://github.com/protocolbuffers/protobuf/releases)

下载操作系统对应版本然后解压然后配置一下环境变量即可。

比如windows就下载protoc-3.14.0-win64.zip 然后把解压后的xxx\protoc-3.14.0-win64\bin配置到环境变量。

linux则下载protoc-3.14.0-linux-x86_64.zip

```
解压
unzip protoc-3.14.0-linux-x86_64.zip -d protoc-3.14.0-linux-x86_64
sudo vim /etc/profile 
#记得改成自己的路径
export PATH=$PATH:/home/lixd/17x/protoc-3.14.0-linux-x86_64/bin
source /etc/profile
```

查看是否安装成功

```
protoc --version

libprotoc 3.14.0

```

## 3. Go Plugins

出了安装 protoc 之外还需要安装各个语言对应的编译插件，我用的 Go 语言，所以还需要安装一个 Go 语言的编译插件。

```
go get google.golang.org/protobuf/cmd/protoc-gen-go

```

## 4. Demo

创建.proto 文件

hello_world.proto

```
//声明 protobuf 版本 只有 proto3 才支持 gRPC
syntax = "proto3";
// .表示生成go文件输出在当前目录，proto 表示生成go文件包名为proto
option go_package = ".;proto";
// 指定当前proto文件属于helloworld包
package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}
//
// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}

```

protoc 编译

编译命令

```
# Syntax: protoc [OPTION] PROTO_FILES
$ protoc --proto_path=IMPORT_PATH  --go_out=OUT_DIR  --go_opt=paths=source_relative path/to/file.proto
```

这里简单介绍一下 golang 的编译姿势:

–proto_path 或者-I ：指定 import 路径，可以指定多个参数，编译时按顺序查找，不指定时默认查找当前目录。
.proto 文件中也可以引入其他 .proto 文件，这里主要用于指定被引入文件的位置。
–go_out：golang编译支持，指定输出文件路径
其他语言则替换即可，比如 --java_out 等等
–go_opt：指定参数，比如--go_opt=paths=source_relative就是表明生成文件输出使用相对路径。
path/to/file.proto ：被编译的 .proto 文件放在最后面

```
protoc --go_out=. hello_word.proto
```

编译后会生成一个hello_word.pb.go文件。
到此为止就ok了。

## 5. 编译过程

可以把 protoc 的编译过程分成简单的两个步骤：

1. 解析 .proto 文件，编译成 protobuf 的原生数据结构保存在内存中；
2. 把 protobuf 相关的数据结构传递给相应语言的编译插件，由插件负责将接收到的 protobuf 原生结构渲染输出为特定语言的模板。

具体过程如图所示：

protobuf-process

protoc 中原生包含了部分语言（java、php、python、ruby等等）的编译插件，但是没有 Go 语言的，所以需要额外安装一个插件。

具体原生支持见源码https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/compiler/main.cc

同样的，后续讲到的 gRPC Plugins、gRPC-Gateway 也是一个个的 protoc 编译插件，将 .proto 文件编译成对应模块需要的源文件。
