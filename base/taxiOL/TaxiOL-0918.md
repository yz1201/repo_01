# TaxiOL-0918-Base

###全文背诵

~~~html
执行目标：
	独立完成注册中心优化，目的是节省服务的上下线时间，降低无效服务的调用率，提高接口调用的成功
	代码review，验证码十倍效率写法
	指导别人，不变的常用的，少走db，善用缓存 IO瓶颈：网络，磁盘，尽量减少
	提高qps两大诀窍：提高io瓶颈
~~~

~~~html
登录业务处理
	注意生产环境，pom文件中版本的选择，快照版本不能上生产，因为有可能会去拉取更新，导致不必要的错误。
~~~

### 线程数估算？

~~~html
线程数= cpu可用核心数/（1-阻塞系数）  --阻塞系数，io密集型接近1，计算密集型cpu接近0。
计算这个的目的也是要提高系统的qps。
提高qps两大诀窍：
	提高并发数
		1，多线程
		2，增加连接数 mysql/redis
		3，服务无状态，便于横向扩展
		4，让服务能力对等，service-url打乱顺序，不要只让某个server节点负载过大。
    减少响应时间
		1，异步响应（只需保证最终一致性的东西，不需要等到执行完毕就先返回响应信息）流量削峰
		2，缓存技术，减少db访问，减少磁盘io，读多写少。
		3，数据库优化
		4，多的数据，分批次返回 |？
		5，减少调用链 
		6，善用长连接，不要轮询
-- BOSS Business Operation Support System
~~~

###接口设计

~~~
api-passemdger发送验证码
url: sms/verify-code/send
params
{
  "phoneNumber":"159*******4"
}
service-verification-code 生成验证码
url: verify-code/generate/{身份：乘客，司机}/(手机号)12345678911

service-sms 发送短信 （通用。）
url：/send/sms-template
params:
{
    "receivers":["phoneNumber",...],
    "data":[
       {
           "id":"SMS_144145499"
           "templateMap":{
               "code":"018900"
            }         
       }
   ]
}
短信模板id：短信模板：您好，短信验证码是${code}，2分钟内有效
响应统一返回参数
{
  "code":"10001",
  "message":"success",
  "data":T
}
~~~

