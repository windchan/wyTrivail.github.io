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
* the num of requests per status in one hours

## what's the whole flow
* Firstly, the log from kafka will be split into ip, host, url, these hosts in whitelist and static request will be ignore
* Secondly, ip, host, url and statusCode will be counting in redis
* A task for sending email will be execute every minutes, which fetchs the top counting data in redis


