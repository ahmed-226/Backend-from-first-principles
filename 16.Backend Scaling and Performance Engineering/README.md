# Backend Scaling and Performance Engineering - Part 1

## Table of Contents
1. [Introduction](#introduction)
2. [Defining Performance](#defining-performance)
3. [Latency: The Core Metric](#latency-the-core-metric)
4. [Throughput: Request Processing Capacity](#throughput-request-processing-capacity)
5. [Utilization: System Capacity Usage](#utilization-system-capacity-usage)
6. [The Utilization-Latency Relationship](#the-utilization-latency-relationship)
7. [Identifying Bottlenecks](#identifying-bottlenecks)
8. [Database Performance Optimization](#database-performance-optimization)
9. [N+1 Query Problem](#n1-query-problem)
10. [Database Indexing](#database-indexing)
11. [Connection Pooling](#connection-pooling)
12. [Caching Fundamentals](#caching-fundamentals)
13. [Key Takeaways](#key-takeaways)

---

## Introduction

### Scope and Focus

Scaling and performance are critical concepts in backend engineering, but the definitions vary significantly across different domains:

- **Browser/Frontend Performance**: DOM rendering, JavaScript execution, asset loading
- **Network Performance**: Bandwidth, latency, packet loss, compression
- **Server/Infrastructure Performance**: Operating system, resource allocation, networking
- **Backend Application Performance**: Application code, data handling, business logic efficiency

This guide focuses specifically on **backend application performance** and the principles that help you understand system behavior under load.

### Learning Goals

By the end of this part, you will:
- Understand fundamental performance metrics (latency, throughput, utilization)
- Develop intuition for how systems behave under load
- Identify where bottlenecks hide in your architecture
- Learn practical optimization techniques for databases
- Build a mental model applicable to any system

**Important**: This guide teaches you how to think about performance, not just techniques to memorize.

---

## Defining Performance

### What Does "System is Fast" Mean?

When we say a system is fast, we're referring to a complete user journey:

```mermaid
sequenceDiagram
    participant USER as User/Browser
    participant FE as Frontend
    participant NET as Internet
    participant BE as Backend Server
    participant DB as Database
    
    USER->>FE: Click Button
    FE->>NET: Send HTTP Request
    NET->>BE: Transmit Request
    BE->>DB: Query Database
    DB->>BE: Return Results
    BE->>NET: Send JSON Response
    NET->>FE: Transmit Response
    FE->>USER: Render Content on Screen
```

**Performance** = The total time from when a user initiates an action until they see the result.

### The User's Perspective

Users don't care about:
- How many milliseconds the database query takes
- How optimized your code is
- How many servers you have

Users only care about:
- How long until something appears on their screen
- Whether the app feels responsive
- Whether they can complete their task

---

## Latency: The Core Metric

### Definition

**Latency** is the time elapsed from when a request begins until the response completes. It's what users **feel** when they say "your app is slow."

### Key Insight: Latency Varies

Latency is **not** a single number. It varies from request to request:

```
Request 1: 50 milliseconds (cache hit)
Request 2: 200 milliseconds (database query)
Request 3: 85 milliseconds (partially cached)
Request 4: 300 milliseconds (complex computation)
```

### Why Does Latency Vary?

Several factors cause variation:

```mermaid
graph TB
    subgraph "Factors Affecting Latency"
        A["Cache Hits<br/>Data already available<br/>→ Fast"]
        B["Server Load<br/>CPU busy processing<br/>→ Slow"]
        C["Network Conditions<br/>Packet loss, congestion<br/>→ Variable"]
        D["Data Size<br/>More data to process<br/>→ Slower"]
        E["Query Complexity<br/>Simple vs complex DB ops<br/>→ Variable"]
    end
    
    style A fill:#90EE90
    style B fill:#FFD93D
    style C fill:#FFD93D
    style D fill:#FFD93D
    style E fill:#FFD93D
```

### Real-World Variations

**Best Case Scenario:**
- Data in cache
- Server idle
- Optimal network
- Small response
- Result: ~50ms

**Worst Case Scenario:**
- Cache miss
- Server fully loaded
- Poor network
- Large response
- Complex query
- Result: ~2000ms (2 seconds)

### Measuring Latency Properly

Instead of reporting a single number, report **distribution metrics**:

```
Minimum:     45ms
P50 (Median): 150ms
P95:          800ms
P99:          1500ms
Maximum:      3000ms
```

**Why P95/P99?** Most users experience these percentiles, and they're more representative of typical experience than just the average.

---

## Throughput: Request Processing Capacity

### Definition

**Throughput** is the number of requests your system can process per unit of time.

```
Throughput Examples:
- 100 requests per second (RPS)
- 1000 queries per minute (QPM)
- 6 million requests per day
```

### Throughput Affects Latency

A critical relationship: As throughput increases, latency increases.

```mermaid
graph TB
    A["Low Throughput<br/>Few requests<br/>Low latency<br/>Quick responses"]
    B["Moderate Throughput<br/>Steady load<br/>Acceptable latency<br/>Manageable queues"]
    C["High Throughput<br/>Many requests<br/>Higher latency<br/>Longer queues"]
    D["Overload Condition<br/>Requests exceeding<br/>system capacity<br/>System degradation"]
    
    A --> B
    B --> C
    C --> D
    
    style A fill:#90EE90
    style B fill:#FFD93D
    style C fill:#FFA07A
    style D fill:#FF6B6B
```

### Real-World Scenarios

**Can our system handle:**
- Black Friday traffic spikes?
- Email campaign surge?
- Featured in a popular podcast?
- Viral social media post?

All these questions require understanding throughput and latency together.

### The Queue Analogy

Think of your system like a service counter:

```
Low Traffic (Throughput):
Server is idle → Request arrives → Served immediately
                 Latency: ~50ms

High Traffic:
Server is busy → Request arrives → Joins queue → Waits → Served
                 Latency: ~500ms (service + wait time)
```

The **worker's speed doesn't change**, but your wait time increases because of the queue.

---

## Utilization: System Capacity Usage

### Definition

**Utilization** is the percentage of your system's capacity currently in use.

```
0% utilization   = System idle, no requests being processed
50% utilization  = Half capacity being used
100% utilization = Fully saturated, at maximum capacity
```

### The Utilization-Latency Curve

This is the **most important relationship** in performance engineering:

```mermaid
graph TB
    A["0-60% Utilization<br/>Linear growth<br/>Acceptable latency<br/>Predictable behavior"]
    B["60-80% Utilization<br/>Curves upward<br/>Latency increases<br/>Queue forming"]
    C["80-95% Utilization<br/>Sharp increase<br/>High latency<br/>Queue growing fast"]
    D["95-100% Utilization<br/>Exponential growth<br/>Severe latency<br/>System struggling"]
    E[">100% Utilization<br/>Queue overflow<br/>Request rejection<br/>System collapse"]
    
    A --> B
    B --> C
    C --> D
    D --> E
    
    style A fill:#90EE90
    style B fill:#FFD93D
    style C fill:#FFA07A
    style D fill:#FF6B6B
    style E fill:#8B0000
```

### The Counterintuitive Truth

Most people expect latency to grow linearly with utilization:

```
Expected (Linear):  Latency = Utilization × Constant
Actual (Exponential): Latency grows exponentially near 100%
```

**Why the exponential curve?**

At high utilization:
- Requests queue up waiting for resources
- Context switching increases overhead
- Contention for resources intensifies
- Small inefficiencies compound

### Highway Analogy

```
50% Capacity:
- Cars can change lanes freely
- Traffic flows smoothly
- Everyone maintains speed
- Predictable travel time

80% Capacity:
- Lane changes are risky
- Need to plan ahead
- Some slowdowns
- Mostly predictable

90% Capacity:
- Barely any space
- One delayed car affects everyone
- Unpredictable jams
- Ripple effects from any disruption

100% Capacity:
- No space to move
- Complete gridlock
- Nothing flows
- Total system failure
```

### The Buffer Requirement

**Critical Realization**: You cannot run systems at 100% utilization.

Production systems typically operate at:
- **60-80% normal utilization** (steady state)
- **20% buffer** (for traffic spikes)

**Why the buffer?**

1. **Traffic comes in bursts**, not smoothly
2. **Spikes can exceed average** by 10-20x
3. **Exponential relationship** means small increases cause large latency jumps

---

## The Utilization-Latency Relationship

### Visual Representation

```mermaid
graph TB
    subgraph "Capacity Management Strategy"
        A["Identify Typical Load<br/>Historical data<br/>Monitoring metrics"]
        B["Determine Peak Bursts<br/>Busiest times<br/>Traffic spikes"]
        C["Calculate Buffer Needed<br/>Peak ÷ Typical<br/>Usually 20-30%"]
        D["Set Scaling Target<br/>Aim for 60-80% utilization<br/>Keep buffer free"]
        E["Configure Autoscaling<br/>Add resources at 70%<br/>Remove at 40%"]
    end
    
    A --> B
    B --> C
    C --> D
    D --> E
    
    style A fill:#87CEEB
    style B fill:#87CEEB
    style C fill:#FFD93D
    style D fill:#90EE90
    style E fill:#90EE90
```

### Example: E-commerce Platform

```
Normal traffic:      1000 RPS (60% capacity)
Peak traffic:        1500 RPS (90% capacity)
Traffic spike:       2500 RPS (150% capacity) ← NEEDS SCALING

System capacity:     ~1667 RPS (100%)
Comfortable running: 1000 RPS (60%)
```

During a spike, the system auto-scales to handle additional load.

---

## Identifying Bottlenecks

### The Reality of Performance Issues

When someone says "the system is slow," it's almost always **one specific component** causing the slowness:

```mermaid
graph TB
    A["System is Slow"]
    
    A --> B{Where's the Bottleneck?}
    
    B -->|CPU| C["CPU-bound operation<br/>Complex computation<br/>Inefficient algorithm<br/>Solution: Optimize code"]
    
    B -->|Memory| D["Memory constraint<br/>Large dataset processing<br/>Memory leak<br/>Solution: Reduce memory usage"]
    
    B -->|Disk I/O| E["Slow disk access<br/>Database bottleneck<br/>File system operations<br/>Solution: Optimize queries"]
    
    B -->|Network| F["Network latency<br/>External API calls<br/>Data transfer size<br/>Solution: Reduce calls/data"]
    
    B -->|Lock Contention| G["Resource locks<br/>Database row locks<br/>Cache conflicts<br/>Solution: Parallel processing"]
    
    style C fill:#FFD93D
    style D fill:#FFD93D
    style E fill:#FFD93D
    style F fill:#FFD93D
    style G fill:#FFD93D
```

### How to Find Bottlenecks

```
1. Measure
   ↓ Collect metrics from all layers (frontend, network, backend, database)
   ↓
2. Analyze
   ↓ Identify which layer has the highest latency
   ↓
3. Focus
   ↓ Optimize the slowest layer first
   ↓
4. Verify
   ↓ Measure again to confirm improvement
   ↓
5. Repeat
   ↓ Move to next bottleneck
```

### The 80/20 Rule

> 80% of performance issues come from 20% of the code.

Focus optimization efforts on:
- Frequently called functions
- Operations affecting multiple requests
- Bottlenecks in critical paths

Don't optimize:
- Code that runs rarely
- Code that's already fast enough
- Code that doesn't affect user experience

---

## Database Performance Optimization

### Why Databases are Often the Bottleneck

Most systems follow this architecture:

```
Frontend → Backend Application → Database
```

Database is the bottleneck because:
- Network latency (request/response)
- Query parsing overhead
- I/O operations (disk access)
- Concurrency management

### Common Database Performance Issues

```mermaid
graph TB
    subgraph "Database Performance Problems"
        A["1. N+1 Query Problem<br/>Making one query<br/>then N more queries<br/>Inefficient loops"]
        B["2. Missing Indexes<br/>Full table scans<br/>Instead of indexed lookups<br/>Slow for large tables"]
        C["3. Poor Query Design<br/>Fetching unneeded columns<br/>Complex joins<br/>Inefficient filters"]
        D["4. Connection Overhead<br/>Creating new connections<br/>for each request<br/>TCP handshake costs"]
        E["5. Lock Contention<br/>Multiple requests<br/>accessing same rows<br/>Waiting for locks"]
    end
    
    style A fill:#FFD93D
    style B fill:#FFD93D
    style C fill:#FFD93D
    style D fill:#FFD93D
    style E fill:#FFD93D
```

---

## N+1 Query Problem

### What is N+1?

The N+1 query problem occurs when you:
1. Make 1 query to fetch N items
2. Then make N more queries to fetch details about each item
3. Total: 1 + N queries (where it should be 1-2 queries)

### Example: Blog Post with Authors

**Inefficient Code (N+1):**

```
Query 1: SELECT * FROM posts WHERE user_id = 123
         Result: [20 posts]

Then in a loop:
Query 2: SELECT * FROM users WHERE id = 1
Query 3: SELECT * FROM users WHERE id = 2
Query 4: SELECT * FROM users WHERE id = 3
...
Query 21: SELECT * FROM users WHERE id = 20

Total Queries: 21
```

**Efficient Code (2 queries):**

```
Query 1: SELECT * FROM posts WHERE user_id = 123
         Result: [20 posts]

Query 2: SELECT * FROM users WHERE id IN (1, 2, 3, ..., 20)
         Result: All authors at once

Total Queries: 2
```

### The Cost of N+1

Each query has overhead:

```
Network latency:        1ms per round trip
TCP connection:         2ms (if pooled, ~0.1ms)
Database parsing:       2ms
Query execution:        5ms
Network return:         1ms
──────────────────────
Per query overhead:     ~11ms minimum

For 1000 items:
1000 queries × 11ms = 11 seconds
vs.
2 queries × 11ms = 22ms
```

**Impact**: 1000× slower performance!

### Preventing N+1

```mermaid
graph TB
    subgraph "Solutions by Framework"
        A["Django ORM<br/>select_related()<br/>prefetch_related()"]
        B["Ruby on Rails<br/>includes()<br/>eager loading"]
        C["TypeScript/Prisma<br/>select()<br/>include()"]
        D["Raw SQL<br/>JOIN<br/>LEFT JOIN<br/>UNION"]
    end
    
    E["Principle:<br/>Fetch related data<br/>in a single query<br/>or bulk query"]
    
    A --> E
    B --> E
    C --> E
    D --> E
    
    style E fill:#90EE90
```

### Best Practice: Enable Query Logging

During development, enable SQL query logging to see what queries your ORM actually executes:

```
[DEBUG] SELECT * FROM posts WHERE user_id = 123
[DEBUG] SELECT * FROM users WHERE id = 1
[DEBUG] SELECT * FROM users WHERE id = 2
[DEBUG] SELECT * FROM users WHERE id = 3
← See the N+1 problem immediately!

vs.

[DEBUG] SELECT * FROM posts WHERE user_id = 123
[DEBUG] SELECT * FROM users WHERE id IN (1, 2, 3, ...)
← Optimized approach
```

---

## Database Indexing

### What is an Index?

An index is a data structure that allows fast lookups without scanning entire tables.

### Library Analogy

**Without Index (Full Table Scan):**

Imagine a library with 1 million books but **no catalog**:
- Customer wants all books by "John Green"
- Librarian must:
  1. Check every single book
  2. Write down locations of John Green books
  3. Walk to all locations and collect books
  4. Time needed: 3 days

**With Index (Using Catalog):**

Same library with a **catalog organized by author**:
- Customer asks for John Green
- Librarian checks catalog
- Catalog shows exact shelf locations
- Librarian walks directly to locations
- Time needed: 2-3 minutes

### How Indexes Work

Indexes typically use **B-Tree** data structure:

```
Index on author_id:
1 → [pointers to rows with author_id=1]
2 → [pointers to rows with author_id=2]
3 → [pointers to rows with author_id=3]
4 → [pointers to rows with author_id=4]
...

To find all posts by author 2:
1. Look up "2" in index (fast, O(log n))
2. Get list of row pointers (instant)
3. Fetch those specific rows (fast)

Without index:
1. Scan entire table (slow, O(n))
2. Check each row (very slow for large tables)
```

### Full Table Scan vs. Indexed Lookup

```mermaid
graph TB
    A["Query: Find all posts by author X"]
    
    A --> B{Is author_id indexed?}
    
    B -->|No| C["Full Table Scan"]
    C --> C1["Read every row<br/>from disk"]
    C1 --> C2["Check if author_id = X"]
    C2 --> C3["Collect matching rows"]
    C3 --> C4["Time: 1-2 seconds<br/>for large table"]
    
    B -->|Yes| D["Index Lookup"]
    D --> D1["Binary search in index<br/>O(log n)"]
    D1 --> D2["Get row pointers"]
    D2 --> D3["Fetch specific rows<br/>from disk"]
    D3 --> D4["Time: 10-50ms<br/>for large table"]
    
    style C4 fill:#FF6B6B
    style D4 fill:#90EE90
```

### When to Create Indexes

Create indexes on columns used in:

```
1. WHERE clauses
   WHERE author_id = X    → Index author_id

2. JOIN conditions
   JOIN users ON posts.user_id = users.id
                           → Index posts.user_id and users.id

3. ORDER BY
   ORDER BY created_at DESC
                           → Index created_at

4. Foreign keys
   Foreign key relationships
                           → Always index foreign keys

Don't index:
- Columns with low cardinality (few unique values)
- Columns rarely used in queries
- Columns with frequent updates (index maintenance overhead)
```

### Index Trade-offs

```mermaid
graph TB
    A["Create Index"]
    
    A --> B["Benefits"]
    B --> B1["Faster SELECT queries<br/>10-100× speedup"]
    B --> B2["Faster WHERE/JOIN<br/>Indexed lookups"]
    B --> B3["Better reporting<br/>ORDER BY, GROUP BY"]
    
    A --> C["Costs"]
    C --> C1["Slower INSERT/UPDATE<br/>Must update index"]
    C --> C2["Slower DELETE<br/>Must update index"]
    C --> C3["Disk space<br/>Index requires storage"]
    C --> C4["Memory usage<br/>Index in memory"]
    
    style B1 fill:#90EE90
    style B2 fill:#90EE90
    style B3 fill:#90EE90
    style C1 fill:#FFD93D
    style C2 fill:#FFD93D
    style C3 fill:#FFD93D
    style C4 fill:#FFD93D
```

**Strategy**: Index columns used in frequent, performance-critical queries.

---

## Connection Pooling

### The Connection Overhead Problem

Creating a new database connection is expensive:

```
1. DNS lookup:              0-5ms
2. TCP connection setup:    5-10ms
3. Authentication:          2-5ms
4. Query execution:         5-50ms (actual work)
   ────────────────
   Total per query:         12-70ms

For 1000 queries:
- With pooling: 1000 × 10ms = 10 seconds
- Without pooling: 1000 × 50ms = 50 seconds (5x slower!)
```

### Connection Pooling Solution

Instead of creating a new connection for each request:

```mermaid
graph TB
    A["Request 1"]
    B["Request 2"]
    C["Request 3"]
    D["Request 4"]
    
    A --> POOL["Connection Pool<br/>Maintains 10-50<br/>open connections<br/>reuses them"]
    B --> POOL
    C --> POOL
    D --> POOL
    
    POOL --> DB["Database<br/>Single set of<br/>connections"]
    
    style POOL fill:#90EE90
```

### Types of Pooling

#### Internal Pooling (Application-level)

```
Server 1 → Connection Pool (10 connections) ↘
Server 2 → Connection Pool (10 connections) → Database (capacity: 30 connections)
Server 3 → Connection Pool (10 connections) ↗

Problem: Multiple servers can exceed database capacity
```

#### External Pooling (Dedicated pooler)

```
Server 1 →┐
Server 2 →├→ External Pooler (25 connections) → Database (capacity: 30 connections)
Server 3 →┘

Benefit: Single pool prevents connection exhaustion
```

### When Pooling Becomes Critical

**Scenario: Traffic Spike with Auto-scaling**

```
Normal state:
- 1 server with 10-connection pool
- Database max: 30 connections
- Utilization: 10/30 = 33% ✓

Traffic spike detected:
- Kubernetes adds 2 more servers
- Now 3 servers × 10-connection pool = 30 connections
- Utilization: 30/30 = 100% (at capacity)

Traffic spikes again:
- Each server tries to grab 20 connections (high load)
- Total needed: 3 × 20 = 60 connections
- Database capacity: 30 connections
- Result: Connection pool exhausted! ✗ Database crash
```

**Solution**: Use external pooler (e.g., PgBouncer for PostgreSQL)

```mermaid
graph TB
    S1["Server 1"]
    S2["Server 2"]
    S3["Server 3"]
    
    S1 --> POOL["External Pooler<br/>Max 25 connections<br/>Manages all requests"]
    S2 --> POOL
    S3 --> POOL
    
    POOL --> DB["Database<br/>30 connections<br/>available"]
    
    A["Benefits:<br/>Single source of truth<br/>Prevents overload<br/>Fair distribution<br/>Connection reuse"]
    
    style POOL fill:#90EE90
    style A fill:#E6F3FF
```

---

## Caching Fundamentals

### The Caching Principle

> Store results of expensive operations, reuse them instead of recomputing.

```mermaid
graph TB
    A["Request comes in"]
    A --> B{Is result<br/>in cache?}
    
    B -->|Yes| C["Return from cache<br/>50ms latency"]
    B -->|No| D["Fetch from database<br/>500ms latency"]
    
    D --> E["Store in cache"]
    E --> F["Return result"]
    
    C --> G["Same result,<br/>10× faster!"]
    F --> G
    
    style C fill:#90EE90
    style D fill:#FFD93D
    style G fill:#90EE90
```

### Caching Impact

```
Without cache:
- All requests query database: 500ms each
- 1000 requests: 500 seconds total

With cache (80% hit rate):
- 800 requests from cache: 50ms each = 40 seconds
- 200 requests from database: 500ms each = 100 seconds
- Total: 140 seconds (71% faster!)
```

### Cache Invalidation

**The Hard Problem**: Keeping cache consistent with database.

```mermaid
graph TB
    A["Data updated<br/>in database"]
    A --> B{Update cache?}
    
    B -->|Yes| C["Cache updated<br/>Extra CPU/latency<br/>But consistent"]
    B -->|No| D["Cache stale<br/>Returns old data<br/>User sees outdated info"]
    
    C --> E["User sees<br/>correct data"]
    D --> F["User sees<br/>outdated data"]
    
    style C fill:#90EE90
    style D fill:#FF6B6B
```

### Cache Invalidation Strategies

```
1. Time-based (TTL):
   Cache expires after 5 minutes
   Example: cache.set(key, value, ttl=300)
   ✓ Simple
   ✗ Data might be stale

2. Event-based:
   Update cache when data changes
   Example: On UPDATE, delete cache entry
   ✓ Consistent
   ✗ Complex to implement

3. Demand-based:
   Application checks and updates
   Example: Check timestamp, refresh if old
   ✓ Flexible
   ✗ Additional complexity

4. Hybrid:
   TTL + event-based + demand checks
   ✓ Most robust
   ✗ Most complex
```

---

## Key Takeaways

### Fundamental Concepts

1. **Performance is Measurable**
   - Latency: time for request to complete
   - Throughput: requests per unit time
   - Utilization: percentage of capacity in use

2. **Utilization Drives Latency Exponentially**
   - Linear from 0-60% utilization
   - Curves upward from 60-80%
   - Exponential from 80-100%
   - Never run at 100% utilization

3. **Bottleneck Mindset**
   - One component usually causes slowness
   - Find it, optimize it, measure improvement
   - Move to next bottleneck

### Database Optimization Priority

```mermaid
graph TB
    A["Database Running Slow?"]
    
    A --> B["1. Fix N+1 Queries<br/>Use joins, bulk fetch<br/>Check query logs"]
    B --> C["2. Add Missing Indexes<br/>Analyze query patterns<br/>Index WHERE/JOIN columns"]
    C --> D["3. Configure Connection Pooling<br/>Use pool for efficiency<br/>External pooler for scaling"]
    D --> E["4. Implement Caching<br/>Cache expensive queries<br/>Handle invalidation"]
    E --> F["Still Slow?"]
    F --> G["Move to next layer<br/>API optimization<br/>Algorithm efficiency"]
    
    style B fill:#FFD93D
    style C fill:#FFD93D
    style D fill:#FFD93D
    style E fill:#FFD93D
```

### Action Items

✓ **Measure everything** - Collect latency, throughput, utilization metrics
✓ **Find the bottleneck** - Use profiling and monitoring tools
✓ **Optimize iteratively** - Fix biggest impact first
✓ **Verify improvements** - Measure before and after
✓ **Monitor production** - Performance changes over time
✓ **Set performance budgets** - Define acceptable latency targets

### Remember

- **Performance is a feature**, not an afterthought
- **Premature optimization is bad**, but informed optimization is essential
- **Understand your system** before optimizing
- **Measure everything** - "You can't improve what you don't measure"
- **Keep buffer capacity** - 20-30% reserved for spikes

---

## Next Steps

This Part 1 covered foundational concepts and database optimization. Part 2 typically covers:
- Scaling strategies (vertical vs. horizontal)
- Load balancing
- Caching strategies in depth
- API optimization
- Message queues and async processing
- Infrastructure scaling

The mental models learned here apply to all these topics.
