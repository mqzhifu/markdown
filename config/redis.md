# redis

```
bind 127.0.0.1
port 6379
daemonize yes
timeout 60
pidfile /var/run/redis_6379.pid
loglevel notice
logfile "/data/logs/redis/redis.log"

maxmemory 256MB

#持久化-快照模式 ，系统触发机制 ,bgsave 开新纯种后台异步
save 900 1：表示900 秒内如果至少有 1 个 key 的值变化，则保存
save 300 10：表示300 秒内如果至少有 10 个 key 的值变化，则保存
save 60 10000：表示60 秒内如果至少有 10000 个 key 的值变化，则保存

rdbcompression no  #不要压缩RDB，占CPU
dbfilename "6379.rdb"
dir /data/logs/redis
stop-writes-on-bgsave-error yes #bgsave持久化rdb时，如果失败了，redis就自动停掉

#异步删除
lazyfree-lazy-eviction yes
lazyfree-lazy-expire yes
lazyfree-lazy-server-del yes
replica-lazy-flush yes

slowlog-log-slower-than 5000
slowlog-max-len 1000 #慢日志，存储多少条

```
