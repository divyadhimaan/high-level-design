# System Design

This repository contains notes, diagrams, and code snippets created while learning system design concepts. It's intended as a personal reference for revision and interview preparation. Topics include high-level design patterns, scalability principles, common system architectures, and more.

## Table of Contents

- [Concepts](#concepts)
- [Additional Concepts](#additional-concepts)
- [System Design Problems](#system-design-problems)
- [Resources](#resources)
- [Articles](#articles)
- [How to Use This Repo](#how-to-use-this-repo)
- [Contributing](#contributing)
- [Glossary](#glossary)
- [Key Differences](#key-differences)

## Concepts
- [Scalability: Basics](./concepts/scalability.md)
- [Load Balancing](./concepts/load-balancing.md)
- [Databases](./concepts/databases.md)
- [Consistency in Distributed Systems](./concepts/consistency.md)
- [Caches](./concepts/caching.md)
- [Networks](./concepts/networks.md)
- [Data Replication and Migration](./concepts/data-replication-and-migration.md)
- [Security in Distributed Systems](./concepts/security.md)
- [Observability in Distributed Systems](./concepts/observability.md)
- [Distributed Consensus](./concepts/distributed-consensus.md)
- [Rate Limiting](./concepts/rate-limiting.md)
- [Tradeoffs](./concepts/tradeoffs.md)

## Additional Concepts
- [API design](./concepts/api-design.md)
- [Kafka](concepts/interview-specific/kafka.md)
- [Message Queue in Event Driven Architectures](concepts/interview-specific/message-queue.md)
- [Idempotency in Distributed Systems](concepts/interview-specific/idempotency.md)
- [Redis & Memcached](concepts/interview-specific/redis-memcached.md)


## System Design Problems
- [Email Service](./design-problems/emailing-service/main.md)
- [Tinder Design - Recommendation](./design-problems/tinder/main.md)
- [Design a Load Balancer](./design-problems/load-balancer/main.md)
- [Google Docs - Collaborative Editor Design](./design-problems/google-docs/main.md)
- [Uber - Cab Aggregator App](./design-problems/uber/main.md)
- [Workflow Management and Recommendation](./design-problems/workflow-management/main.md)
- [Whatsapp Calling](./design-problems/whatsapp-calling-app/main.md)
- [Design a Rate Limiter](./design-problems/rate-limiter/main.md)
- [Design a Consistent Hashing](./design-problems/consistent-hashing/main.md)
- [Design a URL shortener](./design-problems/url-shortener/main.md)
- [Design a Key Value Store](./design-problems/key-value-store/main.md)
- [Design an Unique ID generator](./design-problems/unique-id-generator/main.md)
- [Design a Web Crawler](./design-problems/web-crawler/main.md)
- [Design a Notification System](./design-problems/notification-system/main.md)
- [Design a News Feed System](./design-problems/news-feed-system/main.md)
- [Design a Chat Application](./design-problems/whatsapp-design/main.md)
- [Design a Youtube-like Video Streaming Service](./design-problems/video-streaming-service/main.md)
- [Design Google Drive](./design-problems/google-drive/main.md)
- [Design a Payment System](./design-problems/payment-gateway/main.md)
- [Design a Local Delivery Service](./design-problems/delivery-service/main.md)

## Interview Related Resources
- [Database Selection](./concepts/interview-specific/database-selection.md)

## Resources
A collection of materials referred to while learning:

- System Design Course by Gaurav Sen
- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- [System Design Interview - Alex Xu pdf (Part 1)](./resources/system-design-interview-an-insider-guide-by-Alex-Yu.pdf)
- [Designing Data Intensive Applications](./resources/designing-data-intensive-applications.pdf)
- [System Design - The Big Archive](./resources/system-design-the-big-archive.pdf)
- [Hello Interview](https://www.hellointerview.com)
<!-- - [Grokking the System Design Interview](https://www.designgurus.io/course/system-design) -->

## Articles
- [Optimizing Netflix APIs](https://netflixtechblog.com/optimizing-the-netflix-api-5c9ac715cf19)
- [Service Discovery](https://www.f5.com/company/blog/nginx/service-discovery-in-a-microservices-architecture)
- [Estimations - Numbers everyone should know](https://highscalability.com/numbers-everyone-should-know/)

## How to Use This Repo
- Browse by topic directories or use the table of contents above.
- Diagrams are stored under `/diagrams` and topic-specific folders.
- Use markdown files as revision notes before interviews.
- Refer to example architectures for designing systems in projects.


## Contributing
This is a personal learning repo, but feel free to open issues or PRs if you spot something helpful to add.

---

## Glossary

- [Latency](#latency)
- [Throughput](#throughput)
- [CAP Theorem (Brewer's Theorem)](#cap-theorem-brewers-theorem)
  - [CP - consistency and partition tolerance](#cp---consistency-and-partition-tolerance)
  - [AP - availability and partition tolerance](#ap---availability-and-partition-tolerance)
- [ACID Properties](#acid-properties)
- [Consistent Hashing](#consistent-hashing)
- [Sharding](#sharding)
- [FFMPEG](#ffmpeg)
- [AVI](#avi)
- [Homebrew](#homebrew)
- [HLS](#hls)
- [DASH](#dash)
- [PSTN (Public Switched Telephone Network)](#pstn-public-switched-telephone-network)
- [VoIP (Voice over Internet Protocol)](#voip-voice-over-internet-protocol)
- [SIP (Session Initiation Protocol)](#sip-session-initiation-protocol)
- [NTP (Network Time Protocol)](#ntp-network-time-protocol)
- [Redis Lua Scripts](#redis-lua-scripts)
### Latency
Latency refers to the time it takes for a request to travel from its point of origin to its destination and receive a response. 
It combines a number of delays - Response times, transmission, and processing time.

---

### Throughput
The rate at which a system, process, or network can move data or carry out operations in a particular period of time is referred to as throughput. Bits per second (bps), bytes per second, transactions per second, etc. are common units of measurement. 
It is computed by dividing the total number of operations or objects executed by the time taken.
---

### CAP Theorem (Brewer's Theorem)
In a distributed computer system, you can only support two of the following guarantees:

- `Consistency` - Every read receives the most recent write or an error (All nodes see the same data at the same time.)
- `Availability` - Every request receives a response, without guarantee that it contains the most recent version of the information (The system is always responsive.)
- `Partition Tolerance` - The system continues to operate despite network failures or delays.
(No data loss or crash due to communication break between nodes.)

Networks aren't reliable, so you'll need to support partition tolerance. A distributed system must be partition-tolerant (P). So, a practical tradeoff is between Consistency (C) and Availability (A).

#### CP - consistency and partition tolerance
- Waiting for a response from the partitioned node might result in a timeout error. 
- CP is a good choice if your business needs require atomic reads and writes.
- Banking systems prefer CP

#### AP - availability and partition tolerance
- Responses return the most readily available version of the data available on any node, which might not be the latest. 
- Writes might take some time to propagate when the partition is resolved.
- social media feeds prefer AP.

#### Choosing between CP and AP
- Example
  - If there are 3 servers (n1, n2, n3) and a client wants to write a key-value pair (k1, v1). Lets say k1 is written to n3, but not propagated yet to n1 and n2.
  - Now, if n3 goes down and a client wants to read k1.
    - In `CP` system, the client will not get a response because it cannot guarantee consistency. It will wait until n3 is back up.
    - In `AP` system, the client will get a response from n1 or n2, but it may not be the most recent value of k1.
- For a key-value store, we can choose either `CP` or `AP` based on the use case.
- Choosing between AP and CP depends on the specific requirements of the application.
- For example:
  - A banking application would prefer `CP` to ensure data consistency.
  - A social media application would prefer `AP` to ensure availability.
---

### ACID Properties
ACID is a set of properties that ensure reliable, consistent, and safe transactions in a database system.

| Property    | Ensures...                           | Example                          | Use Case                                 |
|-------------|--------------------------------------|----------------------------------|------------------------------------------|
| Atomicity   | All or none execution                | Debit fails → entire txn rolled back | Bank transfers                          |
| Consistency | Valid state transitions              | No violation of constraints      | E-commerce inventory updates             |
| Isolation   | No interference between transactions | Prevents dirty reads/race conditions | Online booking systems                  |
| Durability  | Changes survive system failures      | Data saved after commit          | Order confirmations in retail apps      |

---
### Consistent Hashing
Consistent Hashing maps servers to the key space and assigns requests(mapped to relevant buckets, called load) to the next clockwise server. Servers can then store relevant request data in them while allowing the system flexibility and scalability.

---

### Sharding

- Database sharding is a horizontal scaling technique used to split a large database into smaller, independent pieces called shards.
- These shards are then distributed across multiple servers or nodes, each responsible for handling a specific subset of the data.

- The sharding process involves several key components including:
  - **Sharding Key**: The shard key is a unique identifier used to determine which shard a particular piece of data belongs to. It can be a single column or a combination of columns. 
  - **Data Partitioning**: Partitioning involves splitting the data into shards based on the shard key. Each shard represents a portion of the total data set. Common strategies to partition database are range-based, hash-based, and directory-based sharding. 
  - **Shard Mapping**: Creating a mapping of shard keys to shard locations. 
  - **Shard Management**: The shard manager oversees the distribution of data across shards, ensuring data consistency and integrity. 
  - **Query Routing**: The query router intercepts incoming queries and directs them to the appropriate shard(s) based on the shard key.
- Sharding Strategies
  - Hash-Based Sharding: Data is distributed using a hash function, which maps data to a specific shard.
  - Range-Based Sharding: Data is distributed based on a range of values, such as dates or numbers.
  - Geo-Based Sharding: Data is distributed based on geographic location, allowing for improved performance and reduced latency for users in different regions.
  - Directory-Based Sharding: A lookup table is used to map data to specific shards.

### FFMPEG
FFmpeg is a powerful, free, and open-source multimedia framework used for decoding, encoding, transcoding, and streaming audio and video files.


---
### AVI

AVI, or Audio Video Interleave, is a multimedia container format developed by Microsoft to store both audio and video data in a single file, allowing for synchronized playback. 

---

### Homebrew

Homebrew is a package manager for macOS (and Linux) that simplifies the installation, updating, and management of software on your system.

---
### HLS
- HLS, or HTTP Live Streaming, is a widely used video streaming protocol developed by Apple that delivers audio and video content over the internet. 
- It's known for its adaptability to changing network conditions, reliability, and compatibility with various devices and browsers. 
- HLS works by breaking down video files into smaller segments, which are then downloaded and played sequentially by a video player.
---

### DASH

- Dynamic Adaptive Streaming over HTTP (DASH) is a streaming technology that adapts video quality in real-time based on network conditions, delivering high-quality video over the internet using standard HTTP servers. 
- It achieves this by breaking down video content into segments, each with varying bitrates, and allowing the client device to dynamically select the most appropriate segment for the current network bandwidth

---

### PSTN (Public Switched Telephone Network)
- Traditional circuit-switched telephone network.
- Requires **telephone lines** and **dedicated bandwidth** per call.
- **Cost depends** on distance and duration.
- Generally **reliable but expensive**.
- Example: Landline phone systems.

---
### VoIP (Voice over Internet Protocol)
- Transmits voice calls over the **internet** using **packet-switching**.
- Requires only **internet connectivity** (no phone lines).
- **Cost-effective** and often **free**.
- **Scalable and flexible**, but quality depends on internet speed.
- Example: Skype, Zoom, WhatsApp calls.

---
### SIP (Session Initiation Protocol)
- **Signaling protocol** used to establish, manage, and terminate **VoIP** calls.
- Handles **call setup, teardown, and modifications**.
- Works with other protocols (e.g., RTP for media transmission).
- Used for **voice, video, and messaging sessions** over IP.
- Example: Used in VoIP softphones and enterprise telephony systems.

---

### NTP (Network Time Protocol)
- Protocol for synchronizing clocks of computer systems over packet-switched, variable-latency data networks.
- Uses a hierarchical system of time sources, with primary servers connected to highly accurate reference clocks.
- Ensures accurate timekeeping for applications like logging, authentication, and scheduling.
- Example: Synchronizing server times in data centers.
---
### Redis Lua Scripts
- Lua = lightweight, high-performance, embeddable scripting language. 
- Designed to be fast, portable, and simple.
- Often used in:
  - Game engines (e.g., Roblox, World of Warcraft mods). 
  - Embedded systems. 
  - Databases (e.g., Redis) for scripting logic.
- In System Design, Redis supports Lua scripting to run custom logic atomically inside Redis.
- Instead of multiple round-trips (client ↔ Redis), you can:
  - Upload Lua script.
  - Execute on server in one step.
- Why Lua Scripts:
  - Ensures atomicity: script executes as a single operation.
  - Reduce network overhead: One call instead of multiple client-server requests.
- Example:
  - Say you want to check if a key exists, and if not, set it.
  - Without Lua:
    - Client does GET key. 
    - If not exists → does SET key value. 
    - Problem: race condition (another client may set in between).
  - With Lua:
  ```lua
  if redis.call("EXISTS", KEYS[1]) == 0 then
  return redis.call("SET", KEYS[1], ARGV[1])
  else
  return nil
  end
  ```
  - Call from client
  ```bash
  EVAL "if redis.call('EXISTS', KEYS[1]) == 0 then return redis.call('SET', KEYS[1], ARGV[1]) else return nil end" 1 myKey myValue
  ```
  
---

## Key Differences

- [Microservices vs Monolithic Architectures](./concepts/Differences/microservices-vs-monoliths.md)