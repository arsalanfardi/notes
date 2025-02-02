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


Now we can talk about improving load/response time by adding a cache layer and shifting static content to the **content delivery network (CDN)**.

### Cache
A temporary storage area that stores the result of expensive operations or frequently accessed data in memory. Caches are much faster than databases and reduce database workloads but are not a persistent data store.

**Read-through cache strategy**: Webserver receives a request and checks if the cache has the available response, if not it will query the database and store that response in the cache before returning it to the client. 

Considerations of using a cache:
- Deciding when to use it: Consider using a cache when data is read frequently but modified infrequently
- Expiration policy: Caches require a reasonable expiration policy which balances reducing database reads and preventing stale data
- Consistency: Caches can become out of sync with the data store since data-modifying operations on the data store and cache are not in a single transaction - maintaining this consistency at scale is challenging
- Mitigating failures: Multiple cache servers are recommended to avoid a single point of failure, also consider overprovisioning the memory to allow for a buffer.
- Eviction policy: Once a cache is full existing items may need to be removed, this is cache eviction. Policies exist such as:
	- Least-recently-used (LRU)
	- Lease-frequently used (LFU)
	- First in First out (FIFO)

### Content delivery network (CDN)
A network of geographically dispersed servers used to deliver static content (JavaScript/CSS/image/video files, etc.).

When a user visits a website, a CDN server closest to the user will deliver static content. The further from that server, the slower the website loads. 

Flow of a request:
1. User A tries to get `image.png` by using an image URL, the domain of the URL is provided by a CDN provider (e.g. `https://mysite.cloudfront.net/logo.jpg`)
2. If `image.png` does not exist in cache the CDN server requests the file from the webserver/online storage origin.
3. The origin returns the image and includes optional HTTP header Time-to-Live to describe how long the image should be cached
4. CDN caches the image and returns to user A, keeping the image in cache until TTL expires
5. User B requests the same image and is returned from the CDN assuming TTL has not expired.

Considerations of using a CDN:
- Cost: CDNs are usually provided by third-parties and are charged for in/out data transfers. Consider excluding infrequently used assets.
- Appropriate cache expiry time
- CDN fallback: Detect outages and request resources from the origin instead
- Invalidating files: Remove files from the CDN before expiry by:
	- Invalidating the object using APIs provided by CDN vendors
	- Use object versioning to serve a different version of the object (e.g. `image.png?v=2`)

### Stateless web tier
To scale the web tier horizontally we need to ensure state (e.g. user session data) is moved out of the web tier. In a stateful architecture, state is specific to a server meaning all requests from a certain user would need to be sent to the same server. This makes scaling horizontally difficult.

In a stateless architecture, HTTP requests from users can be sent to any web servers which fetch data from a shared data store, for example an easily scalable NoSQL database for storing session data.

### Data centers
Users can be geoDNS-routed (AKA geo-routed) to their closest data center. **geoDNS** is a service that allows domain names to be resolved to IP addresses based on the location of a user.

In case of an outage at one data center, traffic is redirected to a healthy one.

Considerations of multi-data center setup:
- Traffic redirection: GeoDNS or other tools to direct traffic to correct data center
- Data synchronization: Different regions could use databases/caches specific to that region. Consider data replication across multiple data centers to prevent problems during failovers.
- Test and deployment: Important to test at different locations and have automated deployment tools to keep services consistent.

Now we need to decouple different components of the system so that they can be scaled independently.
### Message queue
Serves as an in-memory bugger for distributing asychronous requests. Input services (producers/publishers) create messages and publish them to a message queue. Consumers/subscribers connect to the queue and perform actions defined by messages.

Producers can post message even when the consumer is unavailable to process it, and similarly consumers can read messages even when the producer is unavailable. When the size of the queue becomes large, more workers can be added to reduce processing time.

### Logging, metrics, automation
Logging: Monitoring error logs is important to identify and errors and problems in the system. You can monitor them at the server level or use tools to aggregate them to a centralized service for easy search and viewing.

Metrics: Collecting different types of metrics helps us gain business insight and understand health of the system.
- Host level metrics: CPU, memory, disk I/O, etc.
- Aggregated level metrics: e.g. performance of the entire database/cache tier
- Key business metrics: daily average users, retention, revenue, etc.

Automation: Adding continuous integration and continuous deployment (CI/CD) to verify check-ins and detect problems early.

### Database scaling

**Vertical scaling** (scaling up):
- Adding more power (CPU, RAM, DISK, etc.)
- Can run into hardware limits where for large enough user bases a single sever is not enough
- Greater risk of single point of failure
- Overall cost for more powerful servers is expensive 

**Horizontal scaling** (scaling out):
- Adding more servers or "shards"
- Shards share the same schema but the data on it is unique to the shard
- Data is allocated to a database by using a hash function on a column, the **sharding key** (AKA **partition key**)
- When choosing a partition key, it's important to choose one that can evenly distribute data.

While sharding is a great technique, it introduces complexities to the system:
- Resharding data: This is required when shards run out of space due to either rapid data growth or uneven data distribution. Requires updating the sharding function and moving data around. Consistent hashing can be used to solve this problem.
- Celebrity (or hotspot key) problem: Excessive access to a specific shard with popular keys causes server overload. Requires allocating a shard for each key, and maybe even further partitioning.
- Joins and de-normalization: After sharding across multiple servers, it's difficult to perform join operations across database shards. Common workaround is denormalizing the database so that queries are on a single table.
