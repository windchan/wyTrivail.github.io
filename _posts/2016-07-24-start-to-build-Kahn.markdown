---
layout: post
title: "Start to build Kahn"
categories: jekyll update
---
Hey man, let's design a secure system to defense invalid requests, for improving the stability of our web system.

## The Components i will use

* Apache Storm, i will use storm to calculate the realtime log info from kafka
* Redis, the result calculated will be stored in redis temporarily
* Crontab, the system will send a alarm email and ban the suspicious ips based on the data of redis every minute
* Nodejs, there will be a dashboard to display the real-time data

## The Data in Redis do i need for Banning IP

* the num of requests per ip in one minute
* the num of requests per ip-and-url in one minute
* the num of requests per ip-and-statusCode in one minute
* the num of requests per class B of ip in one minute
* the num of requests per class B of ip-and-url in one minute
* the num of requests per class B of ip-and-statusCode in one minute
* the num of requests per class C of ip in one minute
* the num of requests per class C of ip-and-url in one minute
* the num of requests per class C of ip-and-statusCode in one minute
* the num of requests per status in one minutes

## the whole flow

* Firstly, the log from kafka will be split into ip, host, url, these hosts in whitelist and static request will be ignore
* Secondly, ip, host, url and statusCode will be counting in redis
* A task for sending email will be execute every minutes, which fetchs the top counting data in redis

## The Spout and Bolts in Apacha Storm

There are one spout and five bolts

* The Spout will use kafka to send log info
* The First Bolt, named LogProcessor, will split the log info, filter the useless requests, then send the ip info to the counting bolts, which use fieldsGrouping
* The First Counting Bolt, named IpCounter, will counting the num of ip, ip-and-url, ip-and-statusCode by using edis 
* The Second Counting Bolt, named BIpCounter, will counting the num of requests , per class B of ip, class B of ip-and-url, class B of ip-and-statusCode by using redis
* The Second Counting Bolt, named CIpCounter, will counting the num of requests, per class C of ip, class C of ip-and-url, class C of ip-and-statusCode by using redis
* The Third Counting Bolt, named CIpCounter, will counting the num of requests per statusCode by using redis

## The Data Structure in Redis

SortedSet will be used, A group of nums per ip will be in a sorted set.
Below are the name definition of every sortedset, time format is '201607241239', every key must be expired in two minutes

* the num of requests per ip in one minute, the sorted set's name: `sorted-set-for-ip-counting-%date%`, the key in sorted set: `%ip%`
* the num of requests per ip-and-url in one minute, the sorted set's name: `sorted-set-for-url-counting-%ip%-%date%`, the key in sorted set: `%url%`
* the num of requests per ip-and-statusCode in one minute, the sorted set's name: `sorted-set-for-status-counting-%ip%-%date%`, the key in sorted set: `%statusCode%`
* the num of requests per class B of ip in one minute, the sorted set'name: `sorted-set-for-bip-counting-%date%`, the key in sorted set: `%ip%`
* the num of requests per class B of ip-and-url in one minute, the key-value pair's key name: `bip-url-%the class B of ip%-%date%-%url%`
* the num of requests per class B of ip-and-statusCode in one minute, the key-value pair's key name: `bip-status-%the class B of ip%-%date%-%status%`
* the num of requests per class C of ip in one minute, the sorted set'name: `sorted-set-for-cip-counting-%date%`, the key in sorted set: `%ip%`
* the num of requests per class C of ip-and-url in one minute, the key-value pair's key name: `cip-url-%the class C of ip%-%date%-%url%`
* the num of requests per class C of ip-and-statusCode in one minute, the key-value pair's key name: `cip-status-%the class C of ip%-%date%-%status%`
* the num of requests per status in one minutes, the sorted set's name: `sorted-set-for-status-counting-%date%`, the key in set: `%status%`

## The Cronjob

A task will be execute every minute, which config in linux system by using the cmd: crontab.
In order to get the complate data in every minute, The task will be execute after ten seconds for every minute, for example, the task will be execute in 10:01:10, 10:02:10
The task will do the things below:

* Firstly, the task fetchs the top ten data from redis sorted set
* Then, the task will fetch the ip-url, ip-status's data of the top ten ip in redis, by using redis cmd/function: KEYS
* And the next, rendering a html text and send email
* Finally, ban the ips!(it is no need to do so in the early stage because we need analysis the data)


## Tips

* use fieldsGrouping
* use sortedset in Redis
* filter the useless requests
