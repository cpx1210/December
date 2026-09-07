# 北京交通大学智慧高铁系统前沿科学中心存在前台SQL注入漏洞

2026/8/13

## 1.漏洞概述：

目标站点运行 WordPress 6.9.4，存在 CVE-2026-63030（wp2shell）路由混淆漏洞。攻击者无需任何认证，通过向 /wp-json/batch/v1 发送特制的双层嵌套批处理请求，可绕过 REST API 路由校验，，实现未授权布尔盲注。

## 2.利用过程

探测漏洞是否存在，发现响应 HTTP 207，同时出现三个标记，路由混淆存在

![](attachment/1.png)

布尔盲注True：

![](attachment/2.png)

X-WP-Total回显1293

布尔盲注False：

![](attachment/3.png)

X-WP-Total回显28，找到了布尔盲注真假值的回显区别，以下以注出库名为演示

1. 探测数据库长度，length=9时回显1293，说明库名长度为9


![](attachment/4.png)

2.用布尔盲注二分法确认库名各字符acsii值，以下是第一位，当为101时X-WP-Total回显1293，说明第一位是e

![](attachment/5.png)

逐字符探测，最终得到数据库名为exampledb

同样方法探测管理员账号名：

首先验证长度

![](attachment/6.png)

长度为4

提取第一个字符

![](attachment/7.png)

asc为116，是t。

逐字符探测，得到管理员账户名为test

## 3.修复建议

升级 WordPress：从 6.9.4 升级到 6.9.5+，修复 CVE-2026-63030 路由混淆漏洞

禁用 REST API 批处理：add\_filter('rest\_allow\_batch', '\_\_return\_false');

​
