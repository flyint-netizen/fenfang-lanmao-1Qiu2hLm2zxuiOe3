
环境： window10\_x64 \& vs2022pjsip版本： 2\.14\.1python版本： 3\.9\.13


近期有关于windows环境下软电话sdk开发的需求，需要开发动态库给上层应用调用，今天整理下使用pjsip封装简单的自定义软电话sdk笔记，并提供相关资源下载。


我将从以下几个方面展开：


* 功能说明
* 接口设计
* 接口实现
* 接口调用示例 （python及c\#）
* 配套资源下载


pjsip编译及使用，可参考这篇文章：


[https://github.com/MikeZhang/p/18542615/pjsip\-vs2022](https://github.com)


## 一、功能说明


库名称： sipClient.dll功能：windows10环境下，封装pjlib库给上层应用，以便实现基于sip协议的软电话功能。


整体结构如下：


![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217221245172-528967497.png)


 实现原理：1）基于pjsip封装sdk；2）封装基础操作及回调函数；


![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217221333529-487620464.png)


完整代码可从如下渠道获取：


关注微信公众号（聊聊博文，文末可扫码）后回复 20241217 获取。
## 二、接口设计


### 1、init


**功能**


初始化接口，用于初始化dll库。


**接口声明**



```
int init(int sock_type, int local_port);
```

**入参**


sock\_type ： soket类型，设置如下1 \=\> udp2 \=\> tcp （后续支持）3 \=\> tls （后续支持）


local\_port ： 本地监听端口，设置为0则为随机端口


**返回值**


0 ： 成功\-1 ： 失败


### 2、set\_incoming\_cb


**功能**设置呼入回调函数，用于处理呼入请求。


**接口声明**



```
int set_incoming_cb(pCallBack cb);

```

**入参**


cb : 回调函数，格式如下



```
int(*pCallBack)(int, int);

```

**返回值**


0 ： 成功\-1 ： 失败


### 3、acc\_reg


**功能**账号注册操作，提供账号信息注册到指定服务器。


**接口声明**



```
int acc_reg(char* sip_domain, char* sip_user, char* sip_passwd,char* realm);

```

**入参**


sip\_domain ： 注册服务器


sip\_user ： 用户名


sip\_passwd ： 密码


realm ： 注册域


**返回值**


account id ： 成功\-1 ： 失败


### 4、acc\_unreg


**功能**账号注销操作，注销指定账号。


**接口声明**



```
int acc_unreg(int acc_id);

```

**入参**


acc\_id ：账号id


**返回值**


call id ： 成功\-1 ： 失败


### 5、acc\_make\_call


**功能**外呼操作，使用指定账号进行外呼。


**接口声明**



```
int acc_make_call(int acc_id, const char* dst_num);

```

**入参**


acc\_id ：账号iddst\_num ： 被叫号码


**返回值**


call id ： 成功\-1 ： 失败


### 6、acc\_answer\_call


**功能**


挂机操作，挂断指定呼叫。


**接口声明**



```
int acc_answer_call(int call_id);

```

**入参**


call\_id ： 需要接听的呼叫id值。


**返回值**


0 ： 成功\-1 ： 失败


### 7、acc\_hangup\_call


**功能**挂机操作，挂断指定呼叫。


**接口声明**



```
int acc_hangup_call(int call_id);

```

**入参**


call\_id ： 需要挂机的呼叫id值。


**返回值**


0 ： 成功\-1 ： 失败


### 8、destory


**功能**回收pjsua资源。


**接口声明**



```
int destory();

```

**入参**


无


**返回值**


0 ： 成功\-1 ： 失败


## 三、接口实现


 使用vs2022进行dll封装，配置类型设置为dll类型。
![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217221525498-1466051093.png)


**1、编写头文件；**


![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217221541511-345091892.png)


 完整内容可从如下渠道获取：


关注微信公众号（聊聊博文，文末可扫码）后回复 20241217 获取。
**2、编写cpp文件**

![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217221720412-1622981861.png)


完整内容可从如下渠道获取：


关注微信公众号（聊聊博文，文末可扫码）后回复 20241217 获取。


**3、编译成动态库**


在sipClient工程右键选中“生成”即可进行编译。




![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217222033074-1183594555.png)


 打包的vs2022工程文件及编译好的dll文件，可从如下渠道获取：


关注微信公众号（聊聊博文，文末可扫码）后回复 20241217 获取。


## 四、示例代码（python）


这里以python为例，展示下如何调用dll库。


需要注意下，这里的dll和python版本pjsua还是不一样的，对python版本无要求。


如需编译python版pjsua，可参考如下文章：


[https://github.com/MikeZhang/p/centos7py39pjsua20230608\.html](https://github.com):[PodHub豆荚加速器官方网站](https://rikeduke.com)


[https://github.com/MikeZhang/p/pyPjsuaExample20230623\.html](https://github.com)


### 1、来电处理


功能说明：1）注册账号；2）来电自动接听；3）等待特定时间后执行挂机操作；4）挂机操作；5）注销操作；


示例代码如下（autoAnswer1\.py）：






```
import time
from ctypes import *

pDll = CDLL("./sipClient.dll")

# typedef int (*Func)(int, int);
cbType = CFUNCTYPE(c_int, c_int, c_int)

def incoming_callback(acc_id, call_id):
    print("python callback called >>>>>>>>>>>>>>>>")
    print("acc_id : {} , call_id : {}".format(acc_id, call_id))
    pDll.acc_answer_call(call_id)
    time.sleep(30)
    pDll.acc_hangup_call(call_id)
    return 0

pDll.init(1,0)
incoming_cb = cbType(incoming_callback)
pDll.set_incoming_cb(incoming_cb)

reg_host=b"192.168.137.100:5060"

domain=create_string_buffer(reg_host)
user=create_string_buffer(b"1002")
passwd=create_string_buffer(b"0000")
realm=create_string_buffer(b"*")

acc_id = pDll.acc_reg(domain,user,passwd,realm)

print("acc_id : ",acc_id)

time.sleep(1)
print("wait call")
input("#############################\n")

pDll.acc_unreg(acc_id)

time.sleep(1)
pDll.destory()
```


注册效果如下：


![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217222220788-670390205.png)


 自动接听效果如下：


![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217222238382-1990991261.png)


## 2、手动外呼


功能说明：1）注册账号；2）主动发起外呼请求；3）挂机操作；4）注销操作；


示例代码如下（manualCallout1\.py）：






```
import ctypes,time

pDll = ctypes.CDLL("./sipClient.dll")

pDll.init(1,0)

reg_host=b"192.168.137.100:5060"

domain=ctypes.create_string_buffer(reg_host)
user=ctypes.create_string_buffer(b"1002")
passwd=ctypes.create_string_buffer(b"0000")
realm=ctypes.create_string_buffer(b"*")

acc_id = pDll.acc_reg(domain,user,passwd,realm)

print("acc_id : ",acc_id)

input("#############################")

#dst_uri = ctypes.create_string_buffer(b"sip:1000@" + reg_host)
dst_num = ctypes.create_string_buffer(b"1000")
callid = pDll.acc_make_call(acc_id,dst_num)

input("#############################")

pDll.acc_hangup_call(callid)

input("#############################")

pDll.acc_unreg(acc_id)

time.sleep(1)
pDll.destory()
```


呼叫效果如下：


![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217222325749-1337449850.png)


 接听效果如下：


![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217222341027-1259055102.png)


 可运行的python脚本及dll文件，可从文末提供的渠道获取。


## 五、示例代码（c\#）


这里参考python示例代码，用c\#展示如何调用sipClient库。


### 1、来电处理


功能说明：1）注册账号；2）来电自动接听；3）等待特定时间后执行挂机操作；4）挂机操作；5）注销操作； 


示例代码如下（csAutoAnswer1）：




![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217222519351-937888302.png)


完整内容可从可从文末提供的渠道获取。


运行效果如下：


![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217222625490-830641521.png)


 对应的工程文件（csAutoAnswer1\.zip）可从文末提供的渠道获取。


## 2、手动外呼


功能说明：1）注册账号；2）主动发起外呼请求；3）挂机操作；4）注销操作；


示例代码如下（csManualCallout1）：




![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217222716360-753440642.png)


完整内容可从可从文末提供的渠道获取。


 外呼效果如下：


![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217222737928-1026244127.png)


 对应的工程文件（csManualCallout1\.zip）可从文末提供的渠道获取。


## 六、资源下载


本文涉及源码及相关文件，可从如下途径获取：




关注微信公众号（聊聊博文，文末可扫码）后回复 20241217 获取。
![](https://img2024.cnblogs.com/blog/300959/202412/300959-20241217222817211-1952413942.png)


 



