This document is created in order to present a how-to collect OS logs via using nxlog -> logstash -> rabbitmq setup.

## Pre-requirements
*  Java8+ (`sudo apt-get install default-jdk`)
*  [Nxlog](https://nxlog.co/products/nxlog-community-edition/download)
*  [LogStash 6.1+](https://www.elastic.co/downloads/logstash)
*  RabbitMQ 3.7+ ([linux](https://tecadmin.net/install-rabbitmq-server-on-ubuntu/#) [windows](https://cmatskas.com/getting-started-with-rabbitmq-on-windows/))


## Setup
This Setup instructions are tested on Windows machine.

1.  Install Java to your host machine
2.  Install nxlog to your host machine
3.  Install RabbitMQ to your host machine
4.  Download Logstash and extract folder on your host machine

# RabbitMQ
1.  Create admin account in rabbitMQ in order to be able to create queues. You may optionally create accounts for different connections in order to not get authentication failures
2.  Create queue used in named as "rawLogs"
![rabbitQueues](/pics/rabbitQueues.PNG)

3.  Create an exchange in rabbitMQ in order to receive logs from logstash
<br>
3.a.  Call this exchange as "logstash".

![exchangeCreate](/pics/exchangeCreate.PNG)

3.b.  Then bind this exchange into "rawLogs" queue. Now You can send logs from logstash and rabbit will direct logs to "rawLogs".

![exchangeBindPNG](/pics/exchangeBindPNG.PNG)

# Nxlog
[Configuration Sample](https://nxlog.co/docs/elasticsearch-kibana/using-nxlog-with-elasticsearch-and-kibana.html)
1.  Configure nxlog.conf file(under C:\Program Files\nxlog\conf\ )  Here is a sample

![nxlog.conf](/pics/nxlogConf.PNG)

In the sample, eventlogs are written under the file (c:\nxlogOutput\nxlog.txt) in Route 1. Route 2 reads input from this file and currently occuring windows logs. Route 2 forwards these two inputs to logstash under logstash defined in "eventlog_outLogStash".

2.  One can start-stop nxlog service under windows services

# LogStash
1.  Go to the logstash/bin directory (~\logstash-6.1.3\bin)
2.  Create a config file named logstash-simple.conf
3.  Fill the logstash config file. Here is an example.

![logstash.conf](/pics/logstashConf.PNG)

In the sample, previously configures nxlog configuration is defined under input tag. Under filter, each log is added with the fields of "SourceIp" and "id". Finally logs are forwarded to rabbitMQ configured under output tag.

4.  Open command line under current directory (~\logstash-6.1.3\bin)
5.  Execute the command `logstash.bat -f logstash-simple.conf`

Note: if you get `RabbitMQ connection error, will retry. {:error_message=>"Connection to <RABBITMQ_ADDRESS>:5672 refused", :exception=>"MarchHare::ConnectionRefused"}`, try to make an inbound firewall rule, in the machine where rabbitmq installed, for 5672 port
