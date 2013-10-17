---
layout: post
title: "Cluster Name Service"
description: ""
category: 
tags: []
---
{% include JB/setup %}


一个复杂的互联网服务的后台，通常包括很多services程序；互相调用，构成一个有向无环图（directed acyclic graph）。通常被调用者被称为**上游**服务，调用者被称为**下游**服务。

通常，为了容错或者提高吞吐量（throughput），每个service程序会用来启动多个进程，每个进程称为一个instance。

当一个下游服务intance要调用一个上游服务的时候，本来应该只需要知道上游服务的名字（service name），因为连到任何一个instance都可以。可是，为了建立IP连接，需要知道上游服务的某个具体instance的IP地址和端口（简称instance address）。

一个直观的解决之道是利用一个全局名字服务（name service）维护一套从service name到其所有instance addresses的映射。这个名字服务必须很健壮，因为如果挂了，那么机群上所有services就都不能找到他们的上游服务了。幸运的是，我们可以利用Paxos协议来确保这个健壮性。

一个很典型的实现了Paxos协议的系统是Google Stubby。在开源软件中，用Java写的Zookeeper和用Go写的Doozer也都很好的实现了Paxos。

但是Stubby、Zookeeper和Doozer都不能算是name service——虽然它们可以稳定地维护任意映射，但是它们都不支持service name和instance的注册，也不能确定一个注册过的instance是否还活着。

Google有一个name service叫做GNS（Google Name Service），利用Stubby来维护serivce name到instances的映射，同时支持心跳机制（heartbeat）：一个intance启动的时候把自己的address注册到GNS，并且每隔一段时间要声明一下“我还活着”，否则自己的address会被GNS清除，这样下游服务就不会拿到一个失效了的instance address了。

开源圈子里一直缺少这样一个name service，很多机群管理者不得已，在Zookeeper的基础上开发自己的name service。最近终于有人用Go语言写了一个——[SkyDNS](https://github.com/skynetservices/skydns)。这个服务并不需要以来Zookeeper，而是自己实现了Paxos协议来确保服务的健壮性。

SkyDNS启动之后会打开两个端口：一个HTTP端口和一个DNS端口。任何service instances都可以通过HTTP端口注册自己，另外任何程序都可以向访问标准DNS服务一样，通过提供一个service name查找service instance address。

SkynetDNS是Google之外的公司的福音——让我们可以在实际工作中，建设强大好用的并行计算机群。