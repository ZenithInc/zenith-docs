# 配置日志切割


当我们的 Nginx 运行时间比较久或者用户量比较大，就会导致日志迅速增长，日志量非常大。这时候我们去排查日志就不太方便，所以这篇文档介绍了如何对 Nginx 的日志进行切割。

## 按天为单位切割日志 {id="daily"}

一般我们按天为单位切割日志，脚本如下:
```Shell
#!/bin/bash
LOG_PATH="/var/log/nginx/"
RECORD_TIME=$(date -d "yesterday" +百分号Y-百分号m-%d)
PID=/var/run/nginx/nginx.pid
mv ${LOG_PATH}/access.log ${LOG_PATH}/access.${RECORD_TIME}.log
mv ${LOG_PATH}/error.log ${LOG_PATH}/error.${RECORD_TIME}.log

#向Nginx主进程发送信号，用于重新打开日志文件
kill -USR1 `cat $PID`
```

## 设置定时任务每天执行 {id="crontab"}

然后我们设置定时任务，每天执行。首先配置定时任务:
```shell
 $ crontab -e # 打开定时任务配置文件
```
在配置文件中加入如下配置：
```shell
*/1 * * * * /usr/local/nginx/sbin/cut_nginx_log.sh
```
然后重启定时任务:
```shell
$ systemctl restart crond
```
查看定时任务列表:
```shell
$ crontab -l
```
定时任务配置格式:

|      | 分    | 时    | 日    | 月    | 星期几 | 年(可选)   |
|------|------|------|------|------|-----|---------|
| 取值范围 | 0-59 | 0-23 | 1-31 | 1-12 | 1-7 | 比如 2023 |

常用的表达式:
```shell
# 每分钟执行
*/1 * * * *
# 每天晚上 23:59 分执行
59 23 * * *
# 每天凌晨 1 点执行
0 1 * * *
```