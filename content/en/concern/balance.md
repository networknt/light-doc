---
title: "Load Balance"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

The light-4j platform encourages client-side discovery in order to avoid proxies in front of multiple instances of services. This can reduce the network hop and subsequently reduce the latency of service calls.
 
Client-side discovery needs a client-side load balancer to pick up one and only one available service instance from a list of available services for a particular downstream request during runtime. 

Currently, Round-Robin and LocalFirst have been implemented and ConsistentHashing is in progress. By default, round-robin will be used and it gives all service instances an equal opportunity to be invoked.

### LoadBalance interface

All load balances will be implementing the LoadBalance interface.

```
public interface LoadBalance {
    // select one from a list of URLs

    /**
     * Select one url from a list of url with requestKey as optional.
     *
     * @param urls List
     * @param requestKey String
     * @return URL
     */
    URL select(List<URL> urls, String requestKey);

    /**
     * return positive int value of originValue
     * @param originValue original value
     * @return positive int
     */
    default int getPositive(int originValue){
        return 0x7fffffff & originValue;
    }

}

```

### RoundRobinLoadBalance

The Round Robin load balance will pick up a URL from a list of URLs one by one for each service call. It will distribute the load equally to all URLs in the list.

This class has an instance variable called idx which is AtomicInteger. It increases for every select call to make sure all URLs in the list will have an opportunity to be selected.

The assumption for Round Robin is based on the principle that all servers will have the same hardware/cloud resource configuration so that they can be treated as the same priority without any weight.
 
The Round Robin requestKey is not used and should be null. The URL will be selected from the list based on an instance idx.
 
### LocalFirstLoadBalance

The local first load balance gives local services a higher priority than remote services. If there is no local service available, then it will adopt a round-robin strategy.

With all the services in the list of URLs, find local services with IP. Chances are we have multiple local services. Round Robin will be used in this case. If there is no local service, find the first remote service according to round robin.

The local first requestKey is not used as it is IP on the localhost. It first needs to find a list of URLs on the localhost for the service, and then round robin in the list to pick up one.

Currently, this load balance is only used if you deploy the service as a standalone Java process on data center hosts. We need to find a way to identify two VMs or two docker containers sitting on the same physical machine in the future to improve this load balance.

It is also suitable if your services are built on top of [light-hybrid-4j](https://github.com/networknt/light-hybrid-4j) and you want to use the remote interface for service to service communication.

### ConsistentHashLoadBalance

To obtain maximum scalability, microservices allow the Y-Axis scale to break up a big monolithic application into small functional units. However, for some of the heavy load services, we can use data sharding for the Z-Axis scale. This load balancer is designed for that.

In a typical case, the requestKey should be a client_id or user_id from a JWT token. This can guarantee that the same client will always be routed to one service instance or one user will always to be routed to one service instance. However, this key can be a combination of multiple fields from the request.

This load balancer is not yet completed. There are some articles talking about this on the Internet but the majority of the implementations suffer from imbalance because of the random nature of hashing.

Here is a paper that resolves the issue. 

```
https://medium.com/vimeo-engineering-blog/improving-load-balancing-with-a-new-consistent-hashing-algorithm-9f1bd75709ed
https://arxiv.org/abs/1608.01350
```

### Configuration

We have several load balancer implementations available, however, there is only one should be used for one client during runtime, and it is controlled by the service.yml config file.

Here is an example that uses RoundRobinLoadBalance. 

```
# Default load balance is round robin and it can be replace with another implementation.
---
description: singleton service factory configuration
singletons:
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
```
