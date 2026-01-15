## TCP vs UDP

### 1. First, the big question

When one computer sends data to another, how do we make sure:
- The data arrives?
- The data arrives **in the correct order**?
- Missing data is re-sent?

There are **two different answers** to this problem.

They are called:
- **TCP**
- **UDP**

---

### 2.Before we start: where TCP & UDP sit

From earlier blogs, our stack looks like this:

```
Application (HTTP, SSH, DB)
        ↓
Transport Layer (TCP or UDP)
        ↓
IP (IP Address)
        ↓
Network
```

TCP and UDP live in the **transport layer**.  
They decide **how data moves**, not **where it goes**.

---

### 3. TCP — Transmission Control Protocol

TCP is the **most important protocol** for DevOps engineers.

#### 3.1 What TCP guarantees

TCP guarantees:
- Data arrives
- Data arrives **in order**
- Lost data is **re-sent**
- Duplicate data is removed

This makes TCP **reliable**.

---

### 4. What “connection-based” actually means

Before sending data, TCP does something crucial.

It creates a **connection**.

Conceptually:
1. Client: “Can we talk?”
2. Server: “Yes”
3. Client: “Great”

This is called the **TCP handshake**.

If this handshake fails, **nothing else happens**.

---

### 5. Common DevOps errors caused by TCP failure

If TCP connection fails, you’ll see errors like:
- `Connection refused`
- `Connection timed out`
- `No route to host`

These errors mean:
- The app may be running
- But TCP cannot establish a connection

Usually due to:
- Firewall rules
- Closed ports
- App not listening
- Wrong IP or port

---

### 6. What uses TCP? (Very important list)

Almost everything you use daily:

| Protocol | Uses TCP? |
|--------|----------|
| HTTP | ✅ |
| HTTPS | ✅ |
| SSH | ✅ |
| FTP | ✅ |
| Databases | ✅ |

If reliability matters → TCP is used.

---

### 7. UDP — User Datagram Protocol

UDP solves a **different problem**.

#### 7.1 What UDP does (and doesn’t do)

UDP:
- Sends data
- Does **not** guarantee delivery
- Does **not** guarantee order
- Does **not** create a connection

UDP is **fast**, but unreliable.

---

### 8. UDP analogy (simple)

TCP is like:
> Making a phone call and talking step by step.

UDP is like:
> Shouting messages across a room and hoping they’re heard.

---

### 9. Why does UDP even exist?

Because sometimes:
- Speed matters more than perfection
- Losing some data is okay

Examples:
- DNS queries
- Video streaming
- Online gaming
- Metrics & monitoring systems

Retrying TCP connections would be **too slow** here.

---

### 10. What uses UDP? (DevOps-relevant)

| Use Case | Protocol |
|-------|---------|
| DNS (mostly) | UDP |
| Metrics | UDP |
| Service discovery | UDP |
| Streaming | UDP |

DNS often uses UDP first, and TCP only if needed.

---

### 11. Side-by-side comparison (important)

| Feature | TCP | UDP |
|------|----|----|
| Connection | Yes | No |
| Reliable | Yes | No |
| Ordered | Yes | No |
| Speed | Slower | Faster |
| DevOps usage | Very High | Medium |

---

### 12. How this affects DevOps debugging

When something breaks, ask:
- Is this **TCP-based** or **UDP-based**?
- Does it expect reliability?

Examples:
- HTTP API failing → TCP issue
- DNS sometimes failing → UDP + network issue
- Metrics missing → UDP packet loss

Understanding this changes **how you debug**.

---

### 13. Commands mapped to TCP & UDP

#### 13.1 TCP testing

Checks if TCP port 443 is open and reachable

```bash
nc -vz example.com 443
```

### 13.2 UDP testing

Sends a UDP packet to port 53 to see if it’s reachable (best-effort check)

```bash
nc -vu example.com 53
```

Notice:
- TCP checks for connection
- UDP just sends data

---

### 14. Real DevOps scenario

**Problem:**  
Your app can resolve DNS, but sometimes fails to connect.

**Why this happens:**
- DNS uses UDP
- UDP packets can be dropped
- Retry logic may be missing

This is why DNS caching and retries matter (next blogs!).
