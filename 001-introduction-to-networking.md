## Introduction to Networking

### 1. Why should DevOps engineers even care about networking?

Let’s start from basics.

Most DevOps failures are **not code bugs**.

They look like this:

- ❌ “The app works locally but not in production”
- ❌ “The service is running but not reachable”
- ❌ “502 Bad Gateway”
- ❌ “Connection refused”
- ❌ “Request timed out”

All of these comes down to one problem that is **networking problems**.

If you don’t understand networking:
- You won’t know **where** the problem is
- You’ll randomly try fixes
- You’ll depend on someone else to debug

This blog series exists to change that — **from zero**.

---

### 2. Goal of this series

By the end of this series, you should be able to:

- Understand how traffic flows from **browser → cloud → server**
- Debug basic production networking issues
- Understand Docker & Kubernetes networking at a high level
- Read cloud architecture diagrams without fear
- And hopefully, **you and I both can confidently answer networking interview questions**

---

### 3. The ONE mental model we must remember

Everything in networking can be explained using **three boxes**:

```
Client (Browser / App)
        ↓
     Network
(DNS, Internet, Load Balancer)
        ↓
Server (VM / Container / App)
```

**Rule:**  
When something breaks, the problem is **always** in one of these boxes or the connection between them.

We’ll keep coming back to this model in every blog.

---

### 4. DNS — turning names into addresses

When you type any url into our browser:

For Example: 

```
https://google.com
```

Your computer does **not** understand `google.com`.

Computers only understand **numbers** (IP addresses).

👉 DNS exists to solve this problem.

**DNS = Name → IP address**
Means resolving Name(address/url) to IP Address.

Example:
```
google.com → 93.184.216.34
```

Think of DNS like a **phone book**:
- You search by name
- You get a number

If DNS breaks, **nothing works** — even if your server is perfectly fine.

---

### 5. IP addresses & Ports — where and what to connect to

#### 5.1 IP Address
An IP address identifies a **machine on a network**.

Examples:
- `192.168.1.10` (private IP)
- `13.234.56.78` (public IP)

Public IP → reachable from the internet  
Private IP → only reachable inside a private network (like cloud VPCs - Virtual Private Cloud)

---

#### 5.2 Port - Which application?

A port identifies **which application** on that machine we want to talk to.

Okay, so when we connect to an application, we are actually connecting to an IP address where that application is running.

However, this does not mean that only one application can run on a single IP address. In reality, **mulitple application can run on the same IP address**, and they are distinguished from each other using different port numbers.

Example: Let say we have a server that has an IP: 

`192.168.1.10`

Now on this same IP, we might have:

Web app running on port 80
Backend API running on port 3000
Database running on port 5432

So:

192.168.1.10:80 → Web application
192.168.1.10:3000 → Backend service
192.168.1.10:5432 → Database

Common Ports:
- `22` → SSH
- `80` → HTTP
- `443` → HTTPS
- `5432` → PostgreSQL

IP = *which computer*  
Port = *which app on that computer*

---

### 6. TCP vs UDP — how data is sent

This is about **rules of communication**.

#### 6.1 TCP (most important for DevOps)

- Reliable
- Ordered
- Connection-based

Used by:
- HTTP / HTTPS
- SSH
- Databases

Think of TCP like a **phone call**:
- You connect
- You talk
- You hang up


#### 6.2 UDP (less common, but important)

- Fast
- No guarantee of delivery
- No connection

Used by:
- DNS (often)
- Streaming
- Some internal systems

Think of UDP like shouting messages — fast, but no confirmation.

---

### 7. DNS Resolution — What’s Really Happening?

Computers cannot understand domain names.  
Computers only understand IP addresses like:

`93.184.216.34`

So when we type a domain like `google.com`, it must be converted into an IP address first.

👉 **Finding the IP address of a domain name is called DNS resolution.**

If DNS fails → **nothing else matters**.

---

### 8. What Happens Before ping?

Before anything else, we always ask:

> What is the IP address of `google.com`?

Internally, this happens:

```
google.com → DNS → 93.184.216.34
```

So if DNS fails, **everything fails**.

---

### 9. What ping Actually Does

Once the IP is known, we use `ping`.

Ping sends a special network message called an **ICMP Echo Request**.

It’s like asking the server:

> “Hey server, are you alive?”

If the server is reachable and allows ping, it replies with an **ICMP Echo Reply**.

Your terminal prints something like:

```
64 bytes from 93.184.216.34: time=12ms
```

This means:
- The server is reachable
- The network path works
- The round-trip time is 12 milliseconds

By default, ping keeps sending packets every second:

```
ping → reply
ping → reply
ping → reply
```

Until you stop it with:

```
Ctrl + C
```

---

### 10. What ping Verifies vs What It Does NOT

#### What ping verifies:
- The IP is reachable over the network
- Basic connectivity exists

#### What ping does **NOT** verify:
- Application is running
- Website is working
- Port is open
- Server is healthy

A server can reply to ping and still have a down website.

---

### 11. Why ping Sometimes Fails but Website Works

Sometimes this happens:

- `ping google.com` → ❌ fails  
- `curl https://google.com` → ✅ works  

This does **not** mean the site is down.

Many servers block ICMP (ping) for security reasons.

---

### 12. Does ping Do DNS Resolution?

**Yes**, ping does DNS resolution — but only when we give it a domain name.

Example:

```
ping google.com
```

Internally:
- ping sees the domain
- It asks DNS to resolve it
- DNS responds with:
  ```
  google.com → 93.184.216.34
  ```
- ping sends ICMP packets to that IP

📌 **Point to remember:**  
Ping does DNS only as a **prerequisite**, not as its main job.

If you provide an IP directly, DNS is skipped completely.

---

### 13. Why We Still Need dig & nslookup

**Question:**  
If ping does DNS resolution, why do we need `dig` and `nslookup`?

**Answer:**  
Ping does DNS *just enough to proceed*.

`dig` and `nslookup` exist to **inspect and debug DNS itself**.

---

### 14 ping vs dig / nslookup

| Aspect                                               | `ping`                                        | `dig / nslookup`                            |
|----------------------------------------------------|---------------------------------------------|---------------------------------------------|
| **Primary question answered**                      | After resolving the name, can I reach the IP? | Is DNS working correctly?                   |
| **Main purpose**                                   | Network reachability                          | DNS resolution & configuration              |
| **Uses DNS?**                                      | Yes (minimal)                                 | Yes (main focus)                            |
| **Shows DNS server**                               | ❌ No                                         | ✅ Yes                                      |
| **Shows DNS response time**                        | ❌ No                                         | ✅ Yes                                      |
| **Shows record types**                             | ❌ No                                         | ✅ Yes                                      |
| **Shows multiple IPs**                             | ❌ No                                         | ✅ Yes                                      |
| **Shows TTL**                                      | ❌ No                                         | ✅ Yes                                      |
| **Affected if ICMP is blocked**                    | ❌ Yes                                        | ✅ No                                       |

---

### 15. ping Domain vs ping IP

#### CASE A: `ping google.com`

- Domain name is provided
- DNS lookup happens:
  ```
  google.com → 93.184.216.34
  ```
- Ping sends packets to the IP
- DNS involved only once

#### CASE B: `ping 93.184.216.34`

- IP address is provided
- DNS is skipped completely
- Ping sends packets directly

#### One-line difference:

| Command       | DNS involved? | What is pinged        |
|--------------|---------------|-----------------------|
| `ping domain` | ✅ Yes        | Application server IP |
| `ping IP`     | ❌ No         | Application server IP |

---

### 16. Why Cloud Servers Block ping

Ping uses **ICMP**, which is a low-level network check.

Security reasons:
- Attackers can find live servers
- Infrastructure mapping becomes easy
- Can lead to DDoS scanning

Users don’t use ICMP.  
Users use **HTTP, HTTPS, APIs, TCP connections**.

So blocking ping does not affect real users.

Common real-world situation:

```
ping myserver.com  → fails
curl https://myserver.com → works
```

This means:
- Server is running
- Application is working
- ICMP is intentionally blocked

---

### 17. ICMP vs TCP vs HTTP

#### ICMP
- Internet Control Message Protocol
- Used only to check reachability
- Answers: *Can I reach this server at all?*

#### TCP
- Transmission Control Protocol
- Opens reliable connections
- Talks to applications on ports

#### HTTP / HTTPS
- Application-level protocols
- Built on top of TCP
- Used by real users

#### Comparison

| Thing          | ICMP (Ping)  | TCP / HTTP         |
|---------------|-------------|--------------------|
| Purpose        | Reachability | Real communication |
| Uses ports?    | ❌ No        | ✅ Yes             |
| Talks to app?  | ❌ No        | ✅ Yes             |
| Blocked often? | ✅ Yes       | ❌ Rarely          |
| Used by users? | ❌ No        | ✅ Yes             |



### 18. Check DNS Resolution

```bash
dig google.com +short
```

- We ask the `dig` tool:  
  “Hey DNS, what is the IP address of google.com?”
- DNS replies with an IP like: `74.125.68.101`
- `+short` means: show only the final IP

---

### Alternative: nslookup

```bash
nslookup google.com
```

`nslookup` (Name Server Lookup) does the same job as `dig` but also shows:
- The DNS server used
- The response details

Example output:

```bash
Server:  8.8.8.8
Address: 8.8.8.8#53

Non-authoritative answer:
Name:    google.com
Address: 93.184.216.34
```

---

### 19. Check Basic Connectivity

```bash
ping google.com

OR

ping 93.184.216.34
```

If ping fails:
- Network issue
- Firewall issue
- Host down

---

### 20. Check If a Port Is Open (nc)

```bash
nc -vz google.com 443
```

What is `nc` (Netcat)?
- Networking testing tool
- Opens TCP connections
- Tests if a port is reachable

Flags:
- `-v` → verbose
- `-z` → zero-I/O (no data sent)

Internal steps:
1. DNS resolution
2. TCP 3-way handshake
3. Result printed

If you see:
- `connection refused` → service or firewall issue
- `timeout` → routing or security group issue

---

### 21. Test HTTP Directly (curl)

```bash
curl -I https://google.com
```

- `curl` = Client URL
- `-I` = HEAD request (headers only)

Internal flow:
1. DNS resolution
2. TCP handshake
3. HTTP request
4. Response received

---

### 22. Real DevOps Debugging Order

```text
1. DNS        → dig
2. Reachable  → ping
3. Port open  → nc
4. App works  → curl
```

Example:

```bash
dig google.com
ping google.com
nc -vz google.com 443
curl https://google.com
```

---
