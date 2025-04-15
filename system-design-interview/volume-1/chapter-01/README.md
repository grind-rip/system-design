# Chapter 1: Scale from Zero to Millions of Users

This chapter describes how to scale a system from a single server to a system supporting millions of users. It begins with a basic setup where one server handles everything, then progressively introduces scaling techniques: separating web and database tiers, implementing load balancers, database replication (master-slave), caching, CDNs for static content, and creating a stateless web tier. For fault tolerance and reliability, this chapter details the use of multiple data centers, message queues for decoupling components, comprehensive logging and monitoring, and database sharding. The key principles for supporting millions of users include maintaining stateless web tiers, building redundancy, maximizing caching, utilizing CDNs, implementing database sharding, splitting the system into microservices, and employing monitoring and automation tools.

## Single server setup

First, we start with a single server. This server runs everthing: web app, database, cache, etc.

Request flow:
1. A client accesses the website through a domain name.
2. The Domain Name System (DNS) translates the domain name to an Internet
   Protocol (IP) address.
3. Hypertext Transfer Protocol (HTTP) requests are sent to the web server.
4. The web server returns an HTML or JSON response.

**NOTE**: Web applications typically return HTML to the client, whereas mobile applications typically return JSON.

## Database

Separating web/mobile traffic (web tier) and database (data tier) servers allows them to be scaled independently.

### Which database to use?

There are two common types of databases:
1. **Relational**: Relational databases, also called a relational database management system (RDBMS) or SQL database, represent and store data in tables and rows. You can perform join operations using SQL across different database tables.
2. **Non-Relational**: Non-Relational databases, also called NoSQL databases, are grouped into four categories: key-value stores, graph stores, column stores, and document stores. Join operations are generally not supported in non-relational databases.

Typically, you will want to use a relational database, however, non-relational databases might be the right choice if:
1. Your application requires super-low latency.
2. Your data are unstructured, or you do not have any relational data.
3. You only need to serialize and deserialize data (JSON, XML, YAML, etc.).
4. You need to store a massive amount of data.

### Vertical scaling vs. horizontal scaling

Vertical scaling, also referred to as "scale up", is the process of adding more power (CPU, memory, etc.) to your servers. Horizontal scaling, also referred to as "scale out", is the process of adding more servers into your pool of resources.

Vertical scaling is less complex, but there are physical limitations to CPU and memory. Also, vertical scaling does not accomodate failover and redundancy well. Horizontal scaling is more desirable for large scale applications due to the limitations of vertical scaling.

## Load balancer

In order to facilitate horizontal scaling, a load balancer is used. A load balancer evenly distributes incoming traffic among web servers that are defined in a load-balanced set. Clients connect to the public IP of the load balancer, which communicates with the web servers. Typically, the web servers are in a private network that is unreachable over the internet and only reachable via the load balancer.

## Database replication

To support failover and redundancy of the data tier, database replication is used. Database replication uses a master (or write replica), which generally only supports write operations (INSERT, UPDATE, DELETE) and a slave (read replica), which recieves data from the master database and only supports read operations. Since most applications require a much higher ratio of reads to writes, this has the advantage of better performance, better reliability, and high availability.

>How does routing to a write/read replica work?

This can done at the application level, using a dedicated proxy server, or DNS-based routing.

## Cache

A cache is a temporary storage area that stores the result of expensive responses or frequently accessed data in memory, so that subsequent requests are served more quickly. The benefits of having a separate cache tier include better system performance, the ability to reduce database workloads, and the ability to scale the cache tier independently. Consider using cache when data is read frequently, but modified infrequently.

## Content delivery network (CDN)

A Content delivery network (CDN) is a network of geographically dispersed servers used to deliver static content. CDN servers cache static content like images, videos, CSS, JavaScript files, etc.

>How does the closest CDN get used?

There are several mechanisms to connect you to the closest/optimal edge server: DNS-Based geolocation, Anycast routing, and real-time telemetry.

## Stateless web tier

In order to scaling the web tier horizontally, it must be stateless. A good practice is to store session data in persistent storage such as relational database or NoSQL. Therefore, each web server in the cluster can access state data from the data tier. This is called a stateless web tier. A stateful server remembers client data (state) from one request to the next. A stateless server keeps no state information. With a stateless web tier, autoscaling can be utilized based on traffic load.

## Data centers

Having multiple data centers allows for multi-region redundancy. GeoDNS can be used to direct traffic to the nearest data center depending on where a user is located. Asynchronous multi-data center replication is required to synchronize data across data centers.

## Message queue

Using a message queue is a key strategy employed by many real-world distributed systems to decouple its components.

A message queue is a durable component, stored in memory, that supports asynchronous communication. It serves as a buffer and distributes asynchronous requests.

The basic architecture of a message queue is simple. Input services, called producers/publishers, create messages, and publish them to a message queue. Other services or servers, called consumers/subscribers, connect to the queue, and perform actions defined by the messages.

## Logging, metrics, automation

* **Logging** helps to identify errors and problems in the system. You can use tools to aggregate logs into a centralized service for easy searching and viewing.
* **Monitoring** helps us to gain business insights and understand the health status of the system. useful metrics include:
  1. Host level metrics: CPU, memory, disk I/O, etc.
  2. Aggregated level metrics: For example, the performance of the entire database tier, cache tier, etc.
  3. Key business metrics: Daily active users (DAU), retention, revenue, etc.
* **Automation**: When a system gets big and complex, we need to build or leverage automation tools to improve productivity.

## Database scaling

Again, we can employ vertical and horizontal scaling.

Horizontal scaling, also known as sharding, is the practice of adding more servers. Sharding separates large databases into smaller, more easily managed parts called shards. Each shard shares the same schema, though the actual data on each shard is unique to the shard.

The most important factor to consider when implementing a sharding strategy is the choice of the sharding key. A sharding key (known as a partition key) consists of one or more columns that determine how data is distributed.

When choosing a sharding key, one of the most important criteria is to choose a key that can evenly distributed data.

## Millions of users and beyond

To conclude this chapter, we provide a summary of how we scale our system to support millions of users:
- Keep web tier stateless.
- Build redundancy at every tier.
- Cache data as much as you can.
- Support multiple data centers.
- Host static assets in CDN.
- Scale your data tier by sharding.
- Split tiers into individual services.
- Monitor your system and use automation tools.
