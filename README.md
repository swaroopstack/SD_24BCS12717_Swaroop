# Experiment — High-Level Design of a URL Shortener (Bitly)

> **Course:** System Design  
> **Student:** Swaroop Kumar — 24BCS12717

---

## Objective

Design the **High-Level Architecture (HLD)** of a URL Shortener similar to **Bitly**. The system converts long URLs into short URLs and redirects users to the original URL while ensuring scalability, reliability, and high performance.

---

## Functional Requirements

- Convert long URL to short URL
- Redirect users to the original URL
- Store URL mapping
- Support URL expiration (TTL)
- Generate unique short URLs
- Track analytics and click count

---

## Non-Functional Requirements

- High Availability
- Low Latency (<100 ms)
- Scalability (Millions of URLs)
- Fault Tolerance
- Durability (No Data Loss)
- Rate Limiting
- Efficient Caching

---

## System Design Diagram

![HLD — URL Shortener](./url-shortener-hld.png)

---

## Architecture Overview

The system consists of the following major components:

- **API Gateway** – Handles incoming requests and rate limiting.
- **URL Creation Service** – Generates unique short URLs using Base62 encoding.
- **Redirect Service** – Redirects users to the original URL.
- **Redis Cache** – Stores frequently accessed URLs for faster lookup.
- **Database** – Stores URL mappings with replication for reliability.
- **Kafka** – Publishes redirect events asynchronously.
- **Analytics Service** – Processes click events and stores analytics data.

---

## Request Flow

### URL Creation

```
User
   ↓
API Gateway
   ↓
URL Creation Service
   ↓
Generate ID → Base62 Encode
   ↓
Database
```

### URL Redirection

```
User
   ↓
API Gateway
   ↓
Redirect Service
   ↓
Redis Cache
   ↓
Cache Hit → Redirect

Cache Miss
   ↓
Database
   ↓
Update Cache
   ↓
Redirect
```

---

## Technologies Used

- API Gateway
- Redis
- PostgreSQL / Cassandra
- Kafka
- Docker
- Kubernetes

---

## Conclusion

This High-Level Design demonstrates how a URL Shortener like Bitly can efficiently generate short URLs, provide fast redirection using caching, and collect analytics through asynchronous event processing while remaining scalable and fault tolerant.
