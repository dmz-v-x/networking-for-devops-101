## IP Addresses & Ports

### 1. What is an IP address (really)?

An **IP address** identifies a **machine on a network**.

That’s it.

Just remember this:
> **IP address = identity of a machine on a network**

Examples:
```
192.168.1.10
10.0.0.5
13.234.56.78
```

Every machine that communicates on a network must have an IP.

---

### 2. Local IP vs Internet IP (very first distinction)

Your laptop has:
- One IP **inside your local network**
- Possibly another IP **visible to the internet**

Why? Because networks are layered.

Example:
```
Laptop → WiFi Router → Internet
```

Your laptop is **not directly exposed** to the internet.

---

### 3. Public IP Address (Internet-facing)

A **public IP**:
- Is unique on the internet
- Can be accessed from anywhere
- Is assigned by ISPs or cloud providers

Example:
```
13.234.56.78
```

Cloud servers usually have:
- A public IP (optional)
- A private IP (always)

---

### 4. Private IP Address (Internal-only)

A **private IP**:
- Works only inside a private network
- Cannot be reached directly from the internet

Common private IP ranges:
```
10.0.0.0    – 10.255.255.255
172.16.0.0 – 172.31.255.255
192.168.0.0 – 192.168.255.255
```

Your home WiFi, office network, and cloud VPCs all use private IPs.

---

### 5. Why private IPs exist

The internet does **not** have enough IPs for every device.

So:
- Private IPs are reused everywhere
- Public IPs act as gateways

This design allows:
- Security (not everything exposed)
- Scalability
- Cost efficiency

This leads us to a **very important idea** (we’ll learn this concept later):
👉 **NAT (Network Address Translation)**

---

### 6. Tiny preview: What NAT does

NAT allows:
```
Many private IPs → One public IP
```

Your laptop sends traffic:
```
192.168.1.10 → Internet
```

The router translates it to:
```
Public IP → Internet
```

---

### 7. What is a port?

If an IP identifies a **machine**, a **port** identifies a **program on that machine**.

One machine can run **many applications**:
- Web server
- Database
- SSH
- Cache

Ports make this possible.

---

### 8. Port analogy (simple but powerful)

Think of:
- IP address → apartment building
- Port → apartment number

```
13.234.56.78:22   → SSH
13.234.56.78:80   → HTTP
13.234.56.78:443  → HTTPS
```

Same building, different doors.

---

### 9. Common ports every DevOps engineer must know

| Port | Purpose |
|----|----|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |

---

### 10. How IP + Port work together

A **network connection** always needs:

```
IP address + Port
```

Examples:
```
ssh user@13.234.56.78     → port 22
https://example.com       → port 443
localhost:3000            → port 3000
```

Without a port, the OS doesn’t know *which app* should receive the data.

---

### 11. `localhost` vs `127.0.0.1`

When we run an application on our laptop, that's when `localhost` and `127.0.0.1` comes into picture.

#### 11.1 What is 127.0.0.1?

`127.0.0.1` is an IP address, specially it is called loopback IP which means if any network request sent to this IP it comes back to the same machine. 

The request never leaves our machine, no router, no internet and no external network is involved.

#### 11.2 What is `localhost`?

`localhost` is a hostname, it's not an ip. When we type `localhost` our operating system translates this name into an IP address that is `127.0.0.1`

`localhost` → `127.0.0.1`

This translation happens via
- /etc/hosts file (Linux/macOS)
- C:\Windows\System32\drivers\etc\hosts (Windows)

#### 11.3 How does `localhost` actuallly works?

When we type: http://localhost:3000, internally our operating system follow these steps:
- OS sees `localhost`
- It looks up the hosts file
- Resolve it to an IP (usually `127.0.0.1`)
- Send the request to that IP

So the flow becomes:

`localhost` → `127.0.0.1` → your own machine

#### 11.4 Are `localhost` and `127.0.0.1` same?

Short Answer: No

- `127.0.0.1` is hard-coded as a loopback IP
- `localhost` is configurable

Our system can map `localhost` to something else

In many modern systems:

`localhost` → ::1

::1 is the IPv6 loopback address.

#### 11.4 Which one should we use?

Use `localhost` when:
- Developing locally
- You don't care about IPv4 vs IPv6

Use `127.0.0.1` when:
- Debugging networking issues
- Want guaranteed IPv4
- Dealing with Docker, databases, or binding issues

#### 11.5 A DevOps Example:

If our server logs say:

Listening on `127.0.0.1`

That means:
- Only this machine can access it.
- Not accessible from other devices.

If it listens on:

0.0.0.0

That means:
- Accept connections from any network interface.

#### 11.6 Complete flow

```
We type in browser:
   `localhost`
       ↓
 OS resolves name
       ↓
   `127.0.0.1` or ::1
       ↓
 Request comes back to
   your own machine

```

---

### 12. Classic DevOps failure example

**Problem:**  
App works on your laptop but not on the cloud server.

**What’s happening:**
- App is listening on `localhost`
- Cloud traffic comes to **public IP**
- App never sees it

Fix:
- Bind app to `0.0.0.0`
- Open the correct port
- Allow traffic in firewall/security group

We’ll fully break this down later.

---

### 13. Hands-on commands 

#### 13.1 Check your IPs
```bash
ip addr        # Linux
ifconfig       # macOS
```

#### 13.2 See listening ports
```bash
ss -tuln
```

or
```bash
netstat -tuln
```

#### 13.3 Test remote port
```bash
nc -vz 13.234.56.78 443
```

---
