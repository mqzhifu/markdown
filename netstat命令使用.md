# netstat命令使用

netstat \-n | awk '/^tcp/ {\+\+S\[$NF\]} END {for\(a in S\) print a, S\[a\]}'

CLOSED //无连接是活动的或正在进行

LISTEN //服务器在等待进入呼叫

SYN\_RECV //一个连接请求已经到达，等待确认

SYN\_SENT //应用已经开始，打开一个连接

ESTABLISHED //正常数据传输状态/当前并发连接数

FIN\_WAIT1 //应用说它已经完成

FIN\_WAIT2 //另一边已同意释放

ITMED\_WAIT //等待所有分组死掉

CLOSING //两边同时尝试关闭

TIME\_WAIT //另一边已初始化一个释放

LAST\_ACK //等待所有分组死掉
