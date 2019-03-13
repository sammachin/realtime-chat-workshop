+++
title = "Advanced Techniques"
chapter = false
weight = 40
+++

### Load Balancers
If you are using a load balancer be sure to switch on sticky sessions.

### Running Multiple instances
If you run multiple instance you will need a backplane. In signalR you can use Redis as a backplane. 

1. Create a Redis ElastiCache
2. Change the App to use the Redis Cache

```
 var connectionString = "devday-001.nokioc.0001.euw1.cache.amazonaws.com:6379";

services.AddSignalR();
  .AddStackExchangeRedis(connectionString, options => {
      options.Configuration.ChannelPrefix = "Brandersnatch";
  });

```