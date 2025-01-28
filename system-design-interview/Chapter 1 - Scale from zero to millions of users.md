### Single server setup
Flow of a request in a single server setup:
1. Request to DNS to retrieve IP address of webserver
2. API request to your page using Hypertext Transfer Protocol (HTTP)
3. Web server returns HTML pages or JSON response for rendering

### Database
As userbase grows, we separate web/mobile traffic and database servers

Which database to use?
- **Relational** (relational database management system - RDBMS or SQL) - perform joins using SQL across different tables
	- MySQL, Oracle, PostgreSQL, etc.
- **Non-relational** (NoSQL) - join operations are generally not supported
	- Four categories: key-value, graph, column and document stores
	- CouchDB, Neo4j, Cassandra, Amazon DynamoDB, etc.

Non-relational databases might be the right choice if:
- Your application requires super low latency
- Your data are unstructured/does not have relational data
- You only need to serialize/deserialize data (JSON, XML, YAML, etc.)
- You need to store a massive amount of data

### Vertical vs. horizontal scaling
**Vertical** ("scale up"): Adding more power (CPU, RAM, etc.) to your servers
- Only should be used in low traffic use cases
- Has a hard limit
- Does not have failover and redundancy
**Horizontal** ("scale out"): Adding more servers into your pool
- More desirable for large scale applications

### Load balancer
Load balancers evenly distribute incoming traffic across webservers that are defined in a **load-balanced set**.

Flow of a request with a load balanced application:
1. User gets the IP address of the load balancer from DNS
2. Users connect to the public IP of the load balancer (web servers are no longer reachable by public internet)
3. The load balancer distributes traffic to the server using **private** IPs (can handle many servers in the webserver pool gracefully)

A **private IP** is only reachable between servers in the same network.

The webserver now has improved availability due to fixing its failover issue.

### Database replication
Database replication usually involves a master (the original DB) and the slaves (copies of the DB).

A master database generally only supports **writes** (inserts, deletes, updates), whereas slaves receive copies of the data from the master and only support read **operations**.

Advantages of database replication:
- Performance: Increased parallel operations across slave nodes
- Reliability: Maintain copies of data across multiple locations
- High availability: Can access other database severs when one is down
	- If only one slave is available and goes offline, read operations temporarily directed to master (slave replaced once issue is found). If multiple available, read operations redirected to other healthy slave databases.
	- If master goes offline, slave is promoted to the new master and slave database is replaced with a new one. Note in reality, can be more complicated due to out of sync data.


Now we can talk about improving load/response time.

### Cache
