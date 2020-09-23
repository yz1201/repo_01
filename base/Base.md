# CAR 隔离系统的变化点

## 服务拆分

#### 业务层

|   模块   |      项目名      |     描述     |
| :------: | :--------------: | :----------: |
|  乘客端  |  api-passenger   |    乘客端    |
|  司机端  |    api-driver    |    司机端    |
| 司机听单 | api-listen-order | 司机等待接单 |

#### 能力层

|     模块     |          项目名           |     描述     |
| :----------: | :-----------------------: | :----------: |
|   app升级    |      service-update       |   应用升级   |
|     订单     |       service-order       |   订单模块   |
|     派单     |  service-order-dispatch   |   派单模块   |
| 乘客用户管理 |  service-passenger-user   | 乘客用户中心 |
|     短信     |        service-sms        |   短信服务   |
|     计价     |     service-valuation     |   估价服务   |
|    验证码    | service-verification-code |  验证码服务  |
|     钱包     |      service-wallet       |   钱包模块   |
|     支付     |      service-payment      |   支付模块   |

### spring cloud基础服务

|   模块   |         项目名          |
| :------: | :---------------------: |
| 注册中心 |   cloud-eureka/nacos    |
| 配置中心 |   cloud-config-server   |
|   网关   |   cloud-zuul/gateway    |
| 熔断监控 | cloud-hystrix-dashboard |
| 健康检查 |       cloud-admin       |
| 链路追踪 |     cloud-zipkin-ui     |

### 基础common

|           模块            |     项目名      |
| :-----------------------: | :-------------: |
| 通用bean/工具类/异常/校验 | internal-common |

### 涉及到的技术栈

```html
boot cloud maven git mysql redis mq
第三方：
	短信服务（腾讯，阿里短信，华信）
	语音服务： 隐私号，（乘客和司机订单匹配后，A B，号码都为X，但可以录音）
	文件服务：阿里oss
	地图服务：高德地图
	消息推送：极光，透传，通知
	支付：vx zfb
	航旅纵航：查航班
	发票：百望云
能力层，qps：2000（配置，4g8core） 有些300.
```

### 接口设计

```html
意义：接口确定之后，app后端才能同时开发，后端确定接口
restful风格（资源表现层状态转移），http请求。重在资源
协议：https ios只能https
域名：/restapi.domain.com/
版本：v1
路径：/xxoo/xxoo/（restful风格要求uri路径是名词，尽量）
动作：
	post：新建
	put：修改（修改后的全量数据）
	patch：修改（修改哪个传哪个）
	delete：删除
	get：查询
```

### 接口安全

```html
CIA：
	保密性 加密传输 三级等保，手机号，身份证号，脱敏。传输中存储加密
	完整性 数据不丢失，无法被篡改/篡改无效
	可用性 恶意攻击的处理拦截。
数据层面：
	sql注入：select * from table where id = (变量；delete table)  #/$ $看情况用，用不好会出现sql注入问题。接口层面的处理就是要过滤一些数据，用jsoup做接口的参数过滤，把非法字符都过滤了。

	xss：XSS攻击通常指的是通过利用网页开发时留下的漏洞，通过巧妙的方法注入恶意指令代码到网页，使用户加载并执行攻击者恶意制造的网页程序。这些恶意网页程序通常是JavaScript，但实际上也可以包括Java、 VBScript、ActiveX、 Flash 或者甚至是普通的HTML。攻击成功后，攻击者可能得到包括但不限于更高的权限（如执行一些操作）、私密网页内容、会话和cookie等各种内容。
	在正常用户请求中执行了黑客提供的恶意代码，问题出在，用户数据没有过滤，转义。
jsoup里的xss whitelist（白名单）有六种方法，一个构造和5种静态方法。

	csrf：跨站伪造请求。人机交互，token。
	冒充别人的登录信息，没有防范不信任的调用。
	owasp-java-html-sanitizer
	[Java对html标签的过滤和清洗]: https://www.cnblogs.com/qizhelongdeyang/p/9884716.html
	
	Referer：防盗链

	数据权限控制：
		link1链接 
			A用户请求，删除order/a1 
			B用户请求，删除order/a1
	
```











