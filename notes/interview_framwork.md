# A Framework for System Design Interviews

## Overview

System design interviews can be intimidating.

**How could you possibly design a large-scale system in just one hour, when real systems take years to evolve?**

> **Key Insight:** The focus is on problem-solving, not delivering a perfect solution. The goal is to demonstrate your design skills, defend your choices, and respond constructively to feedback.

---

## What Is the Goal of a System Design Interview?

System design interviews evaluate more than just technical design skills:

✅ **Technical Skills:**
- System architecture knowledge
- Trade-off analysis
- Scalability understanding

✅ **Soft Skills:**
- Collaboration ability
- Handling pressure
- Resolving ambiguity
- Asking thoughtful questions

❌ **Red Flags to Avoid:**
- Over-engineering
- Narrow thinking
- Stubbornness
- Poor communication

---

## A Four-Step Process for Effective System Design Interviews

While every interview is unique, these key steps apply to almost all system design interviews.

---

## Step 1 – Understand the Problem and Establish Design Scope

⏰ **Time Allocation:** 3-10 minutes

### Key Principles

🚫 **DO NOT rush into solutions before fully understanding the problem.**

- Jumping to answers without clarifying requirements is a red flag
- Ask questions to clarify requirements and assumptions instead of making guesses
- Engineers often want to tackle hard problems immediately, but that can lead to designing the wrong system
- **Write down answers and assumptions** to keep track

### What Kind of Questions Should You Ask?

#### Functional Requirements
- What features need to be designed?
- What are the most important features?
- What user actions should the system support?

#### Non-Functional Requirements
- How many users will the product support?
- What is the expected growth timeline (3, 6, 12 months)?
- What is the traffic volume? (DAU, QPS, etc.)

#### Technical Constraints
- What is the existing tech stack?
- What services can be leveraged?
- Are there any specific requirements (mobile, web, both)?

#### Business Logic
- How should data be sorted/ranked?
- What are the edge cases?
- What are the limits? (max friends, max file size, etc.)

### Example: Designing a News Feed

| Question | Answer |
|----------|--------|
| Is this for mobile, web, or both? | Both |
| What are the most important features? | Ability to post and view friends' feeds |
| How is the feed sorted? Chronologically or by relevance? | Chronological |
| What is the maximum number of friends? | 5,000 |
| What is the traffic volume? | 10 million DAU |
| Can posts include media (images, videos)? | Yes, including images and videos |

---

## Step 2 – Propose High-Level Design and Get Buy-In

⏰ **Time Allocation:** 10-15 minutes

### Objectives

In this step, **collaborate with the interviewer** to create a high-level system design.

### What to Do

1. **Sketch a Blueprint**
   - Draw core components: clients, APIs, servers, databases, cache, CDN, message queues, etc.
   - Keep it simple and understandable
   - Ask for feedback continuously

2. **Perform Back-of-the-Envelope Calculations**
   - Quick calculations to check scalability
   - Confirm with the interviewer if this is needed

3. **Discuss Use Cases**
   - Walk through key user flows
   - Refine the design
   - Uncover edge cases

4. **API & Database Design** *(if needed)*
   - Including API schema or database design depends on the problem scale
   - Clarify with the interviewer if this level of detail is required

### Example: News Feed System

#### High-Level Features

**1. Feed Publishing**
- A user creates a post
- Post is stored in the database
- Post is distributed to friends' feeds

**2. Feed Building**
- Aggregate friends' posts in chronological order
- Display in user's feed

#### Component Diagram
```
┌─────────────┐
│   Client    │
│ (Web/Mobile)│
└──────┬──────┘
       │
       ├──────────────┬──────────────┐
       │              │              │
┌──────▼─────┐  ┌─────▼─────┐  ┌────▼────┐
│ Post API   │  │ Feed API  │  │ CDN     │
└──────┬─────┘  └─────┬─────┘  └─────────┘
       │              │
       ▼              ▼
┌─────────────────────────────┐
│   Application Servers       │
└──────┬──────────────┬───────┘
       │              │
┌──────▼──────┐  ┌────▼────────┐
│  Post DB    │  │  Cache      │
│  (MySQL)    │  │  (Redis)    │
└─────────────┘  └─────────────┘
       │
       ▼
┌─────────────┐
│ Message     │
│ Queue       │
└─────────────┘
```

#### Key Flows

**Feed Publishing Flow:**
```
User → Post API → App Server → Post DB → Message Queue → Feed Building Service
```

**News Feed Building Flow:**
```
User → Feed API → Cache (check) → DB (if cache miss) → Aggregate → Return Feed
```

---

## Step 3 – Design Deep Dive

⏰ **Time Allocation:** 10-25 minutes

### Prerequisites

By this stage, you should have:

✅ Agreed on goals and features  
✅ Sketched a high-level blueprint  
✅ Received feedback on your design  
✅ Identified key components for deep dive

### Approach

**Work with the interviewer to decide which components to explore in detail.**

### Possible Focus Areas

#### Performance & Scalability
- How to handle high traffic?
- Caching strategies
- Database sharding
- Load balancing

#### Reliability & Availability
- Replication strategies
- Failover mechanisms
- Data consistency

#### Critical Design Trade-offs
- CAP theorem considerations
- Push vs Pull model
- SQL vs NoSQL
- Synchronous vs Asynchronous processing

#### Detailed Component Design
- Algorithm choices
- Data structures
- API design
- Schema design

### Domain-Specific Examples

| System Type | Deep Dive Focus |
|-------------|-----------------|
| **URL Shortener** | Hashing strategy for generating short links |
| **Chat System** | Minimizing latency, tracking online/offline status |
| **News Feed** | Fanout strategy (push vs pull), ranking algorithm |
| **Video Streaming** | CDN strategy, video encoding, adaptive bitrate |
| **Rate Limiter** | Algorithm choice (token bucket, leaky bucket) |

### Time Management

⚠️ **Manage your time wisely. Do not dive into unnecessary details.**

- Focus on what matters most
- Ask the interviewer what they want to explore
- Be prepared to switch topics if needed

### Example Deep Dive: News Feed

#### Feed Publishing Deep Dive

**Challenge:** How to efficiently distribute posts to thousands of friends?

**Solutions:**

**1. Fanout on Write (Push Model)**
```
User posts → Immediately write to all friends' feed cache
✅ Pros: Fast read (pre-computed)
❌ Cons: Slow write, hotkey problem (celebrity users)
```

**2. Fanout on Read (Pull Model)**
```
User posts → Store in user's timeline
Friend reads → Fetch from all friends' timelines
✅ Pros: Fast write
❌ Cons: Slow read (fetch from many users)
```

**3. Hybrid Approach** *(Recommended)*
```
- Use fanout-on-write for normal users
- Use fanout-on-read for celebrities
- Combine results at read time
```

#### News Feed Building Deep Dive

**Components:**

1. **Feed Cache**
   - Store pre-computed feeds
   - Use Redis for fast access
   - TTL: 5 minutes

2. **Feed Ranking**
   - Initially chronological
   - Later: ML-based ranking
   - Consider: recency, engagement, relationships

3. **Feed Pagination**
   - Cursor-based pagination
   - Load more on scroll
   - Prefetch next page

**Database Schema:**
```sql
-- Posts table
CREATE TABLE posts (
    id BIGINT PRIMARY KEY,
    user_id BIGINT,
    content TEXT,
    created_at TIMESTAMP,
    media_urls JSON
);

-- Friendships table
CREATE TABLE friendships (
    user_id BIGINT,
    friend_id BIGINT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, friend_id)
);

-- Feed cache (Redis)
Key: feed:{user_id}
Value: List of post_ids (sorted by timestamp)
```

---

## Step 4 – Wrap-Up

⏰ **Time Allocation:** 3-5 minutes

At the end, the interviewer may ask follow-up questions or allow open discussion.

### Discussion Topics

#### System Improvements
- Identify bottlenecks and discuss improvements
- Recap your design decisions
- Propose refinements

#### Failure Scenarios
- What if a server crashes?
- How to handle network partitions?
- Database failure scenarios
- Data center outage

#### Operational Aspects
- Monitoring and alerting strategies
- Logging and debugging
- Deployment and rollout strategies
- A/B testing framework

#### Scaling Considerations
- How to scale from 1M to 10M users?
- How to scale from 10M to 100M users?
- Geographic distribution
- Multi-region deployment

#### Future Enhancements
- New features to add
- Technology upgrades
- Performance optimizations

---

## Do's and Don'ts

### ✅ Do's

| Do | Why |
|----|-----|
| **Ask for clarification** | Shows you don't make blind assumptions |
| **Understand requirements thoroughly** | Prevents designing the wrong system |
| **Recognize no single perfect answer** | Demonstrates maturity and flexibility |
| **Share your thought process openly** | Allows interviewer to guide you |
| **Suggest multiple approaches** | Shows depth of knowledge |
| **Agree on blueprint before detailing** | Ensures alignment |
| **Treat interviewer as collaborator** | Creates better interview experience |
| **Write down assumptions** | Keeps track of constraints |
| **Use diagrams** | Makes communication clearer |
| **Discuss trade-offs** | Shows critical thinking |

### ❌ Don'ts

| Don't | Why |
|-------|-----|
| **Come unprepared** | Common questions should be practiced |
| **Jump into solutions** | Need to understand problem first |
| **Spend too long on one component** | Time management is crucial |
| **Be afraid to ask for hints** | Interviewer is there to help |
| **Think silently** | Interviewer can't evaluate your thinking |
| **Over-engineer** | Keep it simple and practical |
| **Be stubborn** | Need to adapt based on feedback |
| **Ignore constraints** | Real systems have real limitations |

---

## Time Allocation Guide

### For a 45-Minute Interview

| Step | Time | Percentage |
|------|------|------------|
| **Step 1:** Understand problem & establish scope | 3-10 min | 15-20% |
| **Step 2:** High-level design & get buy-in | 10-15 min | 25-30% |
| **Step 3:** Design deep dive | 10-25 min | 40-50% |
| **Step 4:** Wrap-up | 3-5 min | 10% |

### Time Management Tips

⏰ **A 45-minute interview is never enough to fully cover a design. Use your time wisely.**

- Keep Step 1 concise but thorough (don't rush, but don't over-discuss)
- Spend most time on Steps 2 and 3
- Leave buffer time for wrap-up
- If running short on time, ask interviewer what to prioritize
- Practice with a timer to get comfortable with pacing

---

## Common System Design Interview Questions

Practice these common scenarios:

### Data Storage & Retrieval
- Design a URL shortener (bit.ly)
- Design a key-value store
- Design a distributed cache
- Design a file storage system (Dropbox/Google Drive)

### Social Networks
- Design a news feed (Facebook/Twitter)
- Design a social network (LinkedIn)
- Design Instagram
- Design a notification system

### Communication
- Design a chat system (WhatsApp/Slack)
- Design a video conferencing system (Zoom)
- Design an email service

### Content Delivery
- Design YouTube/Netflix
- Design a CDN
- Design a web crawler
- Design a search engine

### E-Commerce & Transactions
- Design an e-commerce platform (Amazon)
- Design a payment system
- Design a ride-sharing service (Uber)
- Design a food delivery system

### Utilities
- Design a rate limiter
- Design a load balancer
- Design a distributed message queue
- Design a metrics/analytics system

---

## Key Concepts to Master

### Scalability
- Horizontal vs Vertical scaling
- Load balancing strategies
- Database sharding
- Caching strategies
- CDN usage

### Reliability
- Replication
- Failover mechanisms
- Data consistency models
- Backup strategies

### Performance
- Latency optimization
- Throughput optimization
- Indexing strategies
- Query optimization

### Architecture Patterns
- Microservices vs Monolith
- Event-driven architecture
- CQRS (Command Query Responsibility Segregation)
- Saga pattern

### Data Storage
- SQL vs NoSQL
- CAP theorem
- ACID vs BASE
- Database types (Document, Graph, Column-family, Time-series)

### Communication
- REST vs gRPC
- WebSockets
- Message queues
- Pub/Sub patterns

---

## Interview Success Checklist

Before your interview, make sure you can:

- [ ] Estimate capacity and storage requirements
- [ ] Design APIs for common operations
- [ ] Choose appropriate databases for use cases
- [ ] Implement caching strategies
- [ ] Explain trade-offs between different approaches
- [ ] Draw clear system diagrams
- [ ] Discuss monitoring and observability
- [ ] Handle failure scenarios
- [ ] Scale systems from 1K to 1B users
- [ ] Communicate clearly and collaborate effectively

---

## Resources for Practice

### Books
- "Designing Data-Intensive Applications" by Martin Kleppmann
- "System Design Interview" by Alex Xu (Volume 1 & 2)
- "Web Scalability for Startup Engineers" by Artur Ejsmont

### Online Resources
- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- [Grokking the System Design Interview](https://www.educative.io/courses/grokking-the-system-design-interview)
- [ByteByteGo](https://bytebytego.com/)
- [High Scalability Blog](http://highscalability.com/)

### Practice Platforms
- Mock interviews with peers
- LeetCode System Design section
- Pramp (free mock interviews)
- Exponent

---

## Final Tips

1. **Practice, Practice, Practice**
   - Do at least 10-15 mock interviews
   - Record yourself and review
   - Get feedback from experienced engineers

2. **Learn from Real Systems**
   - Read engineering blogs (Netflix, Uber, Facebook)
   - Understand how real companies solved problems
   - Study architecture decision records (ADRs)

3. **Stay Current**
   - Follow system design trends
   - Learn new technologies
   - Understand cloud services (AWS, GCP, Azure)

4. **Focus on Communication**
   - Practice explaining complex concepts simply
   - Use analogies and examples
   - Check for understanding frequently

5. **Be Honest**
   - If you don't know something, say so
   - Show willingness to learn
   - Ask thoughtful questions

---

**Remember:** System design interviews are conversations, not exams. The interviewer wants you to succeed. Work with them, not against them.

**Good luck! 🚀**

