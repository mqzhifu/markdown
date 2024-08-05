# linkerd

```
admin:
  ip: 0.0.0.0
  port: 9990

usage:
  enabled: false

telemetry:
- kind: io.l5d.prometheus
- kind: io.l5d.zipkin
  ##IP地址根据环境更改
  host: 172.19.162.237
  port: 9410
  sampleRate: 1.00

namers:
  - kind: io.l5d.consul
    ##IP地址根据环境更改
    host: 172.19.162.237
    port: 8500

routers:
- protocol: http
  #admin UI label ，上线的时候把test改成online
  label: service_http_consul_test
  bindingTimeoutMs: 5000
  interpreter:
    kind: default
  compressionLevel: -1
  identifier:
    kind: io.l5d.header.token
  service:
    kind: io.l5d.global
    totalTimeoutMs: 10000
    responseClassifier:
      kind: io.l5d.http.retryableAll5XX
    retries:
      budget:
        minRetriesPerSec: 1
        percentCanRetry: 1.0
      backoff:
        kind: jittered
        minMs: 1000
        maxMs: 5000
  client:
    hostConnectionPool:
      minSize: 0
      maxSize: 2000
      idleTimeMs: 60000
      maxWaiters: 500
    failFast: false
    requeueBudget:
      percentCanRetry: 1.0
    failureAccrual:
      kind: io.l5d.successRateWindowed
      successRate: 0.95
      window: 300
      backoff:
        kind: jittered
        minMs: 5000
        maxMs: 60000
    loadBalancer:
      kind: ewma
      maxEffort: 5
      decayTimeMs: 10000
  httpAccessLog: logs/access.log
  httpAccessLogRollPolicy: daily
  httpAccessLogAppend: true
  httpAccessLogRotateCount: 7
  bindingCache:
    paths: 1000
    trees: 1000
    bounds: 1000
    clients: 1000
    idleTtlSecs: 600
  dtab: |
    ##dc名字根据环境更改
    /svc => /#/io.l5d.consul/dc-test01;
  servers:
  - port: 4140
    ip: 0.0.0.0
    addForwardedHeader:
      by: {kind: "ip:port"}
      for: {kind: ip}
```
