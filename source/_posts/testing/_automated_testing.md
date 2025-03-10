---
title: pytest+request+allure搭建接口自动化测试框架
date: 2024-8-20 09:58:30
categories:
  - testing
tags:
  - python
  - selenium
  - pytest
  - dtt
  - automation
  - docker
  - page object
---

TODO~

<more>

在设计一个 pytest + requests + Allure + Excel + 关键字驱动 的接口自动化测试框架时，各层的依赖关系需要从底层到上层逐步构建。以下是分层的排序和依赖关系：

1. 配置层 (Config Layer)
定位：最底层，为整个框架提供基础配置支持。
功能：管理全局配置，如环境变量、接口地址、数据库连接等。
依赖：无依赖，是其他层的基础。
示例：
配置文件（config.py 或 YAML 文件或env文件）。
环境变量管理工具。

2. 测试数据层 (Data Layer)
定位：依赖于配置层，为框架提供数据支持。
功能：管理测试数据，支持数据驱动测试。
依赖：配置层（如读取环境变量或文件路径）。
示例：
Excel 文件（存储测试用例和参数）。
数据读取工具（如 openpyxl 或 pandas）。

3. 核心业务层 (Core Layer)
定位：依赖于配置层和数据层，提供核心工具和业务逻辑支持。
功能：封装通用的工具方法（如 HTTP 请求、数据校验、日志记录）和业务逻辑（如用户管理、订单管理）。
依赖：配置层（如读取接口地址）、数据层（如获取测试数据）。
示例：
请求工具（封装 requests 库）。
校验工具（封装断言逻辑）。
日志工具（记录测试执行过程）。

4. 关键字驱动层 (Keyword Layer)
定位：依赖于核心业务层，提供关键字支持。
功能：将接口请求和业务操作封装为关键字，支持关键字驱动测试。
依赖：核心业务层（如调用请求工具、校验工具）。
示例：
关键字库（如 login、create_order）。
关键字解析器（解析测试数据中的关键字并执行）。

5. 测试用例层 (Test Case Layer)
定位：依赖于关键字驱动层和数据层，定义和组织测试用例。
功能：使用 pytest 编写测试用例，调用关键字执行业务逻辑。
依赖：关键字驱动层（如调用关键字）、数据层（如获取测试数据）。
示例：
测试用例文件（使用 pytest 编写）。
测试夹具（Fixture，提供前置条件和后置操作）。

6. 报告层 (Report Layer)
定位：最上层，依赖于测试用例层，生成测试报告。
功能：使用 Allure 生成测试报告，提供测试结果的可视化。
依赖：测试用例层（如记录测试执行过程）。
示例：
Allure 报告工具（生成 HTML 报告）。
报告增强工具（添加截图、日志等）。

配置层 (Config Layer)
       ↓
测试数据层 (Data Layer)
       ↓
核心业务层 (Core Layer)
       ↓
关键字驱动层 (Keyword Layer)
       ↓
测试用例层 (Test Case Layer)
       ↓
报告层 (Report Layer)




参考资料：
https://blog.csdn.net/qq_48811377/article/details/139423168 pom设计模式
https://www.bilibili.com/video/BV1iV411G7Ya?spm_id_from=333.788.videopod.episodes&vd_source=aaf34ccf61cf5005766e1ee9255c16a5&p=3 视频参考
https://zhuanlan.zhihu.com/p/528071986 关键字+数据驱动
https://www.cnblogs.com/juno3550/p/14431902.html 关键字+数据驱动
https://blog.csdn.net/ouyicnm?type=blog project
https://blog.csdn.net/zhangsiyuan1998/article/details/135201177 allure标记说明
https://blog.csdn.net/qq_43675178/article/details/131871213 pytest-xlsx
https://pypi.org/project/pytest-xlsx/ 文档
https://zhuanlan.zhihu.com/p/87203290 faker
https://blog.csdn.net/u011397981/article/details/129992968 JMeter
https://blog.csdn.net/m0_47747596/article/details/131658904 JMeter1 jmeter --nongui -t ./testplan.jmx -l test_result.jtl -e -o ./jmeter_reports/
https://blog.csdn.net/RoninYang/article/details/107997794 jmeter转换
https://www.cnblogs.com/yoyoketang/p/14108144.html pytest-hooks详解
https://pytest-xlsx-doc.pages.dev/getting-started pytest-xlsx
https://hellowac.github.io/pytest-zh-cn/how-to/writing_hook_functions.html#declaringhooks pytest中文文档
https://blog.csdn.net/fallenjency/article/details/124631757 pytest+requests+Excel+allure接口自动化测试框架实践