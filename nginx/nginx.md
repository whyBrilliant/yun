# nginx

## 安装

服务端安装： yum -y install nginx



## 使用

```
  nginx 使用:
  start nginx  # 启动
  nginx -s reload  # 修改配置后重新加载生效
  nginx -s reopen  # 重新打开日志文件
  nginx -t  # 配置文件检测是否正确
```



常见错误

1. nginx: [error] invalid PID number "" in "/run/nginx.pid"
     解决办法：nginx -c /etc/nginx/nginx.conf

