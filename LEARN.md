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