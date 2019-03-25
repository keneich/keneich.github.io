---
layout: post
categories: articles
#author: mint
title: "TIG Stack을 설치"
excerpt: "Telegraf, InfluxDB, and Grafana on Ubuntu 18.04 LTS "
date: 2019-03-20 01:05:27
modified: 2019-03-20 01:05:27
tags: [telegraf,grafana,influxdb]
image:
  feature: 2019-03-20-TIG-Stack-onUbuntu/TIG.png
share: true
sitemap: true
---


# TIG Stack(Telegraf, InfluxDB and Grafana) 설치 (on Ubuntu 18.04)

- InfluxDB 는 Go로 쓰여진 시계열 database 오픈소스 이다.
- Telegraf 는 collecting, processing, aggregating, and writing metrics을 위한 에이전트이다.
- Grafana는 데이터를 시각화와 모니터링을 수행하는 오픈소스 이다.

이번 포스팅은 Ubuntu 18.04에 TIG Stack을 설치 및 설정 하는 것을 보여준다.

## 사전 준비
- Ubuntu 18.04
- Root 권한


## 해야할 것들
- InfluxDB 설치
- InfluxDB Database 와 User 생성
- Telegraf 에이전트 설치
- Telegraf 설정
- Grafana 설치
- Grafana Data Source 구축
- Grafana Dashboard 구축

## Step 1 - InfluxDB 설치
첫번째 과정으로, 우분투에 InfluxDB를 설치 한다.

- influxdata Key 를 추가한다.

```shell

sudo curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -

```
- influxdata repository를 추가한다.

```shell
source /etc/lsb-release
echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
```

- repository를 업데이트 하고 아래  커맨드로 influxdb 패키지를 설치한다.
```shell
sudo apt update
sudo apt install influxdb -y
```

- 설치가 완료되면, influxdb 서비스를 시작하고 시스템 부팅 시 실행되도록 설정한다.

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
- 8088과 8086 포트가 'LISTEN' 상태이다.

## Step 2 - InfluxDB Database 와 User 생성
Telegraf 에이전트로 부터 데이터를 저장하기위해서, InfluxDB Database 와 User 구성이 필요하다.

- influxdb를 cli로 구동한다.

InfluxDB Database 와 User 생성

8086 포트로 Influxdb server에 접속한다.
DB를 생성하고 telegraf 과 패스워드를 설정한다.
database 와 user를 확인한다.
아래 influxdb 쿼리를 사용하여 사용하였다.


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

## Step 3 - Telegraf 에이전트를 설치한다.
Telegraf는 influxdb가 생성된 같은 구성의 'influxdata'에 의해 생성되었다.
그리고 influxdata key와 repository가 추가되었다.

- telegraf 패키지를 설치한다.
```shell
sudo apt install telegraf -y
```
- 설치가 완료되면, telegraf 서비스를 시작하고 시스템 부팅 시 실행되도록 설정한다.


```shell
keneich@keneich:~$ sudo systemctl start telegraf
keneich@keneich:~$ sudo systemctl enable telegraf
keneich@keneich:~$ sudo systemctl status telegraf
● telegraf.service - The plugin-driven server agent for reporting metrics into InfluxDB
   Loaded: loaded (/lib/systemd/system/telegraf.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2019-03-20 05:02:35 UTC; 2min 42s ago
     Docs: https://github.com/influxdata/telegraf
 Main PID: 3414 (telegraf)
    Tasks: 8 (limit: 4662)
   CGroup: /system.slice/telegraf.service
           └─3414 /usr/bin/telegraf -config /etc/telegraf/telegraf.conf -config-directory /etc/telegraf/telegraf.d

Mar 20 05:02:35 keneich systemd[1]: Started The plugin-driven server agent for reporting metrics into InfluxDB.
Mar 20 05:02:35 keneich telegraf[3414]: 2019-03-20T05:02:35Z I! Starting Telegraf 1.10.1
Mar 20 05:02:35 keneich telegraf[3414]: 2019-03-20T05:02:35Z I! Loaded inputs: mem processes swap system cpu disk disk
Mar 20 05:02:35 keneich telegraf[3414]: 2019-03-20T05:02:35Z I! Loaded aggregators:
Mar 20 05:02:35 keneich telegraf[3414]: 2019-03-20T05:02:35Z I! Loaded processors:
Mar 20 05:02:35 keneich telegraf[3414]: 2019-03-20T05:02:35Z I! Loaded outputs: influxdb
Mar 20 05:02:35 keneich telegraf[3414]: 2019-03-20T05:02:35Z I! Tags enabled: host=keneich
Mar 20 05:02:35 keneich telegraf[3414]: 2019-03-20T05:02:35Z I! [agent] Config: Interval:10s, Quiet:false, Hostname:"k
keneich@keneich:~$

```

## Step 4 - Configure Telegraf
Telegraf 는 4가지 plugin 타입을 가진다.

1. input Plugins : 수집용
2. Processor Plugins : 변형, 가공, 필터링
3. Aggregator Plugin : 결합(Aggregate)
4. Output Plugins : Write용

이번 과정에서는, 기본 input metric으로 사용하기 위해 telegraf를 설정한다.
그리고 output Plugin으로 influxdb를 사용하기 위함이다.

- telegraf.conf 파일을 생성한다.

```shell
cd /etc/telegraf/
mv telegraf.conf telegraf.conf.default
vim telegraf.conf

# Global Agent Configuration
[agent]
  hostname = "hakase-tig"
  flush_interval = "15s"
  interval = "15s"


# Input Plugins
[[inputs.cpu]]
    percpu = true
    totalcpu = true
    collect_cpu_time = false
    report_active = false
[[inputs.disk]]
    ignore_fs = ["tmpfs", "devtmpfs", "devfs"]
[[inputs.io]]
[[inputs.mem]]
[[inputs.net]]
[[inputs.system]]
[[inputs.swap]]
[[inputs.netstat]]
[[inputs.processes]]
[[inputs.kernel]]

# Output Plugin InfluxDB
[[outputs.influxdb]]
  database = "telegraf"
  urls = [ "http://127.0.0.1:8086" ]
  username = "telegraf"
  password = "hakase-ndlr"
```

- 재시작  

```shell
sudo systemctl restart telegraf
```

- 다음 명령어를 사용하여 telegraf 설정을 테스트 한다.  

sudo telegraf -test -config /etc/telegraf/telegraf.conf --input-filter cpu
sudo telegraf -test -config /etc/telegraf/telegraf.conf --input-filter net
sudo telegraf -test -config /etc/telegraf/telegraf.conf --input-filter mem


```shell
keneich@keneich:/etc/telegraf$ sudo telegraf -test -config /etc/telegraf/telegraf.conf --input-filter cpu
2019-03-20T05:34:02Z I! Starting Telegraf 1.10.1
> cpu,cpu=cpu0,host=keneich usage_guest=0,usage_guest_nice=0,usage_idle=100,usage_iowait=0,usage_irq=0,usage_nice=0,usage_softirq=0,usage_steal=0,usage_system=0,usage_user=0 1553060043000000000
> cpu,cpu=cpu-total,host=keneich usage_guest=0,usage_guest_nice=0,usage_idle=100,usage_iowait=0,usage_irq=0,usage_nice=0,usage_softirq=0,usage_steal=0,usage_system=0,usage_user=0 1553060043000000000
keneich@keneich:/etc/telegraf$ sudo telegraf -test -config /etc/telegraf/telegraf.conf --input-filter net
2019-03-20T05:34:11Z I! Starting Telegraf 1.10.1
> net,host=keneich,interface=enp0s3 bytes_recv=70130014i,bytes_sent=1311552i,drop_in=0i,drop_out=0i,err_in=0i,err_out=0i,packets_recv=56990i,packets_sent=13585i 1553060051000000000
> net,host=keneich,interface=all icmp_inaddrmaskreps=0i,icmp_inaddrmasks=0i,icmp_incsumerrors=0i,icmp_indestunreachs=80i,icmp_inechoreps=2i,icmp_inechos=0i,icmp_inerrors=0i,icmp_inmsgs=82i,icmp_inparmprobs=0i,icmp_inredirects=0i,icmp_insrcquenchs=0i,icmp_intimeexcds=0i,icmp_intimestampreps=0i,icmp_intimestamps=0i,icmp_outaddrmaskreps=0i,icmp_outaddrmasks=0i,icmp_outdestunreachs=80i,icmp_outechoreps=0i,icmp_outechos=2i,icmp_outerrors=0i,icmp_outmsgs=82i,icmp_outparmprobs=0i,icmp_outredirects=0i,icmp_outsrcquenchs=0i,icmp_outtimeexcds=0i,icmp_outtimestampreps=0i,icmp_outtimestamps=0i,icmpmsg_intype0=2i,icmpmsg_intype3=80i,icmpmsg_outtype3=80i,icmpmsg_outtype8=2i,ip_defaultttl=64i,ip_forwarding=2i,ip_forwdatagrams=0i,ip_fragcreates=0i,ip_fragfails=0i,ip_fragoks=0i,ip_inaddrerrors=0i,ip_indelivers=57600i,ip_indiscards=0i,ip_inhdrerrors=0i,ip_inreceives=57600i,ip_inunknownprotos=0i,ip_outdiscards=40i,ip_outnoroutes=0i,ip_outrequests=14131i,ip_reasmfails=0i,ip_reasmoks=0i,ip_reasmreqds=0i,ip_reasmtimeout=0i,tcp_activeopens=22i,tcp_attemptfails=0i,tcp_currestab=3i,tcp_estabresets=1i,tcp_incsumerrors=0i,tcp_inerrs=0i,tcp_insegs=61662i,tcp_maxconn=-1i,tcp_outrsts=9i,tcp_outsegs=18270i,tcp_passiveopens=5i,tcp_retranssegs=3i,tcp_rtoalgorithm=1i,tcp_rtomax=120000i,tcp_rtomin=200i,udp_ignoredmulti=24i,udp_incsumerrors=0i,udp_indatagrams=718i,udp_inerrors=0i,udp_noports=80i,udp_outdatagrams=801i,udp_rcvbuferrors=0i,udp_sndbuferrors=0i,udplite_ignoredmulti=0i,udplite_incsumerrors=0i,udplite_indatagrams=0i,udplite_inerrors=0i,udplite_noports=0i,udplite_outdatagrams=0i,udplite_rcvbuferrors=0i,udplite_sndbuferrors=0i 1553060051000000000
keneich@keneich:/etc/telegraf$ sudo telegraf -test -config /etc/telegraf/telegraf.conf --input-filter mem
2019-03-20T05:34:18Z I! Starting Telegraf 1.10.1
> mem,host=keneich active=318726144i,available=3748708352i,available_percent=90.62133019120137,buffered=28745728i,cached=676098048i,commit_limit=5547507712i,committed_as=788303872i,dirty=106496i,free=3274629120i,high_free=0i,high_total=0i,huge_page_size=2097152i,huge_pages_free=0i,huge_pages_total=0i,inactive=423084032i,low_free=0i,low_total=0i,mapped=107220992i,page_tables=4558848i,shared=1028096i,slab=70643712i,swap_cached=0i,swap_free=3479171072i,swap_total=3479171072i,total=4136673280i,used=157200384i,used_percent=3.800164367827473,vmalloc_chunk=0i,vmalloc_total=35184372087808i,vmalloc_used=0i,wired=0i,write_back=0i,write_back_tmp=0i 1553060059000000000
keneich@keneich:/etc/telegraf$
```

InfluxDB 와 Telegraf configuration 이 완료 됬다.


## Step 5 - Grafana 설치

이번 스텝에서는, 데이터 시각화를 위해 Grafana 데쉬보드를 설치할 것이다.

- grafana 키와 레파지토리를 추가한다.  

```shell
sudo curl https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo echo 'deb https://packages.grafana.com/oss/deb stable main' > /etc/apt/sources.list.d/grafana.list
```

- grafana 패키지 설치  

```shell
sudo apt update
sudo apt install grafana -y
```

- grafana 서비스를 시작하고 시스템 부팅 시 실행되도록 설정한다.

```shell
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

## Step 6 - Grafana 설치
- 브라우저를 열고 IP:포트로 접속한다.

http://192.168.1.99:3000  
Login : admin / admin

![TIG](/images/2019-03-20-TIG-Stack-onUbuntu/tig01.png)

- Influxdb data source 를 추가하기 위해 'Add data source' 클릭한다.

- 다음 설정 값을 등록한다.

Name: influxdb
URL: http://localhost:8086/

![TIG](/images/2019-03-20-TIG-Stack-onUbuntu/tig02.png)

-  스트롤을 내려 influxdb database 설정 한다.

InfluxDB Detail 항목
Database: telegraf
User: telegraf
Password: 'Password'

'Save and Test' button을 클릭하면
'Data source is working' 이라는 결과 값이 보여야 한다.

![TIG](/images/2019-03-20-TIG-Stack-onUbuntu/tig03.png)

InfluxDB data 가 Grafana server에 추가되었다.

## Step 7 - Grafana 데쉬보드 구성
Telegraf input plugin을 설정하여 grafana Dashboard를 import 한다.

메뉴의 '+' -> 'Import' 를 클릭한다.

Now open the sample Grafana dashboard from URL 'https://grafana.com/dashboards/5955' and click the 'Copy the ID to Clipboard' button.

Paste the dashboard id.

And you will be redirected automatically to the dashboard setup.

On the options section, click the InfluxDB and choose your influxdb server, then click 'Import' button.

아래 이미지와 같은 Dashboard를 볼수 있다.

![TIG](/images/2019-03-20-TIG-Stack-onUbuntu/tig04.png)

이제 TIG Stack (Telegraf, InfluxDB, and Grafana) 이 Ubuntu 18.04 에서 성공적으로 설치되었다.

<!--
![TIG](/images/2019-03-20-TIG-Stack-onUbuntu/tig01.jpg){: width="300" height="400"}
--->
