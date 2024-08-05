# alertmanager.yml

```
global:
  resolve_timeout: 5m

  smtp_from: '78878296@qq.com'
  smtp_smarthost: 'smtp.qq.com:465'
  smtp_auth_username: '78878296@qq.com'
  smtp_auth_password: 'glnteewafftmcaje'
  smtp_require_tls: false
  smtp_hello: 'qq.com'

route:
  #这里先做一次大分组,下面的routers会再细分分组
  group_by: ['alertname']
  group_wait: 1s
  group_interval: 10s
  repeat_interval: 10s
  receiver: 'op'
  routes:
  - match:
      severity: 'critical'
      receiver: op
  - match:
      service: "frame_sync"
      receiver: backend
  - match:
      service: "app"
      receiver: front

receivers:
- name: 'op'
  email_configs:
  - to: 'mqzhifu@sina.com'
    send_resolved: true

- name: 'front'
  email_configs:
  - to: 'mqzhifu@sina.com'
    send_resolved: true
  webhook_configs:
  - url: 'http://127.0.0.1:8088/alert.php'

- name: 'backend'
  webhook_configs:
  - url: 'http://127.0.0.1:8088/alert.php'

#inhibit_rules:
#  - source_match:
#      severity: 'critical'
#    target_match:
#      severity: 'warning'
#    equal: ['alertname', 'dev', 'instance']
```
