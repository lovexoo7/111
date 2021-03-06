# <a name="目录">目录</a>
+ [附录B 典型咨询问题](#附录B 典型咨询问题)
    * [B.1 基础问题](#B.1 基础问题)
        - [获取技术支持](#获取技术支持)
        - [信息安全](#信息安全)
        - [使用限制](#使用限制)
    * [B.2 移植阶段问题](#B.2 移植阶段问题)
    * [B.3 编译阶段问题](#B.3 编译阶段问题)
    * [B.4 错误码自查](#B.4 错误码自查)
        - [TLS/SSL连接错误](#TLS/SSL连接错误)
            + [-0x7880/-30848/MBEDTLS_ERR_SSL_PEER_CLOSE_NOTIFY](#-0x7880/-30848/MBEDTLS_ERR_SSL_PEER_CLOSE_NOTIFY)
            + [-0x7800/-30720/MBEDTLS_ERR_SSL_PEER_VERIFY_FAILED](#-0x7800/-30720/MBEDTLS_ERR_SSL_PEER_VERIFY_FAILED)
            + [-0x7200/-29184/MBEDTLS_ERR_SSL_INVALID_RECORD](#-0x7200/-29184/MBEDTLS_ERR_SSL_INVALID_RECORD)
            + [-0x2700/-9984/MBEDTLS_ERR_X509_CERT_VERIFY_FAILED](#-0x2700/-9984/MBEDTLS_ERR_X509_CERT_VERIFY_FAILED)
            + [-0x0052/-82/MBEDTLS_ERR_NET_UNKNOWN_HOST](#-0x0052/-82/MBEDTLS_ERR_NET_UNKNOWN_HOST)
            + [-0x0044/-68/MBEDTLS_ERR_NET_CONNECT_FAILED](#-0x0044/-68/MBEDTLS_ERR_NET_CONNECT_FAILED)
            + [-0x0043/-67/MBEDTLS_ERR_NET_BUFFER_TOO_SMALL](#-0x0043/-67/MBEDTLS_ERR_NET_BUFFER_TOO_SMALL)
            + [-0x0042/-66/MBEDTLS_ERR_NET_SOCKET_FAILED](#-0x0042/-66/MBEDTLS_ERR_NET_SOCKET_FAILED)
        - [MQTT连接错误](#MQTT连接错误)
            + [-35/MQTT_CONNACK_BAD_USERDATA_ERROR](#-35/MQTT_CONNACK_BAD_USERDATA_ERROR)
            + [-37/MQTT_CONNACK_IDENTIFIER_REJECTED_ERROR](#-37/MQTT_CONNACK_IDENTIFIER_REJECTED_ERROR)
            + [-42/MQTT_PUSH_TO_LIST_ERROR](#-42/MQTT_PUSH_TO_LIST_ERROR)
    * [B.5 MQTT上云问题](#B.5 MQTT上云问题)
        - [心跳和重连](#心跳和重连)
        - [关于 IOT_MQTT_Yield](#关于 IOT_MQTT_Yield)
        - [订阅相关](#订阅相关)
        - [发布相关](#发布相关)
    * [B.6 高级版问题](#B.6 高级版问题)
    * [B.7 云端/协议问题](#B.7 云端/协议问题)
    * [B.8 本地通信问题](#B.8 本地通信问题)
    * [B.9 WiFi配网问题](#B.9 WiFi配网问题)
    * [B.10 CoAP上云问题](#B.10 CoAP上云问题)
        - [认证连接](#认证连接)
        - [报文上行](#报文上行)
        - [关于 IOT_CoAP_Yield](#关于 IOT_CoAP_Yield)
    * [B.11 HTTP上云问题](#B.11 HTTP上云问题)
        - [认证连接](#认证连接)


# <a name="附录B 典型咨询问题">附录B 典型咨询问题</a>
## <a name="B.1 基础问题">B.1 基础问题</a>
### <a name="获取技术支持">获取技术支持</a>

### <a name="信息安全">信息安全</a>
使用C-SDK连接物联网平台, 传输的安全性怎么保证
---
设备和服务端之间的链路可以通过TLS加密的, 并且使用 productKey/deviceName/deviceSecret 这一组设备相关的信息进行认证, 任何一个环节鉴权失败都会导致连接无法建立

### <a name="使用限制">使用限制</a>
**更多产品使用上的限制边界, 以 [官网限制说明页面](https://help.aliyun.com/document_detail/30527.html) 为准**

一组设备标识支持多个设备登录吗
---
不可以, 一组设备信息(productKey/deviceName/deviceSecret), 同时多个设备登录, 只能连接一个

每次调用 [IOT_MQTT_Construct](#IOT_MQTT_Construct) 或者 [IOT_Linkkit_Connect](#IOT_Linkkit_Connect) 之类的连云接口时都会成功

不过先前建立的链路会被服务器断开, 只保留最近建立的链路, 使同一组设备标识, 同一时间只能有一台实际设备在线

Topic订阅设备超过1000怎么处理
---
广播topic, 最多1000个订阅者. 如果设备超过数1000, 可以对设备进行分组, 1000个一组

比如有5000个设备, 使用广播topic, 用户调5次接口

查看控制台的日志中有以下错误: [error]rate limiter, 请问是什么原因
---
这代表设备被限流了, 单个设备数据上报的上限在 QoS0 下为30条每秒, 在 QoS1 下为10条每秒, 下行接收限制为50条每秒

## <a name="B.2 移植阶段问题">B.2 移植阶段问题</a>

## <a name="B.3 编译阶段问题">B.3 编译阶段问题</a>

## <a name="B.4 错误码自查">B.4 错误码自查</a>

### <a name="TLS/SSL连接错误">TLS/SSL连接错误</a>
#### <a name="-0x7880/-30848/MBEDTLS_ERR_SSL_PEER_CLOSE_NOTIFY">-0x7880/-30848/MBEDTLS_ERR_SSL_PEER_CLOSE_NOTIFY</a>
解释
---
云端把SSL连接断开了: `The peer notified us that the connection is going to be closed`

可能的原因和解决建议
---
- 设备端数据连接过于频繁, 触发云端限流, 断开设备
    * 建议关闭设备, 等待一段时间(5分钟以后)再发起连接重试, 观察错误仍会出现
- 有多个设备使用相同的productKey和deviceName与云端建立连接, 导致被云端踢下线
    * 建议检查当前使用的三元组是否可能被他人使用
- 设备端保活出错, 没有及时发送 MQTT ping packet, 或者被发送了没有及时到达云端
    * 建议用抓包等方式确认心跳包有成功发出或者观察有没有收到来自服务端的 MQTT ping response
- 如果一次都不能连接成功, 可以考虑是不是大小端字节序不匹配
    * 目前C-SDK 默认是适配小端设备, 如果需在大端硬件上工作, 请添加全局编译选项 `REVERSED`

#### <a name="-0x7800/-30720/MBEDTLS_ERR_SSL_PEER_VERIFY_FAILED">-0x7800/-30720/MBEDTLS_ERR_SSL_PEER_VERIFY_FAILED</a>
解释
---
认证错误: `Verification of our peer failed`

可能的原因和解决建议
---
+ 证书错误
    * 如果使用官方C-SDK对接, 证书固化在SDK内部, 不会出现. 若自行对接, 则需检查使用的证书是否和阿里云官方证书匹配
+ 日常环境SSL域名校验错误
    * 如果出错时, 连接的是日常环境, 则考虑日常不支持SSL域名校验, 请将 `FORCE_SSL_VERIFY` 的编译选项定义去掉

#### <a name="-0x7200/-29184/MBEDTLS_ERR_SSL_INVALID_RECORD">-0x7200/-29184/MBEDTLS_ERR_SSL_INVALID_RECORD</a>
解释
---
收到非法数据: `An invalid SSL record was received`

可能的原因和解决建议
---
+ TCP/IP协议栈收到的数据包出错, 需要排查协议栈方面问题
+ SSL所运行的线程栈被设置的过小, 需调整线程栈大小
+ SSL被配置的最大报文长度太小, 当网络报文长度超过该数值时, 则可能出现0x7200错误
    * 可调整 `MBEDTLS_SSL_MAX_CONTENT_LEN` 的值, 重新编译再试
    * `MBEDTLS_SSL_MAX_CONTENT_LEN` 的值, 目前已知最小不能小于 `4096`

#### <a name="-0x2700/-9984/MBEDTLS_ERR_X509_CERT_VERIFY_FAILED">-0x2700/-9984/MBEDTLS_ERR_X509_CERT_VERIFY_FAILED</a>
解释
---
证书错误: `Certificate verification failed, e.g. CRL, CA or signature check failed`

可能的原因和解决建议
---
+ 证书不匹配
    * 若未使用官方C-SDK对接, 请检查证书是否和阿里云官网提供下载的一致
+ 时钟不对
    * 确认系统时间是否准确, 系统时间不对(比如默认的1970-01-01), 也会导致证书校验失败

#### <a name="-0x0052/-82/MBEDTLS_ERR_NET_UNKNOWN_HOST">-0x0052/-82/MBEDTLS_ERR_NET_UNKNOWN_HOST</a>
解释
---
DNS域名解析错误: `Failed to get an IP address for the given hostname`

可能的原因和解决建议
---
+ 大概率是设备端当前网络故障, 无法访问公网
    * 如果是WiFi设备, 检查其和路由器/热点之间的连接状况, 以及上联路由器是否可以正常访问公网

#### <a name="-0x0044/-68/MBEDTLS_ERR_NET_CONNECT_FAILED">-0x0044/-68/MBEDTLS_ERR_NET_CONNECT_FAILED</a>
解释
---
socket连接失败: `The connection to the given server / port failed`

可能的原因和解决建议
---
- TCP连接失败
    * 请确认连接的目标IP地址/域名, 以及端口号是否正确

#### <a name="-0x0043/-67/MBEDTLS_ERR_NET_BUFFER_TOO_SMALL">-0x0043/-67/MBEDTLS_ERR_NET_BUFFER_TOO_SMALL</a>
解释
---
SSL报文缓冲区过短: `Buffer is too small to hold the data`

可能的原因和解决建议
---
+ SSL被配置的最大报文长度太小
    * 可调整 `MBEDTLS_SSL_MAX_CONTENT_LEN` 的值, 重新编译再试
    * `MBEDTLS_SSL_MAX_CONTENT_LEN` 的值, 目前已知最小不能小于 `4096`

#### <a name="-0x0042/-66/MBEDTLS_ERR_NET_SOCKET_FAILED">-0x0042/-66/MBEDTLS_ERR_NET_SOCKET_FAILED</a>
解释
---
创建socket失败: `Failed to open a socket`

可能的原因和解决建议
---
- 系统可用的socket可能已全部被申请, 在有些平台上, 能被开启的socket数量, 有上限的
    * 检查当前的流程是否有socket的泄漏, 有些socket使用完后没有close
    * 如果流程正常, 确需同时建立多个socket, 请调整socket的个数上限

### <a name="MQTT连接错误">MQTT连接错误</a>
#### <a name="-35/MQTT_CONNACK_BAD_USERDATA_ERROR">-35/MQTT_CONNACK_BAD_USERDATA_ERROR</a>
解释
---
发起MQTT协议中的Connect操作失败: `Connect request failed with the server returning a bad userdata error`

可能的原因和解决建议
---
+ 使用了错误的设备三元组
    * 建议检查 productKey/deviceName/deviceSecret 是否正确并且是同一套
+ 三元组传入的参数不正确
    * 建议检查C函数的传参过程
+ 设备三元组被禁用
    * 若经检查确系禁用, 请在控制台或是使用服务端相应API解除禁用

#### <a name="-37/MQTT_CONNACK_IDENTIFIER_REJECTED_ERROR">-37/MQTT_CONNACK_IDENTIFIER_REJECTED_ERROR</a>
解释
---
云端认为设备权限而拒绝设备端的连接请求: `CONNACK` 报文中的错误码是 `2`

可能的原因和解决建议
---
+ 建议检查在控制台创建当前设备三元组的账号, 是否有权限, 是否有发生欠费等

#### <a name="-42/MQTT_PUSH_TO_LIST_ERROR">-42/MQTT_PUSH_TO_LIST_ERROR</a>
解释
---
订阅或是发布(QoS1)太多太快, 导致SDK内部队列满了而出错

可能的原因和解决建议
---
+ 如果是订阅过程中发生的错误, 说明当前订阅使用的队列长度不够
    * 可以考虑调整 `IOTX_MC_SUB_NUM_MAX` 的值
+ 如果是发布过程中发生的错误, 可能是发送的频率太快, 或是网络状态不佳
    * 可以考虑使用QoS0来发布
    * 可以考虑调整 `IOTX_MC_REPUB_NUM_MAX` 的值

## <a name="B.5 MQTT上云问题">B.5 MQTT上云问题</a>

MQTT云端服务器IP地址和端口号
---
端口: 1883

IP地址:

+ 218.11.0.64
+ 116.211.167.65
+ 118.178.217.16
+ 106.15.100.2
+ 139.196.135.135

什么是域名直连, 如何开启
---
C-SDK 的 2.0 以上版本, MQTT连接有两种方式, 一种是认证模式, 一种是域名直连模式

+ 认证模式是设备先用 HTTPS 方式访问类似 https://iot-auth.cn-shanghai.aliyuncs.com:443 的地址, 进行鉴权
+ 鉴权通过之后, 再用 MQTT 方式连接到类似 public.iot-as-mqtt.cn-shanghai.aliyuncs.com:1883 的 MQTT 服务器地址
+ 域名直连模式是设备直接用 MQTT 方式连接到类似 ${productKey}.iot-as-mqtt.cn-shanghai.aliyuncs.com:1883的 MQTT 服务器地址
+ 这样不需要访问 HTTPS+MQTT 两台服务器才建立连接, 更快建立连接, 耗费的端上资源也更少

---
2.0 以上的版本, 默认已经开启了直连模式, 如果配置文件 `make.settings` 不是默认的状态了, 那么

打开 `make.setting` 文件, 在文件尾部写入 `FEATURE_MQTT_DIRECT = y', 然后运行 `make` 命令重新编译, 即可确保开启直连模式

MQTT协议版本是多少
---
C-SDK 的 2.0 以上版本, 封装的MQTT协议版本号是 `3.1.1`

例程 mqtt-example 连接上线后会很快下线, 如何修改可以一直处于在线状态
---
mqtt-example 例程本身的逻辑是发送一次消息后会自动退出

若需要此例程保持长期在线, 执行命令时加上 "loop" 参数即可, 例如:

    .output/release/bin/mqtt-example loop

**当您这样做时, 请确保使用自己申请的 productKey/deviceName/deviceSecret 后重新编译例程**

**因为官方SDK的代码中默认编写的是一台公用设备的信息, 有其他人同时使用时, 您会被踢下线, 同一组设备信息只能同时在线一个连接**

如何持续的接收MQTT消息
---
需要循环多次的调用 [IOT_MQTT_Yield](#IOT_MQTT_Yield) 函数, 在其内部会自动的维持心跳和接收云端下发的MQTT消息

可以参考 examples 目录下, mqtt-example.c 里面的 while 循环部分

### <a name="心跳和重连">心跳和重连</a>
心跳的时间间隔如何设置
---
在 [IOT_MQTT_Construct](#IOT_MQTT_Construct) 的入参中可以设置 `keepalive_interval_ms` 的值, C-SDK用这个值作为心跳间隔时间

keepalive_interval_ms 的取值范围是 60000 - 300000, 即最短心跳间隔为1分钟, 最长心跳间隔为5分钟

设备端是如何侦测到需要重连(reconnect)的
---
设备端C-SDK会在 `keepalive_interval_ms` 时间间隔过去之后, 再次发送 MQTT ping request, 并等待 MQTT ping response

如果在接下来的下一个 `keepalive_interval_ms` 时间间隔内, 没有收到对应的 ping response

或是在进行报文的上行发送(send)或者下行接收(recv)时发生异常, 则 C-SDK 就认为此时网络断开了, 需要进行重连

设备端的重连机制是什么
---
C-SDK 的重连动作是内部触发, 无需用户显式干预. 对于使用基础版的用户, 需要时常调用 [IOT_MQTT_Yield](#IOT_MQTT_Yield) 函数将 CPU 交给SDK, 重连就在其中发生

重连如果不成功, 会在间隔一段时间之后持续重试, 直到再次连接成功为止

相邻的两次重连之间的间隔时间是指数退避+随机数的关系, 比如间隔1s+随机秒数, 间隔2s+随机秒数, 间隔4s+随机秒数, 间隔8s+随机秒数... 直到间隔达到了间隔上限(默认60s)

### <a name="关于 IOT_MQTT_Yield">关于 IOT_MQTT_Yield</a>
IOT_MQTT_Yield 的作用
---
[IOT_MQTT_Yield](#IOT_MQTT_Yield) 的作用主要是给SDK机会去尝试从网络上接收数据

因此在需要接收数据时(subscribe/unsubscribe之后, publish之后, 以及希望收到云端下推数据时), 都需要主动调用该函数

IOT_MQTT_Yield 参数 timeout 的意义
---
[IOT_MQTT_Yield](#IOT_MQTT_Yield) 会阻塞住 timeout 指定的的时间(单位毫秒)去尝试接收数据, 直到超出这个时间, 才会返回调用它的函数

IOT_MQTT_Yield 与 HAL_SleepMs 的区别
---
都会阻塞一段时间才返回, 但是 [IOT_MQTT_Yield](#IOT_MQTT_Yield) 实质是去从网络上接收数据

而 [HAL_SleepMs](#HAL_SleepMs) 则是什么也不做, 单纯等待入参指定的时间间隔过去后返回

### <a name="订阅相关">订阅相关</a>
如果订阅了多个topic, 调用一次IOT_MQTT_Yield, 可能接收到多个topic的消息吗
---
调用一次 [IOT_MQTT_Yield](#IOT_MQTT_Yield), 如果多个topic都被订阅成功并且都有数据下发, 则可以一次性接收到来自多个topic的消息

什么情况下会发生订阅超时
---
+ 调用 [IOT_MQTT_Construct](#IOT_MQTT_Construct) 和云端建立MQTT连接时, 其入参中可以设置请求超时时间 `request_timeout_ms` 的值
+ 如果两倍 `request_timeout_ms` 时间过去, 仍未收到来自云端的 `SUBACK` 消息, 则会触发订阅(Subscribe)超时
+ 超时的事件会通过 [IOT_MQTT_Construct](#IOT_MQTT_Construct) 入参中设置的 `event_handle` 回调函数通知到用户
+ 在调用 [IOT_MQTT_Subscribe](#IOT_MQTT_Subscribe) 之后, 需要尽快执行 [IOT_MQTT_Construct](#IOT_MQTT_Construct) 以接收SUBACK, 请勿使用 [HAL_SleepMs](#HAL_SleepMs)

### <a name="发布相关">发布相关</a>
发布(Publish)的消息最长是多少, 超过会怎么样
---
+ C-SDK的代码上来说, MQTT的消息报文长度, 受限于 [IOT_MQTT_Construct](#IOT_MQTT_Construct) 入参中 `write_buf` 和 `read_buf` 的大小
+ 从云端协议上来说, MQTT消息报文长度不能超过256KB, 具体查阅 [官网限制说明页面](https://help.aliyun.com/document_detail/30527.html) 为准
+ 如果实际上报消息长度大于C-SDK的限制, 消息会被丢掉

被发布消息payload格式是怎么样的
---
阿里云物联网平台并没有指定pub消息的payload格式

需要客户根据应用场景制定自己的协议, 然后以JSON格式或其它格式, 放到pub消息载体里面传给服务端

## <a name="B.6 高级版问题">B.6 高级版问题</a>

## <a name="B.7 云端/协议问题">B.7 云端/协议问题</a>
物联网平台云端是否一有消息就立刻推送给订阅的设备而不做保存
---
消息一到达云端, 就会分发给不同的设备订阅者, 云端服务器不进行保存, 目前不支持 MQTT 协议中的 will 和 retain 特性

MQTT长连接时, 云端如何侦测到设备离线的
---
云端会根据用户调用 [IOT_MQTT_Construct](#IOT_MQTT_Construct) 时传入的 `keepalive_interval_ms`, 作为心跳间隔, 等待 MQTT ping request

如果在约定的心跳间隔过去之后, 最多再等5秒钟, 若还是没有收到 MQTT ping request, 则认为设备离线

## <a name="B.8 本地通信问题">B.8 本地通信问题</a>

## <a name="B.9 WiFi配网问题">B.9 WiFi配网问题</a>

## <a name="B.10 CoAP上云问题">B.10 CoAP上云问题</a>

### <a name="认证连接">认证连接</a>
CoAP上云连接的云端URI是什么
---
+ 在调用 [IOT_CoAP_Init](#IOT_CoAP_Init) 的时候, 可以设置其参数 `iotx_coap_config_t` 里面的 `p_url`
+ 如果 `p_url` 值为NULL, 则C-SDK会自动使用 `IOTX_ONLINE_DTLS_SERVER_URL` 这个宏所定义的默认URL

        #define IOTX_ONLINE_DTLS_SERVER_URL "coaps://%s.iot-as-coap.cn-shanghai.aliyuncs.com:5684"

+ 其中 `%s` 是使用 `p_devinfo` 里面的 `product_key`, 所以请确保在初始化 `iotx_coap_config_t` 的时候一定要对 `p_devinfo` 赋值

### <a name="报文上行">报文上行</a>
IOT_CoAP_SendMessage 发送的消息必须是JSON格式吗, 如果不是JSON会出现什么错误
---
+ 目前, 除了支持JSON格式外, 也可以支持CBOR格式
+ 因为是与云端通信, 需要使用指定格式, 否则可能会出现无法解析的问题

### <a name="关于 IOT_CoAP_Yield">关于 IOT_CoAP_Yield</a>
如何设置 IOT_CoAP_Yield 处理结果最大等待时间
---
目前默认设置是2000ms, 可以修改 `COAP_WAIT_TIME_MS` 这个宏进行调整

## <a name="B.11 HTTP上云问题">B.11 HTTP上云问题</a>
### <a name="认证连接">认证连接</a>
HTTPS进行设备认证时, Server会返回的错误码及其含义
---
+ **10000:** common error(未知错误)
    - HTTPS报文是有一定格式要求, 必须符合要求server才能支持
    - `ContentType`只支持"application/json"
    - 只支持HTTPS
    - 只支持POST方法
+ **40000:** request too many(请求次数过多, 流控限制)
    + 同一个设备在一天内的认证次数是有限制的
    + 解决方法: 每次认证获得的token是有48小时有效期的, 过期前可以反复使用, 无需每次都去认证获取新的token
+ **400:** authorized failure(认证失败)
    + 服务器认为鉴权参数是不合法的, 鉴权失败
    + 解决方法: 检查I`OTX_PRODUCT_KEY`, `IOTX_DEVICE_NAME`, `IOTX_DEVICE_SECRET`, `IOTX_DEVICE_ID`是不是从控制台获得的正确参数
