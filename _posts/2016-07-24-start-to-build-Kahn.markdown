---
layout: post
title: "Start to build Kahn"
categories: jekyll update
---
Hey man! Let's design a security system to ban anomalous IPs and improve the stability of Kuaizhan!

## The components I will use are listed below:

* Apache Storm: I will use storm to process the real-time log from Kafka
* Redis: Suspicious IPs, URLs, and Requests will be stored in Redis
* Crontab: Every minute, it will ban all the suspicious IPs and send an alarm email after analyzing statistics stored in Redis
* Node.js: There will be a dashboard to display the real-time data


## Data Entries in Redis that I need for banning IPs

* Number of requests per IP in one minute
* Number of requests per IP and URL in one minute
* Number of requests per IP and Status Code in one minute
* Number of requests per class B IP in one minute
* Number of requests per class B IP and URL in one minute
* Number of requests per class B IP and Status Code in one minute
* Number of requests per class C IP in one minute
* Number of requests per class C IP and URL in one minute
* Number of requests per class C IP and Status Code in one minute
* Number of requests per Status Code in one minutes

## How does Kahn work?

* Firstly, a Kafka reader spout will read the log from Kafka queue. Each log entry will be made into a tuple that contains IP, host, URL, Status Code, Class B IP, Class C IP, Log Time, and local machine's System Time. Static requests and requests from hosts that are in the whitelist will be ignored.
* Secondly, the bolts taking tuples from the spout will count the number of occurences of each IP, URL, and Status Code and store them in Redis.
* Eventually, a task will fetch IP, URLs, and Status Codes with the most count and send alarm emails every minute.

## The Spout and Bolts in Apacha Storm

There are one spout and five bolts:

* The Spout will read from Kafka queuee and send information to the first bolt ... 
* The 1st Bolt, named LogProcessor, will extract information susch as IP and host and send the information to the counting bolts that connect to it through FieldsGrouping. Requests allowed in the whitelist will not be counted.
* The 1st Counting Bolt, named IpCounter, will increment the count of IP, IP and URL, IP and Status Code in Redis.
* The 2nd Counting Bolt, named HostCounter, will increment the count of each host.
* The 3rd Counting Bolt, named StatusCounter, will increment the count of each Status Code.
* The 4th Counting Bolt, named UrlCounter, will increment the count of each URL.
* The 5th Counting Bolt, named RequestsCounter, will count the total number of requests.
* The 6th Counting Bolt, named SubsysCounter, will count the number of requests of each sub system.
* The 7th Counting Bolt, named BIpCounter, will increment the count of Class B IP, Class B IP and URL, Class B IP and Status Code in Redis.
* The 8th Counting Bolt, named CIpCounter, will increment the count of Class C IP, Class C IP and URL, Class C IP and Status Code in Redis.

## The Data Structure in Redis

We use Sorted Set to count the number of requests for each field.  
Time Format is 'YYYYMMddmmSS'. eg. '201607241239'  
Every key expires in 10 minutes.  

Below are the structures of every sorted set:  

* Number of requests per IP in one minute. Named of the sorted set is `sorted-set-for-ip-counting-%date%`. The key in the sorted set is `%ip%`
* Number of requests per IP and URL in one minute. Named of the sorted set is `sorted-set-for-url-counting-%ip%-%date%`. The key in the sorted set is `%url%`
* Number of requests per IP and Status Code in one minute. Named of the sorted set is `sorted-set-for-status-counting-%ip%-%date%`. The key in the sorted set is `%statusCode%`
* Number of requests per class B IP in one minute. Named of the sorted set is `sorted-set-for-bip-counting-%date%`. The key in the sorted set is `%ip%`
* Number of requests per Status Code in one minutes. Named of the sorted set is `sorted-set-for-status-counting-%date%`. The key in the sorted set is `%status%`
* Number of requests per class B IP and URL in one minute. Named of the sorted set is `sorted-set-for-url-counting-%the class B of ip%-%date%`. The key in the sorted set is `%url%`
* Number of requests per class B IP and Status Code in one minute. Named of the sorted set is `sorted-set-for-status-counting-%the class B of ip%-%date%`. The key in the sorted set is `%statusCode%`
* Number of requests per class C IP in one minute. Named of the sorted set is `sorted-set-for-cip-counting-%date%`. The key in the sorted set is `%ip%`
* Number of requests per class C IP and URL in one minute. Named of the sorted set is `sorted-set-for-url-counting-%the class C of ip%-%date%`. The key in the sorted set is `%url%`
* Number of requests per class C IP and Status Code in one minute. Named of the sorted set is `sorted-set-for-status-counting-%the class C of ip%-%date%`. The key in the sorted set is `%statusCode%`


## 4 Topologies
Due to limited processing power of the Storm Cluster, Kahn is broken into 4 topologies, each dedicating itself to separate tasks.   
Each topology has its own LogProcessor Bolt.  

* Topology 1 includes IPCounter, StatusCounter, and URLCounter
* Topology 2 includes hostCounter
* Topology 3 includes Class B IPCounter and Class C IPCounter
* Topology 4 includes Sub System Counter


## The Cronjob

A Python script will be executed every minute by using the command crontab.   
In order to have complete data in the previous minute, the script will be executed 10 seconds after every minute ends. For example, the script will be executed at 10:01:10, 10:02:10  

The script does the following tasks:  

* Firstly, it fetches top 10 results from IP's sorted set.
* Secondly, it fetches top 10 results from IP-URL and IP-Status Code sorted set.
* Thirdly, organize the information and send the alarm email.
* Finally, it bans the IPs by talking to nginx.


## Tips

* Use FieldGrouping feature in Storm
* Use Sorted Set in Redis
* Ignore requests that are allowed in the whitelist
