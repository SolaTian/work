# Web Services

了解 Web Services之前，先了解一下，什么是 Web 应用程序？和普通的应用程序的区别是什么？

|区别|Web 应用程序|普通应用程序|
|-|-|-|
|部署和访问方式|<li>运行在浏览器中，无需用户安装，只需访问远程链接即可使用。<li>通过网络连接到远程服务器，通常依赖于互联网|安装在用户的设备上，可以脱机使用|
|平台兼容性|跨平台，只要联网可以在任何操作系统和设备上运行|通常针对特定操作系统（Windows、macOS、Linux、iOS、Android等）开发，需要针对不同平台做不同的版本。|
|更新和维护|基于服务器运行，更新和维护只需在服务器端运行|需用户手动更新|
|性能和资源|性能受限于网络速度和浏览器能力，依赖服务器资源|直接访问设备的硬件资源，不受网络限制|
|开发技术|常用技术包括HTML、CSS、JavaScript（前端），以及各种后端服务器技术（如Node.js、PHP、Python、Ruby等）。|开发语言通常与操作系统相关，如C/C++、Java、Swift、Kotlin等。|


> Web Services 可以使应用程序成为 Web 应用程序，是一种应用程序组件。


几乎所有的平台都可以借由 Web 浏览器来访问 Web。为了使不同的平台之间交互，Web 应用程序被开发出来。

Web Services 使用 XML 来编解码数据，并使用 SOAP 借由开放的协议来传输数据。通过使用 Web Services，可以在不同的应用程序与平台之间来交换数据。

## Web Services 的三种基本元素

分别是 SOAP、 WSDL 和 UDDI

### 1、SOAP

> SOAP（Simple Object Access Protocol，简单对象访问协议）：基于 XML 的简易协议，可以使应用程序在 HTTP 上进行信息交换。属于应用层协议。

#### 1.1、SOAP 的工作原理

SOAP 协议使用 XML 格式来定义消息的结构和内容。在使用 SOAP 进行通信时，发送方会将消息打包成 XML 格式，然后通过 HTTP 或 SMTP 等协议发送给接收方。接收方会解析 XML 格式的消息，从中提取出所需的信息。

SOAP 协议提供了一组规范来定义消息的结构和内容。这些规范包括 SOAP Envelope、SOAP Header 和 SOAP Body。SOAP Envelope 是 SOAP 消息的根元素，它包含了整个 SOAP 消息的描述信息。SOAP Header 是可选的，用于传递与消息相关的其他信息，如安全认证信息。SOAP Body 包含了实际的消息内容。

#### 1.2、SOAP 的优劣势

优势：
- 跨语言性和跨平台性：由于 SOAP 使用 XML 格式定义消息结构，因此它可以在不同的操作系统和编程语言之间进行通信。
- 支持多种安全机制：如数字签名和加密，确保安全性。

劣势：
- 消息格式冗长：SOAP 的消息格式比较冗长，可能会导致传输效率低下。
- 需要额外的协议层：SOAP 构建在额外的应用层协议之上，如 HTTP 和 SMTP来传递消息，其本身并不处理底层传输。


#### 1.3、SOAP 的部分语法

一条 SOAP 消息就是一个普通的 XML 文档，包含下列元素

- 必需的 Envelope 元素，标识 XML 为一条 SOAP 消息
- 可选的 Header 元素，包含头部信息
- 必需的 Body 元素，包含所有的调用和响应信息
- 可选的 Fault 元素，提供有关在处理此消息所发生错误的信息

基本结构如下：

    <?xml version="1.0"?>
    <soap:Envelope
        <!-- xmlns:soap 命名空间-->
    xmlns:soap="http://www.w3.org/2001/12/soap-envelope"
        <!-- encodingStyle 属性用于定义在文档中使用的数据类型-->
    soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">

    <soap:Header>
        <m:Trans xmlns:m="http://www.w3schools.com/transaction/"
        soap:mustUnderstand="1">234
        </m:Trans>
    </soap:Header>

    <soap:Body>
        <m:GetPrice xmlns:m="http://www.w3schools.com/prices">
        <m:Item>Apples</m:Item>
        </m:GetPrice>
        
        <soap:Fault>
        ...
        </soap:Fault>
    </soap:Body>

    </soap:Envelope>

SOAP 请求可能是 HTTP POST 或者 HTTP GET，且至少规定两个头部，Content-Type 和 Content-Length

示例

    POST /item HTTP/1.1
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: 250

#### 1.4、一个完整的 SOAP 请求和响应

SOAP 请求：

    POST /InStock HTTP/1.1
    Host: www.example.org
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: nnn

    <?xml version="1.0"?>
    <soap:Envelope
    xmlns:soap="http://www.w3.org/2001/12/soap-envelope"
    soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">

    <soap:Body xmlns:m="http://www.example.org/stock">
        <m:GetStockPrice>
        <m:StockName>IBM</m:StockName>
        </m:GetStockPrice>
    </soap:Body>

    </soap:Envelope>

SOAP 响应：

    HTTP/1.1 200 OK
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: nnn

    <?xml version="1.0"?>
    <soap:Envelope
    xmlns:soap="http://www.w3.org/2001/12/soap-envelope"
    soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">

    <soap:Body xmlns:m="http://www.example.org/stock">
        <m:GetStockPriceResponse>
        <m:Price>34.5</m:Price>
        </m:GetStockPriceResponse>
    </soap:Body>

    </soap:Envelope>

### 2、WSDL

> WSDL（网络服务描述语言，Web Services Description Language）是一门基于 XML 的语言，用于描述 Web Services 以及如何访问 Web Services 的语言。

WSDL 本质就是一种使用 XML 编写的文档。这种文档可描述某个 Web service。它可规定服务的位置，以及此服务提供的操作（或方法）


#### 2.1、WSDL 的组成

|元素|	定义|
|-|-|
| `<portType>` |Web Service 可执行的操作，可以比作传统编程语言中的一个函数库（或一个模块、或一个类）|
| `<message>` |Web Service 使用的消息，每个消息均由一个或多个部件组成。可以把这些部件比作传统编程语言中一个函数调用的参数。|
| `<types>` |Web Service 使用的数据类型|
| `<binding>` |Web Service 使用的通信协议|

一个简单的 WSDL 实例

    <message name="getTermRequest">
        <part name="term" type="xs:string"/>
    </message>
 
    <message name="getTermResponse">
        <part name="value" type="xs:string"/>
    </message>
 
    <portType name="glossaryTerms">
        <operation name="getTerm">
            <input message="getTermRequest"/>
            <output message="getTermResponse"/>
        </operation>
    </portType>

    <binding type="glossaryTerms" name="b1">
        <soap:binding style="document"
        transport="http://schemas.xmlsoap.org/soap/http" />
        <operation>
            <soap:operation soapAction="http://example.com/getTerm"/>
            <input><soap:body use="literal"/></input>
            <output><soap:body use="literal"/></output>
        </operation>
    </binding>

`<portType>` 元素把 "glossaryTerms" 定义为某个端口的名称，把 "getTerm" 定义为某个操作的名称。操作 "getTerm" 拥有一个名为 "getTermRequest" 的输入消息，以及一个名为 "getTermResponse" 的输出消息。`<message>` 元素可定义每个消息的部件，以及相关联的数据类型。

`<binding>` 元素有两个属性 name 属性和 type 属性。name 属性定义 binding 的名称，而 type 属性指向用于 binding 的端口，在这个例子中是 "glossaryTerms" 端口。

soap:binding 元素有两个属性 style 属性和 transport 属性。style 属性可取值 "rpc" 或 "document"。在这个例子中使用 document。transport 属性定义了要使用的 SOAP 协议。在这个例子中使用 HTTP。

`<operation>` 元素定义了每个端口提供的操作符。


### 3、UDDI

> UDDI 是一种目录服务，通过它，企业可注册并搜索 Web services。

它是一个基于 XML 的跨平台的描述规范，可以使世界范围内的企业在互联网上发布自己所提供的服务。

假如行业发布了一个用于航班比率检测和预订的 UDDI 标准，航空公司就可以把它们的服务注册到一个 UDDI 目录中。然后旅行社就能够搜索这个 UDDI 目录以找到航空公司预订界面。当此界面被找到后，旅行社就能够立即与此服务进行通信，这样由于它使用了一套定义良好的预订界面。

## 其他的 Web 数据交互方式

上面介绍了 Web Services 这一 Web 应用程序（浏览器之间，浏览器与服务器之间）的数据交互方式，还有很多其他的方式。

|Web 应用程序数据交互方式|说明|应用场景|
|-|-|-|
|传统的表单提交|直接通过 HTTP/HTTPS POST 或 GET 等方法进行数据交换|从服务器请求页面内容、API接口、上传文件、提交表单数据等|
|AJAX|一种在不重新加载整个页面的情况下，浏览器与服务器进行异步数据交换的技术。|动态加载数据、部分页面更新（例如，实时聊天、推荐系统、自动保存等）|
|Web Socket|允许浏览器和服务器在建立连接后进行双向实时数据传输。与传统的HTTP请求不同，WebSocket连接是持久的，可以随时发送数据而不需要重新建立连接|实时应用（如在线游戏、聊天应用、股票行情更新等），需要快速、低延迟的数据传输。|
|Web Services（RESTful API）|上面介绍的 Web Services 是基于 SOAP 的，Web Services（RESTful API）一种基于 HTTP 协议、采用 REST 架构风格的 Web 服务，它通常使用标准的 HTTP 动词（如 GET、POST、PUT、DELETE）来操作资源，返回数据的格式通常是 JSON 或 XML| 比如前端 Web 应用通过 RESTful API 向服务器提交用户输入的表单数据（如注册、登录、评论）|
|中间件和消息传递协议|如 MQTT 等消息队列|适合带宽有限或网络不稳定的环境|

