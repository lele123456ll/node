# RPC框架之Thrift

## Thrift是什么?

> Thrift是一种接口描述语言和二进制通讯协议，它被用来定义和创建跨语言的服务。它被当作一个远程过程调用（RPC）框架来使用，是由Facebook为“大规模跨语言服务开发”而开发的。它通过一个代码生成引擎联合了一个软件栈，来创建不同程度的、无缝的跨平台高效服务，可以使用C#、C++（基于POSIX兼容系统）、Cappuccino、Cocoa、Delphi、Erlang、Go、Haskell、Java、Node.js、OCaml、Perl、PHP、Python、Ruby和Smalltalk。

## 支持的通讯协议

- TBinaryProtocol : `一种简单的二进制格式，简单，但没有为空间效率而优化。比文本协议处理起来更快，但更难于调试。`
- **TCompactProtocol** : `更紧凑的二进制格式，处理起来通常同样高效。`
- TDenseProtocol :`与TCompactProtocol类似，将传输数据的元信息剥离。`
- TJSONProtocol: `使用JSON对数据编码。`
- TSimpleJSONProtocol: `一种只写协议，它不能被Thrift解析，因为它使用JSON时丢弃了元数据。适合用`脚本语言来解析

## 支持的传输协议

- TFileTransport : `该传输协议会写文件。`
- **TFramedTransport** :`当使用一个非阻塞服务器时，要求使用这个传输协议。它按帧来发送数据，其中每一帧的开头是长度信息。`
- TMemoryTransport : `使用存储器映射输入输出。(Java的实现使用了一个简单的ByteArrayOutputStream。)`
- TSocket : `使用阻塞的套接字I/O来传输。`
- TZlibTransport : `用zlib执行压缩。用于连接另一个传输协议。`

## 支持的服务模型

- TNonblockingServer : `一个多线程服务器，它使用非阻塞I/O（Java的实现使用了NIO通道）。TFramedTransport必须跟这个服务器配套使用。`
- TSimpleServer : `一个单线程服务器，它使用标准的阻塞I/O。测试时很有用。`
- TThreadPoolServer : `一个多线程服务器，它使用标准的阻塞I/O`
- **THsHaServer** : `YHsHa引入了线程池去处理（需要使用TFramedTransport数据传输方式），其模型把读写任务放到线程池去处理;Half-sync/Half-async（半同步半异步）的处理模式;Half-sync是在处理IO时间上（sccept/read/writr io）,Half-async用于handler对RPC的同步处理`