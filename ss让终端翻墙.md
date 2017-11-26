
### brew install polipo 这是一个可以代理终端请求到ss的武器(目前已经不维护了), 类似的还有
  * Squid
  * Privoxy
  * Varnish Cache
  * Tinyproxy
### polipo 配置文件里关键的一句式
```
socksParentProxy = "localhost:1080" // 一般ss的代理端口就是1080
socksProxyType = socks5 // 使用协议
```
### 若是上面的配置文件放到了 /etc/polipo/p.conf中，则启动时是这样的 polipo -c /etc/polipo/p.conf

### 最后一步就是设置环境变量
```
http_proxy="localhost:8123"
https_proxy="locahost:8123"
```

### 检验 curl ip.gs即可查看当前ip即地址
```
// 例如
Current IP / 当前 IP: 45.32.37.110
ISP / 运营商:  choopa.com
City / 城市: Tokyo Tokyo
Country / 国家: Japan
```