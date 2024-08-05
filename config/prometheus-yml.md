# prometheus.yml

```
# my global config                       
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.                                              
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.                           
  # scrape_timeout is set to the global default (10s). 
# Alertmanager configuration                                                                               
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['127.0.0.1:9093']                                   
      #- "127.0.0.1:9093"                                                 
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.                                                                                 
rule_files:
  - "first_rules.yml"
  # - "second_rules.yml"
  # A scrape configuration containing exactly one endpoint to scrape:
  # Here it's Prometheus itself.
  
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.                                                                                          
  - job_name: 'prometheus'
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.                                          
    
    static_configs:
    - targets: ['localhost:9090']
    
  - job_name: 'centos_node'
    static_configs:
      - targets: ['39.106.65.76:9100']
      
  - job_name: 'apache_node'
    static_configs:
      - targets: ['39.106.65.76:9117']
      
  - job_name: 'nginx-vts'
    static_configs:
      - targets: ['39.106.65.76:9913']
      
  - job_name: 'php-fpm-exporter'
    static_configs:
     - targets: ['39.106.65.76:9190']
     
  - job_name: 'mysqld-exporter'
    static_configs:
     - targets: ['39.106.65.76:9104']
     
  - job_name: 'redis-exporter'
    static_configs:
     - targets: ['39.106.65.76:27000']
     
  - job_name: 'rabbitmq_prometheus'
    static_configs:
     - targets: ['39.106.65.76:27000']
     
```
