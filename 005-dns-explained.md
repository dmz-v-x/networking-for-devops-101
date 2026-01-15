## DNS Explained

### 1. Quick recap (from earlier blogs)

From Part 1:
> Computers do NOT understand domain names.  
> They only understand **IP addresses**.

DNS exists to translate:

```
myapp.com → 13.234.56.78
```

DNS happens **before**:
- TCP connection
- HTTPS
- Any application logic

If DNS fails, nothing else matters.

---

### 2. What actually happens during DNS resolution

When your browser wants to reach:

```
https://myapp.com
```

It asks:

> “What IP address belongs to `myapp.com`?”

But **who does it ask**?

---

### 3. DNS resolution flow (simplified)

```
Browser
  ↓
OS DNS Cache
  ↓
Local Resolver (ISP / 8.8.8.8 / 1.1.1.1)
  ↓
Authoritative DNS Server
  ↓
IP Address returned
```

DNS is **distributed**, not centralized.

This design makes DNS:
- Fast
- Scalable
- Cache-heavy (important!)

---

### 4. What is DNS caching?

To avoid repeating DNS lookups:
- Browsers cache results
- OS caches results
- DNS resolvers cache results

Each cached record has a **TTL**.

---

### 5. TTL — Time To Live

TTL tells DNS:
> “How long can I remember this answer?”

Example:
```
TTL = 300 seconds (5 minutes)
```

Meaning:
- DNS answer can be reused for 5 minutes
- No fresh lookup needed during this time

---

### 6. Real DevOps DNS failure example

**Scenario:**
- You update your server IP
- You update DNS record
- Some users still hit the old server

**Why?**
- Old IP is cached
- TTL hasn’t expired yet

DNS is **eventually consistent**, not instant.

---

### 7. Common DNS record types (DevOps must-know)

You do NOT need to know all DNS records.

These are enough.

#### 7.1 A Record
Maps a domain to an IP address.

```
myapp.com → 13.234.56.78
```

#### 7.2 CNAME Record
Maps one domain to another domain.

```
www.myapp.com → myapp.com
```

Used for:
- Aliases
- Load balancers
- Cloud services

#### 7.3 TXT Record
Stores arbitrary text.

Used for:
- Domain verification
- SSL certificates
- Email security

Used often when working with cloud providers.

---

### 8. Why CNAMEs matter in DevOps

CNAMEs allow:
- Changing infrastructure without changing domains
- Blue/green deployments
- Easier migrations

Example:
```
app.mycompany.com → alb-123.aws.com
```

You update the load balancer — DNS stays same.

---

### 9. Common DNS debugging commands

#### 9.1 Check DNS resolution
```bash
dig myapp.com
```

#### 9.2 Use a specific DNS server
```bash
dig myapp.com @8.8.8.8
```

#### 9.3 See only the IP
```bash
dig myapp.com +short
```

#### 9.4 Using nslookup
```bash
nslookup myapp.com
```

---

### 10. Why DNS works for some users and not others

Because:
- Different users use different DNS resolvers
- Different caches expire at different times
- ISPs cache aggressively

This is normal DNS behavior, not a bug.

---

### 11. DNS vs TCP vs HTTP (clear separation)

| Layer | Failure Example |
|----|----|
| DNS | Domain not resolving |
| TCP | Connection refused |
| TLS | Certificate error |
| HTTP | 500 error |
| App | Logic bug |

Always identify **which layer is failing**.

---

### 12. Classic DevOps mistake

**Mistake:**  
“DNS change didn’t work — let’s restart the server.”

Wrong.

DNS:
- Lives outside your server
- Is cached everywhere
- Needs time to propagate

Restarting apps won’t fix DNS.

---

### 13. Mini hands-on exercise

Run:
```bash
dig google.com
```

Look at:
- ANSWER section
- TTL value

Run it again:
- Notice TTL decreases

That’s caching in action.

---

### 14. Mental model upgrade

Your updated flow:

```
Browser
  ↓
DNS (cached or fresh)
  ↓
IP + Port
  ↓
TCP / UDP
  ↓
Application
```

DNS is **always first**.

---
