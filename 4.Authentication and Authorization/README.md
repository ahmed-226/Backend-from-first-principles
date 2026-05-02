# Authentication and Authorization: A Complete Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Historical Context of Authentication](#historical-context-of-authentication)
3. [Core Concepts](#core-concepts)
4. [Authentication Components](#authentication-components)
5. [Sessions](#sessions)
6. [JWT (JSON Web Tokens)](#jwt-json-web-tokens)
7. [Cookies](#cookies)
8. [Authentication Types](#authentication-types)
9. [API Key Based Authentication](#api-key-based-authentication)
10. [OAuth 2.0 and OpenID Connect](#oauth-20-and-openid-connect)
11. [Security Best Practices](#security-best-practices)

---

## Introduction

### Two Fundamental Concepts

**Authentication** = WHO are you?
- Mechanism to assign an identity to a subject
- Answers: "Who are you in a given context?"
- Examples: Login screens, identity verification

**Authorization** = WHAT can you do?
- Answers: "What can you do in this context?"
- Your capabilities and permissions
- Examples: Admin access, user roles, resource permissions

### Quick Summary
> **Authentication** identifies you. **Authorization** determines what you can access.

---

## Historical Context of Authentication

### Pre-Industrial Era (Implicit Trust)

```mermaid
graph TD
    A[Pre-Industrial Society] --> B[Implicit Authentication]
    B --> C[Personal Recognition]
    C --> D[Village Elder vouches for person]
    D --> E[Handshake - Mutual recognition]
    E --> F[Seals on documents/agreements]
    
```

**Characteristics:**
- Based on human contextual trust
- Identity = Personal recognition
- Village Elders vouched for community members
- Wax seals used as signatures on documents
- **Limitation**: Cannot scale beyond familiar circles

### Medieval Period (Early Cryptography)

```mermaid
graph LR
    A[Medieval Period] --> B[Wax Seals]
    B --> C[Unique Patterns]
    C --> D[Physical Identity Token]
    D --> E[Possession-based Authentication]
    
    E --> F[Vulnerabilities]
    F --> G[Forgery - First bypass attacks]
    
```

**Innovations:**
- Wax seals = First widely adopted authentication tokens
- Unique patterns as cryptographic mechanism
- Watermarks and encrypted codes emerged
- **First recorded bypass attacks** (forgery)

### Industrial Revolution (Shared Secrets)

```mermaid
graph TD
    A[Industrial Revolution] --> B[Telegraph Technology]
    B --> C[Pre-agreed Pass Phrases]
    C --> D[Static Passwords]
    D --> E[Principle: Something you know]
    
```

**Advancements:**
- Telegraph operators used pass phrases
- Early form of shared secrets
- Principle shifted: "Something you know" vs "Something you possess" (seal)

### Mid-20th Century (Digital Phase)

```mermaid
graph LR
    A[1961 - MIT Project MAC] --> B[Multi-user Systems]
    B --> C[Passwords for authentication]
    C --> D[Stored in plaintext - VULNERABLE!]
    D --> E[1970s - Hashing invented]
    E --> F[CRYPTO: Confidentiality, Integrity, Availability]
    
```

**Key Events:**
- **1961**: MIT researchers introduced passwords for CTSS (Compatible Time-Sharing System)
- **Critical Incident**: Password file printed in plaintext → Led to secure storage philosophy
- **1970s**: Hashing algorithms emerged (Diffie-Hellman key exchange)
- **Core tenets**: Confidentiality, Integrity, Availability

### 1990s-2000s (Modern Authentication)

```mermaid
graph TD
    A[1990s - Internet Growth] --> B[Username/Password not enough]
    B --> C[Brute force & dictionary attacks]
    C --> D[Multi-Factor Authentication MFA]
    D --> E[Something you know + have + are]
    
    E --> F[2000s - Distributed Systems]
    F --> G[JWT - Stateless tokens]
    G --> H[2010s - OAuth 2.0]
    
```

**Evolution:**
- **MFA**: Combination of passwords + smart cards + biometrics
- **Challenges**: Biometric systems (false positives/negatives, template security)
- **2000s**: JWTs emerged for globally distributed systems
- **2010s**: OAuth 2.0, OpenID Connect, Zero Trust, Passwordless auth

### 21st Century (Current & Future)

```mermaid
graph LR
    A[21st Century] --> B[Cloud Computing]
    A --> C[Mobile Devices]
    A --> D[API-based Architectures]
    
    B --> E[Advanced Frameworks Needed]
    C --> E
    D --> E
    
    E --> F[OAuth 2.0 / JWT]
    E --> G[Decentralized Identity - Blockchain]
    E --> H[Post-Quantum Cryptography]
    E --> I[Behavioral Biometrics]
    
```

---

## Core Concepts

### Three Key Components of Modern Authentication

| Component | Description | Use Case |
|-----------|-------------|----------|
| **Sessions** | Server-side state storage | Traditional web apps |
| **JWTs** | Self-contained tokens | Distributed systems, APIs |
| **Cookies** | Browser storage mechanism | Web authentication |

---

## Authentication Components

### 1. Sessions

#### What is a Session?

A session provides a way to establish **temporary server-side context** for each user, giving the web "memory" despite HTTP being stateless.

#### How Sessions Work

```mermaid
sequenceDiagram
    participant Client as Browser Client
    participant Server as Server
    participant Store as Redis/Database
    
    Note over Client,Server: Step 1: User Login
    Client->>Server: POST /login (username, password)
    Server->>Server: Validate credentials
    Server->>Store: Create Session ID + User Data
    Store-->>Server: Session stored (session_abc123)
    Server->>Client: Set-Cookie: session_id=abc123
    
    Note over Client,Server: Step 2: Subsequent Requests
    Client->>Server: GET /api/data (Cookie: session_id=abc123)
    Server->>Store: Lookup session_abc123
    Store-->>Server: User data found, valid
    Server->>Client: Return requested data
    
    Note over Client,Server: Step 3: Session Expiry
    Client->>Server: GET /api/data (Cookie: session_id=abc123)
    Server->>Store: Lookup session_abc123
    Store-->>Server: Session expired!
    Server->>Client: 401 Unauthorized - Please login again
```

#### Session Storage Evolution

```mermaid
graph TD
    A[Early Sessions] --> B[File-based Storage]
    B --> C[Not scalable - many users]
    C --> D[Database Storage]
    D --> E[Faster lookups]
    E --> F[Distributed Architecture]
    F --> G[Redis / Memcached]
    G --> H[In-memory - Very fast]
    
```

**Storage Types:**
1. **File-based** (Early): Simple but not scalable
2. **Database**: Persistent, survives server restarts
3. **Redis/Memcached** (Modern): In-memory, very fast read times

#### Pros & Cons of Session-Based Auth

| ✅ Pros | ❌ Cons |
|---------|---------|
| Centralized control over sessions | Scalability issues with millions of users |
| Real-time session information | Replication challenges across regions |
| Easy to revoke access | Latency in distributed systems |
| Stateful - remembers user | Higher operational complexity |
| Well-suited for traditional web apps | |

---

### 2. JWT (JSON Web Tokens)

#### What is JWT?

JWT is a **self-contained token** that contains user data and cryptographic signatures in a stateless manner.

#### JWT Structure

```mermaid
graph TD
    A[JWT Token - 3 Parts] --> B[1. Header]
    A --> C[2. Payload]
    A --> D[3. Signature]
    
    B --> B1["alg: HS256<br/>type: JWT"]
    C --> C1["sub: 123456<br/>name: John<br/>iat: 1516239022<br/>role: admin"]
    D --> D1["HMACSHA256(<br/>base64UrlEncode(header) + '.' + base64UrlEncode(payload),<br/>secret)"]    
```

**JWT Format:** `base64(header).base64(payload).signature`

#### JWT Workflow

```mermaid
sequenceDiagram
    participant Client as Browser Client
    participant Server as Server
    
    Note over Client,Server: Step 1: User Login
    Client->>Server: POST /login (username, password)
    Server->>Server: Validate credentials
    Server->>Server: Sign JWT with secret key
    Server->>Server: Create token with user ID, role, etc.
    Server->>Client: Return JWT token
    
    Note over Client,Server: Step 2: Subsequent Requests
    Client->>Server: GET /api/data (Authorization: Bearer eyJhbGci...)
    Server->>Server: Verify JWT signature with secret key
    Server->>Server: Extract user ID from payload
    Server->>Client: Return requested data
    
    Note over Client,Server: Step 3: Token Expiry
    Client->>Server: GET /api/data (Authorization: Bearer eyJhbGci...)
    Server->>Server: JWT expired!
    Server->>Client: 401 Unauthorized
```

#### JWT Stateless Advantage

```mermaid
graph LR
    A[Client] --> B[Server 1 - US]
    A --> C[Server 2 - Europe]
    A --> D[Server 3 - Asia]
    
    B --> E[Shared Secret Key]
    C --> E
    D --> E
    
    E --> F[Any server can verify JWT!]
    
```

**Key Benefit:** No session store needed - any server can verify the token using the shared secret key.

#### Pros & Cons of JWT

| ✅ Pros | ❌ Cons |
|---------|---------|
| **Stateless** - No server-side storage | **Token revocation is complex** |
| **Scalable** - Works across distributed systems | No way to track token status |
| **Portable** - URL-friendly base64 format | Once issued, valid until expiry |
| **Lightweight** - Every request is independent | Compromised token = attacker can impersonate |
| Ideal for microservices & mobile apps | Changing secret invalidates ALL tokens |

#### Hybrid Approach (Best of Both Worlds)

```mermaid
graph TD
    A[Client Request with JWT] --> B{Verify JWT}
    B -->|Valid| C[Check Blacklist in Redis/DB]
    C -->|Not blacklisted| D[Process Request]
    C -->|Blacklisted| E[401 Unauthorized]
    
    B -->|Invalid| F[401 Unauthorized]
    
```

**How it works:**
1. Verify JWT signature (stateless)
2. Check token against blacklist (stateful)
3. If blacklisted → Reject (revocation capability!)
4. Process the request

---

### 3. Cookies

#### What is a Cookie?

A cookie is a small piece of data stored in the user's browser by the server.

#### Cookie Workflow

```mermaid
sequenceDiagram
    participant Client as Browser Chrome/Firefox
    participant Server as Server
    
    Note over Client,Server: Authentication Flow
    Client->>Server: POST /login (username, password)
    Server->>Server: Validate credentials
    Server-->>Client: Set-Cookie: session_id=abc123 (HttpOnly)
    
    Note over Client: Cookie stored (JavaScript can't access if HttpOnly)
    
    Client->>Server: GET /api/data (Cookie: session_id=abc123)
    Server->>Server: Extract cookie
    Server->>Server: Validate session/token
    Server->>Client: Return data
    
    Note over Client,Server: Every subsequent request includes the cookie!
```

**Key Features:**
- **Automatic**: Browser sends cookie with every request to that server
- **HttpOnly**: JavaScript cannot access (prevents XSS attacks)
- **Domain-specific**: One server cannot see another server's cookies
- **Secure flag**: Only sent over HTTPS

---

## Authentication Types

### 1. Stateful Authentication

```mermaid
sequenceDiagram
    participant Client as Browser
    participant Server as Server
    participant Redis as Redis Store
    
    Client->>Server: Login (username, password)
    Server->>Redis: Store session_abc + user data
    Server-->>Client: Set-Cookie: session_id=abc
    
    Client->>Server: GET /api/data (Cookie: session_id=abc)
    Server->>Redis: Lookup session_abc
    Redis-->>Server: User data (valid)
    Server-->>Client: Data response
```

**Characteristics:**
- Server remembers user (stateful)
- Session ID stored in cookie
- User data stored server-side (Redis/DB)
- Can revoke access easily
- Centralized control

### 2. Stateless Authentication

```mermaid
sequenceDiagram
    participant Client as Browser
    participant Server as Server
    
    Client->>Server: Login (username, password)
    Server->>Server: Sign JWT with secret
    Server-->>Client: Return JWT token
    
    Client->>Server: GET /api/data (Authorization: Bearer xyz)
    Server->>Server: Verify JWT signature
    Server-->>Client: Data response (no DB lookup needed!)
```

**Characteristics:**
- No server-side storage
- JWT contains all user info
- Scalable across multiple servers
- No session store dependency
- Token revocation is harder

### When to Use Which?

```mermaid
graph TD
    A[Choose Authentication Type] --> B{Application Type?}
    
    B -->|Traditional Web App| C[Stateful - Sessions]
    B -->|Microservices/APIs| D[Stateless - JWT]
    B -->|Mobile App| D
    B -->|Third-party Integration| D
    
    C --> E[Pros: Easy revocation, real-time control]
    D --> F[Pros: Scalable, no session store]
    
```

**Recommendation:**
- **Stateful**: Traditional web apps, strict session requirements
- **Stateless**: Distributed systems, mobile apps, third-party integrations
- **Hybrid**: Web apps use sessions, APIs use JWT

---

## API Key Based Authentication

### What is API Key Auth?

API keys are **cryptographically random strings** generated by a platform for programmatic access.

### How API Keys Work

```mermaid
sequenceDiagram
    participant User as User
    participant Platform as Platform UI
    participant Server as API Server
    
    Note over User,Platform: Step 1: Generate API Key
    User->>Platform: Click "Generate API Key"
    Platform-->>User: Return: "sk_live_abc123xyz789"
    
    Note over User,Server: Step 2: Use API Key
    User->>Server: GET /api/data (Authorization: Bearer sk_live_abc123xyz789)
    Server->>Server: Validate API key
    Server-->>User: Data response
    
    Note over User: Use Case: ChatGPT API, programmatic access
```

**Use Cases:**
- **ChatGPT API**: Programmatic access to LLM models
- **Server-to-server** communication
- **Machine-to-machine** authentication (no UI/human interaction)
- **Third-party integrations**

**Characteristics:**
- Easy to generate
- Ideal for machine-to-machine communication
- Typically has permissions and expiry dates
- Can be revoked when compromised

---

## OAuth 2.0 and OpenID Connect

### The Delegation Problem

```mermaid
graph TD
    A[User] --> B[App A - Travel Booking]
    A --> C[App B - Gmail]
    A --> D[App C - Social Media]
    
    B -->|Needs access to| E[Gmail Contacts]
    D -->|Needs access to| E
    
    E --> F[Problem: Sharing passwords is INSECURE!]
    F --> G[OAuth 2.0 Solution: Share TOKENS instead]
    
```

**The Problem:** Apps need access to other platforms' resources, but sharing passwords is:
- **Insecure**: Gives full access to everything
- **Impossible to revoke** without changing password everywhere

### OAuth 2.0 Flow

```mermaid
sequenceDiagram
    participant User as Resource Owner (You)
    participant Client as Client App (Facebook)
    participant AuthServer as Authorization Server (Google)
    participant ResourceServer as Resource Server (Google Contacts)
    
    Note over User,Client: Step 1: Request Access
    User->>Client: "I want to import Google contacts"
    Client->>AuthServer: Redirect user to authorize
    
    Note over User,AuthServer: Step 2: User Authenticates
    User->>AuthServer: Login to Google + Grant Permissions
    AuthServer-->>User: Authorization Code
    
    User->>Client: Return with Authorization Code
    Client->>AuthServer: Exchange code for Access Token
    AuthServer-->>Client: Access Token + ID Token (JWT)
    
    Note over Client,ResourceServer: Step 3: Use Token
    Client->>ResourceServer: GET /contacts (Authorization: Bearer <access_token>)
    ResourceServer-->>Client: Return Google contacts
```

**Key Components:**
1. **Resource Owner**: You (the user)
2. **Client**: App requesting access (Facebook)
3. **Authorization Server**: Issues tokens (Google OAuth)
4. **Resource Server**: Holds the data (Google contacts)

### OpenID Connect (OIDC)

```mermaid
graph TD
    A[OAuth 2.0] --> B[Authorization only]
    B --> B1[What can you do?]
    
    C[OpenID Connect] --> D[Authentication + Authorization]
    D --> D1[WHO are you? + What can you do?]
    D --> D2[ID Token = JWT with user info]
    
```

**OAuth 2.0 vs OpenID Connect:**
- **OAuth 2.0**: Authorization only (what can you do?)
- **OpenID Connect**: Built on OAuth 2.0, adds authentication (who are you?)
- **ID Token**: JWT containing user ID, name, email, profile picture

### OAuth 2.0 Flows

```mermaid
graph LR
    A[OAuth 2.0 Flows] --> B[Authorization Code Flow]
    A --> C[Implicit Flow - Discouraged]
    A --> D[Client Credentials Flow]
    A --> E[Device Code Flow]
    
    B --> B1[For server-side apps]
    C --> C1[For browser-based apps]
    D --> D1[For machine-to-machine]
    E --> E1[For smart TVs, limited input devices]
    
```

---

## Security Best Practices

### 1. Generic Error Messages

```mermaid
graph TD
    A[Authentication Failure] --> B{What happened?}
    
    B -->|Username not found| C[❌ Don't reveal!]
    B -->|Password incorrect| D[❌ Don't reveal!]
    
    C --> E[Authentication failed]
    D --> E
    
    E --> F[Attackers can't determine if username exists]
    
```

**Why?** Generic messages prevent:
- Attackers learning which usernames exist
- Targeted attacks on specific accounts
- Brute force optimization

**Always send:** "Authentication failed" (never specify if it's username or password)

### 2. Timing Attack Prevention

```mermaid
graph TD
    A[Login Request] --> B{Check Username}
    B -->|Exists| C[Hash Password]
    B -->|Not exists| D[Simulate delay]
    
    C --> E[Compare hashed password]
    E --> F[Return Authentication failed]
    
    D --> G[Delay ~200ms to simulate hashing]
    G --> F
    
```

**Timing Attack:** Attackers measure response time to determine:
- Fast response = Username invalid
- Slower response = Username valid, password wrong

**Defense:** 
- Use constant-time comparison functions
- Simulate delays for non-existent users

---

## Summary: Choosing the Right Authentication

```mermaid
graph TD
    A[Authentication Decision] --> B{What are you building?}
    
    B -->|Traditional Web App| C[Stateful - Sessions]
    B -->|APIs / Microservices| D[Stateless - JWT]
    B -->|Machine-to-Machine| E[API Keys]
    B -->|Third-party Login| F[OAuth 2.0 / OIDC]
    
    C --> C1[Examples: E-commerce, SaaS]
    D --> D1[Examples: Distributed systems, Mobile]
    E --> E1[Examples: ChatGPT API, Server-to-server]
    F --> F1[Examples: Login with Google, Login with Facebook]
    
```

### Quick Reference Table

| Authentication Type | Best For | Pros | Cons |
|------------------|----------|------|------|
| **Sessions (Stateful)** | Traditional web apps | Easy revocation, real-time control | Scalability issues |
| **JWT (Stateless)** | APIs, Microservices | No session store, scalable | Hard to revoke |
| **API Keys** | Machine-to-machine | Simple, programmatic access | Limited permissions |
| **OAuth 2.0 + OIDC** | Third-party login | Secure delegation, no passwords shared | Complex to implement |

---

## Final Advice for Backend Engineers

> **For learning:** Implement your own authentication to understand the concepts, tradeoffs, and security implications.
>
> **For production:** Use established auth providers (Auth0, Clerk, Firebase Auth) unless you're very confident in your authentication workflows.

### Key Takeaways
1. **Authentication** = WHO are you (identity)
2. **Authorization** = WHAT can you do (permissions)
3. **Sessions** = Server-side state (stateful)
4. **JWT** = Self-contained tokens (stateless)
5. **Cookies** = Browser storage mechanism
6. **API Keys** = Machine-to-machine auth
7. **OAuth 2.0** = Delegation protocol
8. **OpenID Connect** = OAuth + Authentication layer

---

**You now have a solid understanding of authentication & authorization - the foundation of secure backend systems!**
