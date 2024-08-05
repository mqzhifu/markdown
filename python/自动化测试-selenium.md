# 自动化测试/selenium

1. 性能测试
    jemeter loadRunner,locust
2. 功能测试
    robotFramework
    AirTest
    Playwright
    seleniumIDE
    cypress\(弃了，nodejs编写\)

自动化测试是针对部分业务，而不是全链路，其主要是为：核心业务。并且可标准化流程的，或者人为操作即期复杂的情况。

自动化测试的衡量标准是：覆盖系统的比重是多少

大的分层

1. UI
2. SERVER
3. UNIT

绝对不是线性代码形态，以代码与数据分离形态来实现。关键字驱动\+POM

selenium4 \+ webDrive \+ seleniumGrid

chrom 最好别自动 更新，webDrive依赖

> pip install selenium

web drive

https://selenium.dev/

下包，解压，CP

> cp /data/www/python/zpy/chromedriver /usr/local/Cellar/python@3.9/3.9.6/Frameworks/Python.framework/Versions/3.9

chown wangdongyan:staff chromedriver

这里不CP也行，在PY代码里指定文件目录也行

也可以本地启动试试：

./chromedriver

web drive:是个代理（服务），应该是接收python的操作，再调用浏览器API。

数据驱动

就是把所有配置信息放在一个文件中，日常维护这个文档即可，程序写一次就够了。

配置文件中包含，如：

域名

API\-\>URL
