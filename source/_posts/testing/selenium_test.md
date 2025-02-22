---
title: selenium+python+docker搭建自动化测试
date: 2024-7-20 09:58:30
categories:
  - testing
tags:
  - python
  - selenium
---

Selenium Grid是Selenium套件的一部分，提供了在多台机器，不同浏览器，跨平台运行测试的能力。在web中要做好自动化测试，selenium是非常适配的选择。

<!-- more -->

# 一、Selenium Grid 简介

Selenium Grid是Selenium套件的一部分，提供了在多台机器，不同浏览器，跨平台运行测试的能力。在web中要做好自动化测试，selenium是非常适配的选择。

## 1.1 简介

Selenium Grid是Selenium套件的一部分，提供了在多台机器，不同浏览器，跨平台运行测试的能力。

### grid组件

grid由路由器router、分发器distributor、会话映射session map、新会话队列new session queue、节点node、事件总线event bus组成。
架构如下图所示：
![](/pics/testing/selenium/selenium_components.png)

#### 路由器router

作用：grid的入口点，接收所有外部请求，转发给正确的组件。
功能：
- 处理新的会话请求，转发到新会话队列。
- 查询会话映射，转发请求到正确节点
- 平衡负载

#### 分发器distributor

作用：
- 注册跟踪所有的node以及其功能
- 查询新会话队列并处理任何待处理的新会话请求。
功能：
- 注册节点跟踪其功能
- 查询新会话队列，处理新会话请求，分配给对应节点
- 通过gridmodel跟踪节点功能


#### 会话映射session map

作用：
- 保存会话id和节点的关系
- 路由器在会话映射中查询确认会话id与关联节点。

#### 新会话队列new session queue

作用：
- 先进先出顺序保存新会话请求
- 可配置请求超时和重试间隔
功能：
- 新会话请求超时的会被拒绝和删除
- 轮训队列，方便将请求分配给可用的节点插槽
- 节点插槽决定了一个节点可以同时运行多少个测试用例

#### 节点node

作用：
- 一个grid可以包含多个node，Node管理它所在机器上可用浏览器的插槽
功能：
- Node通过事件总线向分发器注册自己，报告自己的配置
- 自动注册可用的浏览器驱动
- 创建浏览器插槽，执行接收到的命令

#### 时间总线event bus
作用：
- 充当各组件之间的通信路径
功能：
- 处理节点注册事件和其他组件内部通信
- 通过消息进行通信，避免http通信消耗

# 二、docker部署hub&nodes

selenium的官方docker hub提供了hub,node-chrome,node-firefox节点
https://hub.docker.com/r/selenium/hub
https://hub.docker.com/r/selenium/node-chrome
https://hub.docker.com/r/selenium/node-firefox


如果遇到网络问题，需要配置docker镜像，[请点击这里查看docker篇3.3小节](/cicd/basic_docker/#3-3-docker源修改)

**我们先看看怎么镜像标签怎么看解读**


hub标签：selenium/hub-\<Major\>.\<Minor\>.\<Patch\>-\<YYYYMMDD\>
```
Selenium Server 4.9.0
Release date 20230426

e126989f151e        selenium/hub   4
e126989f151e        selenium/hub   4.9
e126989f151e        selenium/hub   4.9.0
e126989f151e        selenium/hub   4.9.0-20230426
```

node-chrome标签：selenium/node-chrome-\<browserVersion\>-\<browserDriver\>-\<browserDriverVersion\>-\<Major\>.\<Minor\>.\<Patch\>-\<YYYYMMDD\>
其他浏览器同理，作者只是按照个人习惯会挑完整的标签，这个不强制
```
Chrome 112.0
ChromeDriver 112.0
Selenium Server 4.9.0
Release date 20230426

e126989f151e        selenium/node-chrome   4
e126989f151e        selenium/node-chrome   4.9
e126989f151e        selenium/node-chrome   4.9.0
e126989f151e        selenium/node-chrome   4.9.0-20230426
e126989f151e        selenium/node-chrome   112.0                  
e126989f151e        selenium/node-chrome   112.0-20230426         
e126989f151e        selenium/node-chrome   112.0-chromedriver-112.0 
e126989f151e        selenium/node-chrome   112.0-chromedriver-112.0-20230426
e126989f151e        selenium/node-chrome   112.0-chromedriver-112.0-grid-4.9.0-20230426  
```

**创建docker网络启动镜像**

docker创建网络，hub和nodes基于这个网络进行启动

```
$ sudo docker network create selenium-grid-network
```

启动hub

```
$ sudo docker run -d \
-p 4442-4444:4442-4444 \
--net selenium-grid-network \
--name selenium-hub \
--restart=always \
selenium/hub:4.28.1-20250202
```

启动node-chrome容器
```
$ sudo docker run -d --net selenium-grid-network -e SE_EVENT_BUS_HOST=selenium-hub \
    --shm-size="1g" \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    --name node-chrome \
    --restart=always \
    selenium/node-chrome:132.0.6834.159-chromedriver-132.0.6834.159-grid-4.28.1-20250202
```

启动node-firefox容器
```
$ sudo docker run -d --net selenium-grid-network --shm-size="1g" \
 -e SE_EVENT_BUS_HOST=selenium-hub \
-e SE_EVENT_BUS_PUBLISH_PORT=4442 \
-e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
--name node-firefox \
--restart=always \
selenium/node-firefox:129.0-geckodriver-0.35-grid-4.28.1-20250202 
```

检查一下容器运行情况

```
$ sudo docker ps -a
 sudo docker ps -a
CONTAINER ID   IMAGE                                                                                  COMMAND                  CREATED         STATUS                  PORTS                                                           NAMES
c1cbb0187194   selenium/node-firefox:129.0-geckodriver-0.35-grid-4.28.1-20250202                      "/opt/bin/entry_poin…"   3 minutes ago   Up 3 minutes            5900/tcp, 9000/tcp                                              node-firefox
13131972b822   selenium/node-chrome:132.0.6834.159-chromedriver-132.0.6834.159-grid-4.28.1-20250202   "/opt/bin/entry_poin…"   7 minutes ago   Up 7 minutes            5900/tcp, 9000/tcp                                              node-chrome
39eb162b7ef0   selenium/hub:4.28.1-20250202                                                           "/opt/bin/entry_poin…"   2 hours ago     Up 52 minutes           0.0.0.0:4442-4444->4442-4444/tcp, :::4442-4444->4442-4444/tcp   selenium-hub
```

检查grid的ui页面，se给我们的grid提供了ui页面，作者搭建的时候是在虚拟机中的docker.
**作者本地**linux虚拟机地址是192.168.174.128，看官们根据自己的实际地址进行测试。**在虚拟机中搭建docker，再基于docker搭建selenium grid是为了模拟服务器搭建**
只要打开192.168.174.128:4444就能进入到se grid管理页面

![](/pics/testing/selenium/selenium_1.png)


# 三、客户端编写代码与测试

基于上一篇的[python嵌入式开发环境搭建](/python/python_embedable)，准备好基础环境之后，我们要安装selenium相关的库了。

安装selenium

```
$ cd path/to/python-3.13.2-embed-amd64
$ ./python.exe -m pip install selenium -t ./Lib/site-packages
```

code/\__main\__.py

```
from selenium import webdriver  

def get_default_chrome_options():
    options = webdriver.ChromeOptions()
    options.add_argument("--no-sandbox")
    return options

options = get_default_chrome_options()

driver = webdriver.Remote(  
    command_executor='http://192.168.174.128:4444',  
    options=options  
)

driver.get("https://www.baidu.com")  
print(driver.title)  
driver.quit()  
```

按下ctrl + f5执行代码，大概过了6s，就看到终端打印了baidu首页的title

```
E:\test_learning\selenium_test>  e:; cd 'e:\test_learning\selenium_test'; & 'e:\test_learning\selenium_test\python-3.13.2-embed-amd64\python.exe' 'c:\Users\89504\.vscode\extensions\ms-python.debugpy-2025.0.1-win32-x64\bundled\libs\debugpy\launcher' '5234' '--' 'E:\test_learning\selenium_test/code/__main__.py'
百度一下，你就知道
```


参考资料：
https://www.selenium.dev/zh-cn/documentation/grid/ grid功能
https://blog.csdn.net/TestLiChangAn/article/details/137156566 grid介绍
https://www.oryoy.com/news/docker-huan-jing-xia-kuai-su-da-jian-selenium-zi-dong-hua-ce-shi-fu-wu-qi-shi-jian-zhi-nan.html grid搭建
https://blog.csdn.net/u013948010/article/details/78539215 测试用例参考
https://hub.docker.com/u/selenium
https://hub.docker.com/r/selenium/hub
https://hub.docker.com/r/selenium/node-chrome
https://hub.docker.com/r/selenium/node-firefox
https://github.com/SeleniumHQ/docker-selenium/blob/trunk/ENV_VARIABLES.md 环境变量
https://github.com/SeleniumHQ/seleniumhq.github.io/blob/trunk/examples/python/tests/drivers/test_remote_webdriver.py#L68 python se远程