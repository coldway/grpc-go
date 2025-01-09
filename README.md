# gRPC-Go

[![GoDoc](https://pkg.go.dev/badge/google.golang.org/grpc)][API]
[![GoReportCard](https://goreportcard.com/badge/grpc/grpc-go)](https://goreportcard.com/report/github.com/grpc/grpc-go)
[![codecov](https://codecov.io/gh/grpc/grpc-go/graph/badge.svg)](https://codecov.io/gh/grpc/grpc-go)

The [Go][] implementation of [gRPC][]: A high performance, open source, general
RPC framework that puts mobile and HTTP/2 first. For more information see the
[Go gRPC docs][], or jump directly into the [quick start][].

## Prerequisites

- **[Go][]**: any one of the **two latest major** [releases][go-releases].

## Installation

Simply add the following import to your code, and then `go [build|run|test]`
will automatically fetch the necessary dependencies:


```go
import "google.golang.org/grpc"
```

> **Note:** If you are trying to access `grpc-go` from **China**, see the
> [FAQ](#FAQ) below.

## Learn more

- [Go gRPC docs][], which include a [quick start][] and [API
  reference][API] among other resources
- [Low-level technical docs](Documentation) from this repository
- [Performance benchmark][]
- [Examples](examples)

## FAQ

### I/O Timeout Errors

The `golang.org` domain may be blocked from some countries. `go get` usually
produces an error like the following when this happens:

```console
$ go get -u google.golang.org/grpc
package google.golang.org/grpc: unrecognized import path "google.golang.org/grpc" (https fetch: Get https://google.golang.org/grpc?go-get=1: dial tcp 216.239.37.1:443: i/o timeout)
```

To build Go code, there are several options:

- Set up a VPN and access google.golang.org through that.

- With Go module support: it is possible to use the `replace` feature of `go
  mod` to create aliases for golang.org packages.  In your project's directory:

  ```sh
  go mod edit -replace=google.golang.org/grpc=github.com/grpc/grpc-go@latest
  go mod tidy
  go mod vendor
  go build -mod=vendor
  ```

  Again, this will need to be done for all transitive dependencies hosted on
  golang.org as well. For details, refer to [golang/go issue
  #28652](https://github.com/golang/go/issues/28652).

### Compiling error, undefined: grpc.SupportPackageIsVersion

Please update to the latest version of gRPC-Go using
`go get google.golang.org/grpc`.

### How to turn on logging

The default logger is controlled by environment variables. Turn everything on
like this:

```console
$ export GRPC_GO_LOG_VERBOSITY_LEVEL=99
$ export GRPC_GO_LOG_SEVERITY_LEVEL=info
```

### The RPC failed with error `"code = Unavailable desc = transport is closing"`

This error means the connection the RPC is using was closed, and there are many
possible reasons, including:
 1. mis-configured transport credentials, connection failed on handshaking
 1. bytes disrupted, possibly by a proxy in between
 1. server shutdown
 1. Keepalive parameters caused connection shutdown, for example if you have
    configured your server to terminate connections regularly to [trigger DNS
    lookups](https://github.com/grpc/grpc-go/issues/3170#issuecomment-552517779).
    If this is the case, you may want to increase your
    [MaxConnectionAgeGrace](https://pkg.go.dev/google.golang.org/grpc/keepalive?tab=doc#ServerParameters),
    to allow longer RPC calls to finish.

It can be tricky to debug this because the error happens on the client side but
the root cause of the connection being closed is on the server side. Turn on
logging on __both client and server__, and see if there are any transport
errors.

[API]: https://pkg.go.dev/google.golang.org/grpc
[Go]: https://golang.org
[Go module]: https://github.com/golang/go/wiki/Modules
[gRPC]: https://grpc.io
[Go gRPC docs]: https://grpc.io/docs/languages/go
[Performance benchmark]: https://performance-dot-grpc-testing.appspot.com/explore?dashboard=5180705743044608
[quick start]: https://grpc.io/docs/languages/go/quickstart
[go-releases]: https://golang.org/doc/devel/release.html

# Learn
Server：
```
func NewServer(opt ...ServerOption) *Server  // 新建服务函数
func (s *Server) Serve(lis net.Listener)     // 服务启动函数 //accept连接，然后handleRawConn处理此连接请求（新建goroutine）
func (s *Server) handleRawConn(lisAddr string, rawConn net.Conn) // 处理请求。目前来看：唯一入口
```
client：
```
func (cc *ClientConn) Invoke(ctx context.Context, method string, args, reply any, opts ...CallOption) error // 所有client调用的入口
```
## 流和一元 grpc 接口
在gRPC（Google Remote Procedure Call）框架中，streamDesc和MethodDesc是与服务定义和RPC调用相关的两个概念，但它们各自扮演着不同的角色。然而，需要注意的是，直接提及streamDesc和MethodDesc作为gRPC核心API或概念的一部分可能不是非常标准或广泛认可的，因为gRPC的官方文档和API通常使用更高级别的术语来描述这些功能。不过，基于您的问题和gRPC的一般工作原理，我可以尝试解释这两个概念可能代表的含义和区别。
## streamDesc（流描述符）
在gRPC的上下文中，streamDesc可能是一个内部或框架级别的概念，用于描述一个RPC调用中的流（stream）特性。在gRPC中，流允许客户端和服务器之间发送一系列的消息，而不是单个请求-响应对。这种机制支持四种模式：一元（unary）、客户端流式（client streaming）、服务器流式（server streaming）和双向流式（bidirectional streaming）‌1。
streamDesc可能是一个内部数据结构或元数据，用于在gRPC框架内部表示这些流的特性，如流的方向、消息类型等。然而，这个术语并不是gRPC官方文档中明确提到的，因此它可能更多地与gRPC框架的实现细节相关。
## MethodDesc（方法描述符）
相比之下，MethodDesc（或类似的概念）更可能是一个用于描述RPC方法本身的元数据或数据结构。在gRPC中，每个服务都定义了一组可以被远程调用的方法，每个方法都有其特定的请求和响应类型‌12。MethodDesc可能包含了关于这些方法的信息，如方法名、请求类型、响应类型以及它是否支持流式传输等。
在gRPC的实现中，这些信息通常用于在客户端和服务器之间建立连接时，进行服务发现和方法的调用。例如，当客户端想要调用服务器上的一个RPC方法时，它会发送一个包含方法描述符的请求，服务器则根据这个描述符来查找并执行相应的方法。
## 区别
综上所述，streamDesc和MethodDesc的主要区别在于它们所描述的内容不同：
- streamDesc

（如果确实存在这样的概念）可能更侧重于描述RPC调用中的流特性，如流的方向和消息类型。
- MethodDesc

则更可能是一个用于描述RPC方法本身的元数据或数据结构，包括方法名、请求和响应类型等信息。
然而，需要注意的是，由于streamDesc可能不是一个广泛认可的gRPC术语，因此上述解释可能基于假设和gRPC的一般工作原理。在实际应用中，您应该参考gRPC的官方文档和API来获取最准确的信息。
如果您是在处理gRPC框架的源代码或特定于某个语言的gRPC实现时遇到这些术语，那么最好查阅该实现的具体文档或源代码注释来获取更详细的信息。
下面是自动生成ServiceDesc，里面包含些默认信息，不同的grpc 模式生成的ServiceDesc结果不一样，其中所有的stream 模式的方法，服务器都是生成Streams方法。
steam和普通Methods除了handler不同，stream还多了两个bool值，用来标识客户端或者服务端可以用流模式发送数据
```
ServerStreams bool // indicates the server can perform streaming sends
ClientStreams bool // indicates the client can perform streaming sends
```
下面是一元rpc 的desc
```
var Greeter_ServiceDesc = grpc.ServiceDesc{
  ServiceName: “helloworld.Greeter”,
  HandlerType: (*GreeterServer)(nil),
  Methods: []grpc.MethodDesc{
    {
      MethodName: “SayHello”,
      Handler: _Greeter_SayHello_Handler,
    },
  },
  Streams: []grpc.StreamDesc{},
  Metadata: “examples/helloworld/helloworld/helloworld.proto”,
}
```
服务端流模式
```
var _PropService_serviceDesc = grpc.ServiceDesc{
  ServiceName: “PropService”,
  HandlerType: (*PropServiceServer)(nil),
  Methods: []grpc.MethodDesc{},
  Streams: []grpc.StreamDesc{
    {
      StreamName: “UserProp”,
      Handler: _PropService_UserProp_Handler,
      ServerStreams: true, // 服务端流只有服务端可以发stream数据，所以 ServerStreams为true
    },
  },
  Metadata: “protos/item_update.proto”,
}
```
客户端流模式
```
var _DataService_serviceDesc = grpc.ServiceDesc{
  ServiceName: “DataService”,
  HandlerType: (*DataServiceServer)(nil),
  Methods: []grpc.MethodDesc{},
  Streams: []grpc.StreamDesc{
    {
      StreamName: “DataUpload”,
      Handler: _DataService_DataUpload_Handler,
      ClientStreams: true, // 客户端流只有客户端可以发steam数据，所以 ClientStreams为true
    },
  },
  Metadata: “protos/data.proto”,
```
双向流模式
```
var _BattleService_serviceDesc = grpc.ServiceDesc{
  ServiceName: “BattleService”,
  HandlerType: (*BattleServiceServer)(nil),
  Methods: []grpc.MethodDesc{},
  Streams: []grpc.StreamDesc{
    {
      StreamName: “Battle”,
      Handler: _BattleService_Battle_Handler,
      // 双向流两端都是可以发stream数据的，所以ServerStreams和ClientStreams都是true
      ServerStreams: true,
      ClientStreams: true,
    },
  },
  Metadata: “protos/battle.proto”,
}
```
注意到一元rpc 的ServiceName和其他不一样,为helloworld.Greeter，这是因为一元rpc 命令定义了包名package helloworld，所以ServiceDesc的ServiceName为"protobuf包名.protobuf 定义服务名"



另外的文档：
不良的命名造成的问题

reference
https://blog.csdn.net/2401_84239525/article/details/137994865