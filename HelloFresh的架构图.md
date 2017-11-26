![](http://cdn.infoqstatic.com/statics_s1_20170228-0434_4/resource/news/2017/03/Startups-API-practice/zh/resources/1.png)

* ### 主API
  * #### 10多个主API部署在装配了高端CPU的主机上
  * #### 主从MySQL（3个副本）
* ### 认证服务
  * #### 4个应用服务器
  * #### 主从PostgreSQL（2个副本）
  * #### RabbitMQ用于异步地处理用户的更新操作
* ### API网关
  * #### 4个应用服务器
  * #### 主从MongoDB（4个副本）
* ### 其它
  * #### 所有机器上都运行着Ansible命令，部署只需要几秒钟
  * #### 使用Amazon CloudFront作为CDN/WAF
  * #### 使用Consul和HAProxy作为服务发现和客户端负载均衡工具
  * #### 使用Statsd和Grafana收集系统度量指标并触发告警
  * #### 使用ELK技术栈从不同的服务收集日志
* #### 使用Concourse CI作为持续集成工具