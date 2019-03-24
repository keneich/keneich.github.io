# TIG Stack(Telegraf, InfluxDB and Grafana) Œ³Ä¡ (on Ubuntu 18.04)

- InfluxDB ŽÂ Go·Î Ÿ²¿©Áø œÃ°è¿­ database ¿ÀÇÂŒÒœº ÀÌŽÙ. 
- Telegraf ŽÂ collecting, processing, aggregating, and writing metricsÀ» À§ÇÑ ¿¡ÀÌÀüÆ®ÀÌŽÙ.
- GrafanaŽÂ µ¥ÀÌÅÍžŠ œÃ°¢È­¿Í žðŽÏÅÍžµÀ» ŒöÇàÇÏŽÂ ¿ÀÇÂŒÒœº ÀÌŽÙ.

ÀÌ¹ø Æ÷œºÆÃÀº Ubuntu 18.04¿¡ TIG StackÀ» Œ³Ä¡ ¹× Œ³Á€ ÇÏŽÂ °ÍÀ» ºž¿©ÁØŽÙ.

## »çÀü ÁØºñ
- Ubuntu 18.04
- Root ±ÇÇÑ


## ÇØŸßÇÒ °Íµé
- InfluxDB Œ³Ä¡
- InfluxDB Database ¿Í User »ýŒº
- Telegraf ¿¡ÀÌÀüÆ® Œ³Ä¡
- Telegraf Œ³Á€
- Grafana Œ³Ä¡
- Grafana Data Source ±žÃà
- Grafana Dashboard ±žÃà

## Step 1 - InfluxDB Œ³Ä¡
Ã¹¹øÂ° °úÁ€Àž·Î, ¿ìºÐÅõ¿¡ InfluxDBžŠ Œ³Ä¡ ÇÑŽÙ.

- influxdata Key žŠ Ãß°¡ÇÑŽÙ.

```shell

sudo curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -

```
- influxdata repositoryžŠ Ãß°¡ÇÑŽÙ.

```shell
source /etc/lsb-release
echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
```

- repositoryžŠ Ÿ÷µ¥ÀÌÆ® ÇÏ°í ŸÆ·¡  Ä¿žÇµå·Î influxdb ÆÐÅ°ÁöžŠ Œ³Ä¡ÇÑŽÙ.
```shell
sudo apt update
sudo apt install influxdb -y
```

- Œ³Ä¡°¡ ¿Ï·áµÇžé, influxdb Œ­ºñœºžŠ œÃÀÛÇÏ°í œÃœºÅÛ ºÎÆÃ œÃ œÇÇàµÇµµ·Ï Œ³Á€ÇÑŽÙ.
```shell
sudo systemctl start influxdb
sudo systemctl enable influxdb

keneich@keneich:~$ sudo netstat -plntu
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      807/systemd-resolve 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1419/sshd           
tcp        0      0 127.0.0.1:8088          0.0.0.0:*               LISTEN      3204/influxd        
tcp6       0      0 :::8086                 :::*                    LISTEN      3204/influxd        
tcp6       0      0 :::22                   :::*                    LISTEN      1419/sshd           
udp        0      0 127.0.0.53:53           0.0.0.0:*                           807/systemd-resolve 
udp6       0      0 fe80::a00:27ff:fe6a:546 :::*                                790/systemd-network 
```
- 8088°ú 8086 Æ÷Æ®°¡ 'LISTEN' »óÅÂÀÌŽÙ.

## Step 2 - InfluxDB Database ¿Í User »ýŒº
Telegraf ¿¡ÀÌÀüÆ®·Î ºÎÅÍ µ¥ÀÌÅÍžŠ ÀúÀåÇÏ±âÀ§ÇØŒ­, InfluxDB Database ¿Í User ±žŒºÀÌ ÇÊ¿äÇÏŽÙ. 

- influxdbžŠ cli·Î ±žµ¿ÇÑŽÙ.

InfluxDB Database ¿Í User »ýŒº

8086 Æ÷Æ®·Î Influxdb server¿¡ Á¢ŒÓÇÑŽÙ.
DBžŠ »ýŒºÇÏ°í telegraf °ú ÆÐœº¿öµåžŠ Œ³Á€ÇÑŽÙ.
database ¿Í useržŠ È®ÀÎÇÑŽÙ.
ŸÆ·¡ influxdb Äõž®žŠ »ç¿ëÇÏ¿© »ç¿ëÇÏ¿ŽŽÙ.


```shell
keneich@keneich:~$ influx
Connected to http://localhost:8086 version 1.7.4
InfluxDB shell version: 1.7.4
Enter an InfluxQL query
> 
> create database telegraf
> create user telegraf with password 'password'
> 
> show databases
name: databases
name
----
_internal
telegraf
> show users
user     admin
----     -----
telegraf false

```

## Step 3 - Telegraf ¿¡ÀÌÀüÆ®žŠ Œ³Ä¡ÇÑŽÙ.
