# Prometheus 入门

## Prometheus 简介

Prometheus 是一个系统监控工具，支持。。。。

## Prometheus 安装 

* 通过连接中下载相应版本的 Prometheus [Download](https://prometheus.io/download)

* 解压下载的文件

```properties
tar xvfz prometheus-*.tar.gz
cd prometheus-*
```

## Prometheus 的使用

​	这部分主要用 Prometheus 来监控自己的使用：

 * 创建文件 prometheus.yml，内容如下：

   ```yaml
   global:
     scrape_interval:     15s # By default, scrape targets every 15 seconds.
   
     # Attach these labels to any time series or alerts when communicating with
     # external systems (federation, remote storage, Alertmanager).
     external_labels:
       monitor: 'codelab-monitor'
   
   # A scrape configuration containing exactly one endpoint to scrape:
   # Here it's Prometheus itself.
   scrape_configs:
     # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
     - job_name: 'prometheus'
   
       # Override the global default and scrape targets from this job every 5 seconds.
       scrape_interval: 5s
   
       static_configs:
         - targets: ['localhost:9090']
   ```

* 启动 Prometheus

  ```bash
  # Start Prometheus.
  # By default, Prometheus stores its database in ./data (flag --storage.tsdb.path).
  ./prometheus --config.file=prometheus.yml
  ```

  默认的，Prometheus 开启了 9090 端口，我们可以通过访问 http://localhost:9090 来查看 Prometheus 中的监控信息。

## 总结

