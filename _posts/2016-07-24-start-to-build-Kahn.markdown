---
layout: post
title: "Start to build Kahn"
categories: jekyll update
---
Today, i will design a secure system to defense invalid requests, in order to improve the stability of our web system.

## what components i will use
* Apache Storm, i will use storm to calculate the realtime log info from kafka
* Redis, the result calculated will be stored in redis temporarily
* Crontab, the system will send a alarm email and ban the suspicious ips based on the data of redis every minute
* Nodejs, there will be a dashboard to display the real-time data

## what data in redis do i need for banning ip
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

SortedSet and simple key-value pair will be used, A group of nums per ip will be in a sorted set, and the other num will just store in key-value pairs.
Below are the name definition of every sortedset and key-value pairs, time format is '201607241239', every key must be expired in two minutes

* the num of requests per ip in one minute, the sorted set'name: 'sorted-set-for-ip-counting-%date%', the key in sorted set: '%ip%'
* the num of requests per ip-and-url in one minute, the key-value pair's key name: 'ip-url-%ip%-%url%-%date%'
* the num of requests per ip-and-statusCode in one minute, the key-value pair's key name: 'ip-status-%ip%-%status%-%date%'
* the num of requests per class B of ip in one minute, the sorted set'name: 'sorted-set-for-bip-counting-%date%', the key in sorted set: '%ip%'
* the num of requests per class B of ip-and-url in one minute, the key-value pair's key name: 'bip-url-%the class B of ip%-%url%-%date%'
* the num of requests per class B of ip-and-statusCode in one minute, the key-value pair's key name: 'bip-status-%the class B of ip%-%status%-%date%'
* the num of requests per class C of ip in one minute, the sorted set'name: 'sorted-set-for-cip-counting-%date%', the key in sorted set: '%ip%'
* the num of requests per class C of ip-and-url in one minute, the key-value pair's key name: 'cip-url-%the class C of ip%-%url%-%date%'
* the num of requests per class C of ip-and-statusCode in one minute, the key-value pair's key name: 'cip-status-%the class C of ip%-%status%-%date%'
* the num of requests per status in one hours, the key-value pair's key name: 'count-status-%status%-%date%'
