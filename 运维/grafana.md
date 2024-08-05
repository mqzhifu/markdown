# grafana

## Grafana

https://grafana.com/grafana/download

方式1

> wget https://dl.grafana.com/oss/release/grafana\-7.1.3\-1.x86\_64.rpm
> 
> 
> sudo yum install grafana\-7.1.3\-1.x86\_64.rpm

方式2

> https://dl.grafana.com/oss/release/grafana\-7.3.1.linux\-amd64.tar.gz
> 
> 
> 启动
> 
> 
> ./grafana\-server

39.107.127.244:3000

默认密码

admin

admin

强制修改密码

admin

mqzhifu

## grafana

先添加数据源，这里用promethue

data source

http://127.0.0.1:9000

save & test

没问题后，开始添加DashBoard

还是 data\-course 窗口，右侧有直接 有个tab ，选择 promethue stats 2.0 ,import

回到dashboard 页，就会看到了

服务器主要监控的关注点

cpu process num : cpu总数

mem cache

mem buffer

mem total

内存使用率：

100 \- \(\(node\_memory\_MemFree{instance="xxx"}\+node\_memory\_Cached{instance="xxx"}\+node\_memory\_Buffers{instance="xxx"}\)/node\_memory\_MemTotal\) \* 100

CPU使用率

100 \- \(avg by \(instance\) \(irate\(node\_cpu{instance="xxx", mode="idle"}\[5m\]\)\) \* 100\)

硬盘使用率

100 \- node\_filesystem\_free{instance="xxx",fstype\!~"rootfs|selinuxfs|autofs|rpc\_pipefs|tmpfs|udev|none|devpts|sysfs|debugfs|fuse._"} / node\_filesystem\_size{instance="xxx",fstype\!~"rootfs|selinuxfs|autofs|rpc\_pipefs|tmpfs|udev|none|devpts|sysfs|debugfs|fuse._"} \* 100

// 上行带宽

sum by \(instance\) \(irate\(node\_network\_receive\_bytes{instance="xxx",device\!~"bond.\*?|lo"}\[5m\]\)/128\)

// 下行带宽

sum by \(instance\) \(irate\(node\_network\_transmit\_bytes{instance="xxx",device\!~"bond.\*?|lo"}\[5m\]\)/128\)

插件目录：

/soft/grafana\-7.3.1/data/plugins

./grafana\-cli plugins install

## 添加node expoter grafana

左侧 \+ 号，create import
