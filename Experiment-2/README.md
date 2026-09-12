# Experiment 2 — System Design of Netflix

**Name:** Swaroop Kumar
**UID:** 24BCS12717
**Subject:** System Design

---

## 1. Objective

To design a scalable and highly available video streaming platform similar to Netflix. The system should support millions of users, provide low-latency video playback, handle large-scale traffic, and maintain high availability.

The design covers:

* Functional and non-functional requirements
* Capacity estimation
* API design
* High-level architecture
* Media streaming pipeline
* Database and caching strategy
* Scalability and fault-tolerance mechanisms

---

## 2. Functional Requirements

| #  | Requirement                         |
| -- | ----------------------------------- |
| 1  | User registration and login         |
| 2  | Multiple user profiles              |
| 3  | Browse movies and TV shows          |
| 4  | Search movies and shows             |
| 5  | Play, pause, resume and seek videos |
| 6  | Adaptive video quality              |
| 7  | Continue watching                   |
| 8  | Watch history                       |
| 9  | My List / Watchlist                 |
| 10 | Subscription and payment management |
| 11 | Personalized recommendations        |
| 12 | Notifications for new content       |

---

## 3. Non-Functional Requirements

| Requirement         | Target                                       |
| ------------------- | -------------------------------------------- |
| Availability        | 99.99%+                                      |
| Scalability         | Millions of concurrent users                 |
| API Latency         | < 200–500 ms                                 |
| Video Startup       | ~1–3 seconds                                 |
| Fault Tolerance     | No single point of failure                   |
| Durability          | No loss of critical user data                |
| Global Availability | Multi-region deployment                      |
| Security            | Authentication, authorization and encryption |
| Performance         | Efficient content delivery through CDN       |

---

# 4. Capacity Estimation

## 4.1 User Estimation

| Metric                   | Estimated Value |
| ------------------------ | --------------: |
| Total registered users   |     300 Million |
| Daily Active Users (DAU) |     100 Million |
| Peak concurrent users    |      30 Million |
| Average session duration |       1.5 hours |

Assuming approximately **30% of DAU** can be active simultaneously:

**Peak concurrent users = 100M × 30% = 30M users**

---

## 4.2 API Request Estimation

Assume each active user generates approximately **10 API requests per minute**.

Peak request rate:

**30M × 10 / 60 ≈ 5M requests/second**

Adding a 2× capacity buffer:

**Required capacity ≈ 10M requests/second**

Therefore, the backend should be designed to handle approximately **10 million requests per second** during peak traffic.

---

## 4.3 Video Streaming Bandwidth

Assume an average streaming bitrate of **5 Mbps per user**.

Peak bandwidth:

**30M × 5 Mbps = 150 Tbps**

With 2× headroom:

**Required bandwidth ≈ 300 Tbps**

Such bandwidth cannot efficiently be served directly from application servers. Therefore, a distributed **CDN** is required to deliver video content close to users.

---

## 4.4 Video Storage

Assume:

* Approximately 100,000 titles
* Average encoded size = 5 GB per title

Base storage:

**100K × 5 GB = 500 TB**

Multiple resolutions, codecs, audio tracks and subtitles can increase storage requirements significantly.

Estimated storage:

**≈ 5 PB**

Considering replicas, 4K content and future growth:

**≈ 5–50 PB**

Object storage is therefore preferred for storing large media files.

---

## 4.5 User Data Storage

Assume:

* 100M daily active users
* Approximately 10 KB of basic user/profile information per user

Base storage:

**100M × 10 KB ≈ 1 TB**

After including:

* Watch history
* Continue-watching information
* User preferences
* Devices
* Watchlists
* Session information

Estimated storage:

**≈ 5–50 TB**

---

## 4.6 Cache Requirements

Frequently accessed data should be cached to reduce database load.

Examples:

* Movie/show metadata
* Popular searches
* Recommendations
* User sessions
* Frequently accessed configuration

Estimated cache requirement:

**100 GB – 1 TB+ per region**

A distributed cache such as Redis can be used.

---

## 4.7 Availability

Target availability:

**99.99%+**

To achieve this:

* Deploy services across multiple availability zones
* Use multiple regions
* Replicate critical data
* Use automatic failover
* Use redundant load balancers
* Use multiple CDN locations
* Avoid single points of failure

---

# 5. Core API Endpoints

| Method | Endpoint                      | Description                      |
| ------ | ----------------------------- | -------------------------------- |
| POST   | `/api/v1/auth/login`          | Login or register user           |
| GET    | `/api/v1/home`                | Get homepage and recommendations |
| GET    | `/api/v1/search?q={query}`    | Search movies and shows          |
| GET    | `/api/v1/content/{contentId}` | Get content details              |
| POST   | `/api/v1/playback`            | Create playback session          |
| POST   | `/api/v1/history`             | Update watch progress            |
| GET    | `/api/v1/my-list`             | Get user's watchlist             |
| POST   | `/api/v1/my-list/{contentId}` | Add/remove content               |
| GET    | `/api/v1/subscription`        | Get subscription details         |
| POST   | `/api/v1/payment`             | Process subscription payment     |

---

# 6. High-Level Architecture

```text
                    ┌─────────────────┐
                    │     Clients     │
                    │ Web / Mobile / TV│
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   DNS / GeoDNS  │
                    └────────┬────────┘
                             │
                             ▼
                 ┌────────────────────────┐
                 │ External Load Balancer  │
                 │   + DDoS Protection     │
                 └───────────┬────────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   API Gateway   │
                    └────────┬────────┘
                             │
                             ▼
               ┌────────────────────────────┐
               │    Internal Load Balancer │
               └─────────────┬──────────────┘
                             │
        ┌────────────────────┼─────────────────────┐
        │                    │                     │
        ▼                    ▼                     ▼
 ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
 │ Auth/User   │      │   Search    │      │Recommendation│
 │  Service    │      │   Service   │      │   Service   │
 └─────────────┘      └─────────────┘      └─────────────┘

        ┌─────────────┐      ┌─────────────┐
        │Subscription │      │  Playback   │
        │   Service   │      │   Service   │
        └─────────────┘      └──────┬──────┘
                                    │
                                    ▼
                             ┌─────────────┐
                             │     CDN     │
                             └──────┬──────┘
                                    │
                                    ▼
                               End Users
```

The architecture follows a **microservices-based design**. Each major business function is separated into an independent service so that services can be scaled and deployed independently.

---

# 6.1 Media Playback Pipeline

```text
User
 │
 ▼
Playback Service
 │
 ├── Authentication
 ├── Subscription Check
 └── Generate Signed Playback URL
 │
 ▼
CDN / Open Connect
 │
 ▼
Video Segments
 │
 ▼
User Device
```

Content preparation occurs before delivery:

```text
Original Video
      │
      ▼
 Transcoding
      │
      ▼
Multiple Resolutions
      │
      ▼
DRM + Encryption
      │
      ▼
Object Storage
      │
      ▼
CDN
```

Adaptive Bitrate Streaming (ABR) allows the client to automatically switch between different video qualities according to available network bandwidth.

---

# 6.2 Search Pipeline

```text
User
 │
 ▼
API Gateway
 │
 ▼
Search Service
 │
 ▼
OpenSearch / Elasticsearch
 │
 ├── Keyword Search
 ├── Fuzzy Search
 └── Typo Correction
 │
 ▼
Search Results
```

A dedicated search engine prevents frequent search operations from putting unnecessary load on the primary database.

---

# 6.3 Notification Pipeline

```text
Microservices
      │
      ▼
 Message Queue
(Kafka / SQS)
      │
      ▼
Notification Workers
      │
 ┌────┴───────────┐
 ▼                ▼
Push             Email
FCM/APNs         Service
      │
      ▼
 Dead Letter Queue
       (DLQ)
```

Message queues make notification processing asynchronous. If a notification fails, it can be retried without affecting the main user request.

---

# 6.4 Caching and Database Layer

```text
                Microservice
                     │
                     ▼
              ┌─────────────┐
              │    Redis    │
              │    Cache    │
              └──────┬──────┘
                     │
              Cache Miss
                     │
                     ▼
          ┌────────────────────┐
          │ Database Cluster   │
          │ Sharding + Replicas│
          └────────────────────┘
```

Caching frequently accessed information reduces database load and improves response time.

---

# 7. Database Design

A **polyglot persistence** approach is used because different types of data have different storage requirements.

| Database                   | Type          | Main Use                                                       |
| -------------------------- | ------------- | -------------------------------------------------------------- |
| PostgreSQL                 | SQL           | Payments, subscriptions and transactional data                 |
| Cassandra                  | NoSQL         | Watch history, playback progress and high-volume user activity |
| Redis                      | In-memory     | Cache, sessions and frequently accessed data                   |
| OpenSearch / Elasticsearch | Search Engine | Movie and show search                                          |
| Object Storage             | File Storage  | Video files, subtitles and media assets                        |

### PostgreSQL

Used for data where strong consistency and transactions are important, such as:

* Payment records
* Subscription information
* Account-related transactional data

### Cassandra

Used for high-volume distributed data such as:

* Watch history
* Resume position
* Device information
* User activity

### Redis

Used for low-latency access to frequently requested data:

* Sessions
* Popular content
* Metadata
* Recommendations

---

# 8. Key Design Decisions

### 1. CDN-Based Video Delivery

Video files are delivered through a distributed CDN instead of application servers. This reduces latency and prevents the backend from becoming a bottleneck.

### 2. Microservices Architecture

Services such as authentication, search, playback, payment and recommendations are separated. Each service can be independently scaled and deployed.

### 3. API Gateway

The API Gateway acts as the main entry point for client requests and handles:

* Routing
* Authentication
* Rate limiting
* Request validation
* Security controls

### 4. Message Queues

Kafka/SQS can be used for asynchronous tasks such as notifications and background processing.

This reduces coupling between services and improves fault tolerance.

### 5. Database Replication and Sharding

Replication improves availability and read performance, while sharding distributes large datasets across multiple database nodes.

### 6. Distributed Caching

Redis reduces repeated database queries and improves API response times for frequently accessed information.

### 7. Adaptive Bitrate Streaming

Multiple video qualities allow the system to adapt playback to the user's network conditions and device capabilities.

### 8. DRM and Encryption

DRM and encryption protect premium video content from unauthorized access and distribution.

---

# 9. Scalability and Fault Tolerance

The system can scale horizontally by adding more instances of individual services.

For example:

```text
                 Load Balancer
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      Instance 1   Instance 2   Instance 3
          │            │            │
          └────────────┼────────────┘
                       ▼
                  Shared Data
```

If one instance fails, traffic can automatically be redirected to healthy instances.

Additional reliability mechanisms include:

* Health checks
* Automatic failover
* Database replication
* Multi-region deployment
* Retry mechanisms
* Circuit breakers
* Dead Letter Queues
* CDN redundancy

---

# 10. Security

The system should implement:

* HTTPS/TLS for communication
* Secure password hashing
* Authentication using tokens/sessions
* Role-based authorization
* Rate limiting
* DDoS protection
* Encryption of sensitive data
* Signed URLs for video access
* DRM for protected content
* Secure payment processing

---

# 11. Conclusion

This experiment presents a scalable system design for a Netflix-like video streaming platform. The architecture uses microservices, API gateways, distributed caching, database replication, message queues and a global CDN to support large-scale traffic.

The design targets approximately **10M requests/second**, **30M concurrent users**, and hundreds of terabits per second of potential video bandwidth. Separating control-plane services from media delivery allows the system to scale efficiently while maintaining low playback latency.

The combination of **CDN-based delivery, adaptive bitrate streaming, polyglot databases, caching, replication and fault-tolerant services** provides the scalability, availability and performance required by a modern global streaming platform.
