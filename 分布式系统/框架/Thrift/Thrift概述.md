## Thrift
Thrift是一个可伸缩的跨语言服务开发框架。

目前流行的服务调用方式有很多种，例如基于 SOAP 消息格式的 Web Service，基于 JSON 消息格式的 RESTful 服务等

。其中所用到的数据传输方式包括 XML，JSON 等，**然而 XML 相对体积太大，传输效率低，JSON 体积较小，新颖，但还不够完善**。

本文将介绍由 Facebook 开发的远程服务调用框架 Apache Thrift，它采用接口描述语言定义并创建服务，支持可扩展的跨语言服务开发，所包含的代码生成引擎可以在多种语言中，如 C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, Smalltalk 等创建高效的、无缝的服务，其传输数据采用二进制格式，相对 XML 和 JSON 体积更小，对于高并发、大数据量和多语言的环境更有优势。本文将详细介绍 Thrift 的使用，并且提供丰富的实例代码加以解释说明，帮助使用者快速构建服务。

Thrift的核心在于利用IDL(**Interface Definition Language,接口定义语言**),该语言可以用于定义一系列与具体编程无关的的类型和接口信息，并最后生成特定语言的代码。
