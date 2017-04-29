- [一、缩写解释](#一缩写解释)
- [二、背景知识](#二背景知识)
    - [1. 非对称加密](#1-非对称加密)
    - [3. 数字证书（X.509）](#3-数字证书x509)
- [三、什么是HTTPS](#三什么是https)
    - [1. SSL/TLS](#1-ssltls)
    - [2. HTTPS](#2-https)
    - [3. HTTPS的主要流程](#3-https的主要流程)
    - [4. HTTPS特性小结](#4-https特性小结)
    - [5. HTTPS的缺点](#5-https的缺点)
- [四、应用](#四应用)
    - [1. 获得证书](#1-获得证书)
    - [2. 使用证书](#2-使用证书)


## 一、缩写解释
缩写 | 解释 
--|--
HTTP|HyperText Transfer Protocol
HTTPS|HTTP Over SSL
SSL|Secure Sockets Layer
TLS|Transport Layer Security
CA|Certification Authorities


## 二、背景知识
### 1. 非对称加密
- **概念**：非对称加密算法需要__两个密钥__来分别进行加密和解密
    - 公钥： 非对称加密中的__公开的密钥__
    - 私钥： 非对称加密中的__私有的密钥__（绝对不能泄露）
- **特性**： 公钥和私钥可以__双向加解密__
- **应用**： 非对称加密在通讯中的应用：
    - 公钥加密内容，私钥解密内容
    - 私钥加密签名，公钥解密签名
    
- **举个栗子**：
    - **场景**：A写信给B，
    - **过程**：
    ```sequence
    A->>A: 创建信件内容
    A->>A: 使用B的公钥加密信件内容
    A->>A: 给信件添加签名信息
    A->>A: 使用自己的私钥加密签名信息
    A->>B: 寄送信件给B
    B->>B: 使用自己的私钥解密信件内容
    B->>B: 用A的公钥解密签名信息
    B->>B: 验证签名信息
    ```

    - **结果**
        - 信件是__内容隐秘__的，只有持有B的私钥才能获取内容
        - B可以通过验证签名__确定信息的来源__，确保不是伪造的信件
        - A如果在签名中加入内容的hash值，B就可以通过对内容做hash来确保__内容完整__
    

### 3. 数字证书（X.509）

- **CA和数字证书**
    - CA是证书颁发机构
    - CA可以用自己的证书为用户颁发新的数字证书
    - 顶级的证书颁发机构，叫 Root CA
    - Root CA持有的证书称为__根证书__
    - 数字证书在私钥没有泄露的前提下，是__无法伪造__的
    - 每一个数字证书都有相应CA的信息，CA的证书包含上级CA的信息，一直到Root CA，形成证书链
    - 主流操作系统和浏览器一般都内置（信任）__权威CA__的根证书
    
- **数字证书的主要购成**：
    - CA的信息
    - 持有者信息
    - 持有者的公钥
    - 持有者信息的数字签名指纹
    - 有效期

- **颁发数字证书**：
    - CA验证申请者信息的合法性
    - 将申请者的签名信息做hash采样，再使用自己的私钥加密hashcode，得到数字签名指纹
    - 将数字签名指纹，以及其他信息合并，再使用自己的私钥加密（防篡改）
    - 添加CA信息，合并生成数字证书
    
- **客户端验数字证书**
    - 查询CA信息
    - 获取CA公钥（不受信任的CA无法获取到公钥）
    - 使用CA的__公钥__解密证书信息
    - 使用CA的__公钥__解密数字签名指纹，得到持有者签名信息
    - 验证持有者的签名信息

- **举个栗子**  
下图是百度的数字证书,
浏览器使用CA"Symantec Class 3..."的公钥对指纹（数字签名）进行解密，就可以验证持有者的信息是否真实  

![image](http://note.youdao.com/yws/api/personal/file/6278EA8ABFD643DAB8A1FB369D2DFEB2?method=download&shareKey=84c27ceb03b7f81ed3adb52604932a09)



## 三、什么是HTTPS

### 1. SSL/TLS
- SSL是上世纪90年代由网景公司设计
- 1999年IETF将SSL标准化，命名为TLS
- 位于OSI **应用层**和 __传输层__之间
- 用于两个应用程序之间提供__保密性__和__数据完整性__


### 2. HTTPS   
- HTTP位于OSI的应用层
- HTTPS = HTTP Over SSL/TLS



### 3. HTTPS的主要流程

- **单向认证**：仅客户端对服务器进行认证
```sequence
Client->>Server: 连接请求
Server->>Client: 服务器证书+公钥
Client->>Client: 验证服务器证书签名
Client->>Client: 产生随机码
Client->>Server: 随机码（使用服务器公钥加密）
Server->>Server: 使用服务器私钥解密获得随机码

Client->>Server: 使用随机码做对称加密通讯
Server->>Client: 使用随机码做对称加密通讯
```

- **双向认证**： 客户端与服务器互相认证
```sequence
Client->>Server: 连接请求
Server->>Client: 服务器证书+公钥
Client->>Client: 验证服务器证书签名
Note Right of Client:认证客户端开始
Client->>Server: * 客户端证书+公钥
Server->>Server: * 验证客户端证书签名
Server->>Client: * 加密方案（使用客户端公钥加密）
Client->>Client: * 使用客户端私钥解密获得加密方案
Note Right of Client: 认证客户端结束
Client->>Client: 产生随机码
Client->>Server: 随机码（使用服务器公钥加密）
Server->>Server: 使用服务器私钥解密获得随机码

Client->>Server: 使用随机码做对称加密通讯
Server->>Client: 使用随机码做对称加密通讯
```

### 4. HTTPS特性小结
- 通过数字证书技术，确保了网站身份的__真实性__
- 通过非对称加密技术，保证了对称加密密码的__安全性__
- 通过对称加密技术，保证了传输内容的__保密性__、__防篡改性__，同时保证了一定的__性能__

### 5. HTTPS的缺点
- **性能影响**
HTTPS因为建链时的握手流程复杂，传输需要加解密，导致服务端增加内存、CPU、网络带宽的开销，有人测试过一次响应时间差别在几十毫秒的级别，不过这个要看具体数据包大小、加密算法、服务器性能和安全级别等因素
- **证书收费**
现在市场上的证书根据级别，授权域名数量，年收费从几百~上万价格不等，参考国内授权商[wosign](http://www.wosign.com/price.htm)的价格  
可以免费申请的DV（Domain Validation）证书，只进行普通的域名验证和传输加密，对机构身份和其他信息不进行验证
- **非绝对安全**
HTTPS的安全性建立在SSL协议，加密算法和服务器安全的基础上，它们中任何一个出现问题，都有可能导致HTTPS安全性失效


### 6. HTTPS趋势
- 2017年4月的谷歌搜索数据，HTTPS首页占比达到50%（9个月前为30%）
- Chrome浏览器（市占接近60%）从2016年开始将所有HTTP网站标记为不安全
- 目前国外大多数主流网站都已经开启了全站HTTPS加密，如Google、微软、Facebook、Amazon等
- 国内的一些大网站也开启了全站HTTPS（淘宝，支付宝，各银行网银），部分网站在敏感页面开启了强制HTTPS（登陆，重要数据，授权API等）


## 四、应用
### 1. 获得证书
- **流程**
	- **CA签名证书**：由权威机构签名，收费证书，可以用来验证身份，可以传输加密，大部分互联网网站就是使用的这种签名
  - **自签名**：使用openssl等工具，自行生成的根证书签名，需要用户手动导入信任根证书，否则只能使用传输加密的功能，主要用于有加密需求，没有验真需求的场景，如局域网
```flow

st=>start: 开始
op1=>operation: 生成非对称公/私密钥对（*.key）
op2=>operation: 生成包含签名信息和公钥的签名请求文件（*.csr）
cond=>condition: CA签名 ?

op3=>operation: 将请求文件提交到权威CA进行签名
op4=>operation: 使用自签名的根证书进行签名
op5=>operation: 得到证书（*.crt）
e=>end: 结束

st->op1->op2->cond
cond(yes)->op3->op5->e
cond(no)->op4->op5->e
```
- **举个栗子**：使用openssl生成证书.pdf

### 2. 使用证书
- **使用主流的https library**: 主流支持https的library都已经实现了https的认证流程，用户只需要将密钥和证书加载到库中即可
- **举个栗子**
    - C/C++ - microhttpd
    ```c
    MHD_OptionItem options[SIZE];
    // 设置密钥文件
    options[0] = MHD_OptionItem{
                MHD_OPTION_HTTPS_MEM_KEY, intptr_t(0),
                static_cast<void*> (const_cast<char*> (KEY_FILE))
            }
    // 设置证书文件
    options[1] = MHD_OptionItem{
                MHD_OPTION_HTTPS_MEM_CERT, intptr_t(0),
                static_cast<void*> (const_cast<char*> (CERT_FILE))
            }

    
    // 启动deamon
    MHD_start_daemon(..., &options, ...)
    ```
    - Python - BaseHTTPServer，SimpleHTTPServer
    ```python
    import BaseHTTPServer, SimpleHTTPServer
    import ssl
    httpd = BaseHTTPServer.HTTPServer(('localhost', 443), 
        SimpleHTTPServer.SimpleHTTPRequestHandler)
    # pem是openssl工具生成的key+cert的组合文件
    httpd.socket = ssl.wrap_socket (httpd.socket, certfile='./server.pem',
        server_side=True)
    httpd.serve_forever()
    
    ```
    - Golang - net/http
    ```golang
  import (
	"log"
	"net/http"
    )
    
    func handler(w http.ResponseWriter, req *http.Request) {
    	w.Header().Set("Content-Type", "text/plain")
    	w.Write([]byte("This is an example server.\n"))
    }
    
    func main() {
    	http.HandleFunc("/", handler)
    	log.Printf("About to listen on 443. Go to https://127.0.0.1:443/")
    	err := http.ListenAndServeTLS(":443", "server.crt", "server.key", nil)
    	log.Fatal(err)
    }
    
    ```
        
