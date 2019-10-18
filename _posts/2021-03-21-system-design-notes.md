---
layout: post
title: "System Design Interview Notes"
categories: [All, Technical]
tags: [coding interview, interview, engineer interview, system design]
fullview: false
excerpt: My notes for how to approach system design problems
comments: true
---

# Introduction

Sometime ago when I was preparing for coding interviews, I wrote down a bunch of notes about system design. I shared with a friend who found them useful, so I'm sharing here, hoping that they help others as well.

**How to use**: I used them before the interview to get myself familiar with the approach, steps, and terminologies. This can prevent me from getting ahead of myself during the interview and help me (appear to) be more systematic.

_Disclaimer_: These are my personal notes, and comes with lots of assumptions or biases from my experience, and they may not be _complete_. Use with caution.

# Steps

[This AWS design has a good overall process](https://github.com/donnemartin/system-design-primer/blob/master/solutions/system_design/scaling_aws/README.md)

**All system evolution has to be based on benchmarking, profiling, and bottlenecks identified.**

## Step 1. Outline **use cases**, **constraints** and **assumptions**
  - Functional requirements
    - input / output
    - user registration
    - analytics / tracking (real-time?)
      - may use mapreduce on server logs
    - expiry
  - Storage (SQL vs. NoSQL)
    - Do back of the envelope calculation of storage requirements
    - Consistency
    - Structured?
    - Read-write ratio
  - Non-functional requirements
    - Security
    - HA / Reliability
    - Latency
    - Throughput

## Step 2. Sketch high level design
Usually start from C/S + DB + object store

## Step 3. Design core components
- Use relational database for hashing table
- Use S3 or mongo to store objects, e.g., files, images.


## Step 4. Scale the design
  - Break task into bg processing (Sidekiq)
  - Add CDN
  - Add LB
  - Single DB -> Master-slave
  - Add caching
  - Add micro-services
  - Message queues (MQ)

Note: Below, the number of `+` denotes the scalability level in a rough fashion.

### +
  - Use object store
    - Reasons: Static content takes up space / vertical scaling expensive

### ++
  - Add CDN -> read object store
    - Serve static content from CDN to reduce latency
  - Add LB / AWS ELB
    - balance loads -> latency
    - HA -> avoid single point of failure
    - SSL termination
  - Read / Write API
  - DB master - slave / read replica

### +++
  - Add memory cache
    - Try configuring MySQL / Postgres cache
    - Reduce read latency
    - Store session data

### ++++
  - Web server auto scaling
  - Add write queues
  - DB sharding / federation

## Security
- Authentication
- Encrypt the traffic / Https
- VPC
- Sanitize user inputs
- Use parameterized query to avoid SQL injection
- Least privilege design
- Rate limit requests to avoid DDoS / brute-force
- Try using unguessable UUID

Other concerns: https://github.com/shieldfy/API-Security-Checklist

# Other topics
- Messaging systems
  - **Simplicity**: Message producing decoupled from consuming
    - simplify the communication between components
  - **Better performance**
    - Producer: Produce and go
    - Consumer: Consume when ready
  - **Reliable** / Data persistence.
    - Service goes down, we don't lose data
  - **Scalable**
    - Increase the prod / consumer number easily
- Message queue vs. pub/sub
  - message queue
    - message is locked when being consumed.
    - put back in the queue if failed
  - pub / sub
- Kafka vs. RabbitMQ
  - Kafka: Distributed streaming platform
    - has no message broker
    - a purely pub / sub pattern
    - **Best for processing streams**
    - consumer responsible for **retry** logic
    - message order guaranteed within a partition
    - retains messages, consumed or not
    - not good for
      - no message filtering
      - no support of delayed / scheduled messages
  - RabbitMQ supports queuing and pub/sub mode
    - ephemeral and durable modes
    - **unordered messages** because failed messages go back in queue
    - Good for
      - built-in **retry** logic
      - message filtering
      - delayed / scheduled messages
    - evicts messages after consumed
- API gateway
- Service discovery

## Kafka vs. RabbitMQ: When to use?
RabbitMQ is preferable when we need:

    Advanced and flexible routing rules.
    Message timing control (controlling either message expiry or message delay).
    Advanced fault handling capabilities, in cases when consumers are more likely to fail to process messages (either temporarily or permanently).
    Simpler consumer implementations.

Kafka is preferable when we require:

    Strict message ordering.
    Message retention for extended periods, including the possibility of replaying past messages.
    The ability to reach a high scale when traditional solutions do not suffice.


## Relational vs. NoSQL

### NoSQL categories
- Key-value store
- JSON documents
- Column store
  - a set of nested-key/value pairs within a column
- Graph store

### NoSQL characteristics
- large volume
- low latency response
- Unstructured, semi-structured

![CAP theorem, RDMS, NoSQL](https://docs.microsoft.com/en-us/dotnet/architecture/cloud-native/media/cap-theorem.png)

### [When to use NoSQL](https://docs.microsoft.com/en-us/dotnet/architecture/cloud-native/relational-vs-nosql-data)
- high volume workloads that require large scale
- workloads don't require ACID guarantees
- data is dynamic and frequently changes
- Data can be expressed without relationships
- You need fast writes and write safety isn't critical
- Data retrieval is simple and tends to be flat
- Your data requires a wide geographic distribution
- Your application will be deployed to commodity hardware, such as with public clouds

## [Database federation](https://github.com/donnemartin/system-design-primer#federation) == functional partitioning

> Federation (or functional partitioning) splits up databases by function. For example, instead of a single, monolithic database, you could have three databases: forums, users, and products, resulting in less read and write traffic to each database and therefore less replication lag. Smaller databases result in more data that can fit in memory, which in turn results in more cache hits due to improved cache locality. With no single central master serializing writes you can write in parallel, increasing throughput.

## [DB sharding](https://github.com/donnemartin/system-design-primer#sharding)

> Sharding distributes data across different databases such that each database can only manage a subset of the data. Taking a users database as an example, as the number of users increases, more shards are added to the cluster.

Similar to the advantages of federation, sharding results in less read and write traffic, less replication, and more cache hits. Index size is also reduced, which generally improves performance with faster queries.