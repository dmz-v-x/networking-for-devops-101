## Subnet, CIDR & NAT

### 1. First: why do subnets even exist?

Let’s start with a simple question.

Why don’t we just put **all computers on one big network**?

Because:
- It would be insecure
- It would be chaotic
- It would not scale

So networks are **divided into smaller networks**.

These smaller networks are called **subnets**.

---

### 2. What is a subnet?

A **subnet** is:
> A **group of IP addresses** that belong to the same internal network.

That’s it.

Example:
```
10.0.0.0 – 10.0.0.255
```

All machines in this range:
- Can talk to each other directly
- Are part of the same private network

---


### 3. Why cloud providers force you to use subnets

In cloud platforms (AWS, GCP, Azure):
- You don’t get a “flat” network
- You must create:
  - VPC (virtual private network)
  - Subnets inside it

Why?
- Security isolation
- Routing control
- Scalability

This is why you always see subnet selection during VM creation.

---

### 4. What is CIDR (the scary `/24` thing)?

CIDR looks scary but is simple.

Example:
```
10.0.0.0/24
```

CIDR means:
> “How many IPs belong to this network”

### 4.1 CIDR without math (promise)

Here’s the only thing you need to remember:

| CIDR | Approx IPs |
|----|-----------|
| /24 | ~256 IPs |
| /16 | ~65,000 IPs |
| /8 | ~16 million IPs |

Smaller number after `/` → bigger network  
Bigger number after `/` → smaller network

That’s enough for DevOps.

### 4.2 What `/24` actually means

```
10.0.0.0/24
```

Means:
- Network starts at `10.0.0.0`
- Ends at `10.0.0.255`
- Total ≈ 256 IPs

Cloud providers reserve some IPs internally.

---

### 5. Why CIDR matters in real DevOps work

CIDR decides:
- How many servers you can run
- How isolated your network is
- Whether services can talk to each other

Common mistake:
> Choosing a subnet too small → running out of IPs

---

### 6. Public vs Private subnets

#### 6.1 Public Subnet
- Has a route to the internet
- Used for:
  - Load balancers
  - Bastion hosts
  - Public-facing services

#### 6.1 Private Subnet
- No direct internet access
- Used for:
  - App servers
  - Databases
  - Internal services

Best practice:
> Only expose what must be public.

---

### 7. How do private subnets access the internet?

This is where **NAT** comes in.

#### 7.1 What is NAT (Network Address Translation)?

NAT allows:
```
Private IPs → Internet
```

Without exposing:
```
Internet → Private IPs
```

NAT is **one-way** by default.

#### 7.2 NAT explained simply

Private server:
```
10.0.1.5
```

Wants to access:
```
google.com
```

Flow:
```
10.0.1.5 → NAT Gateway → Internet
```

Internet sees:
```
Public IP → Internet
```

Replies come back through NAT.

---

### 8. Why NAT is critical for security

Without NAT:
- Every private server needs a public IP
- Everything is exposed

With NAT:
- Servers stay private
- Only outbound traffic allowed

This is why databases are almost always in private subnets.

---

### 9. Common DevOps architecture

```
Internet
   ↓
Load Balancer (Public Subnet)
   ↓
App Servers (Private Subnet)
   ↓
Database (Private Subnet)
```

NAT allows:
- App → Internet (updates, APIs)
- Database → No internet access

---

### 10. How subnet issues show up in real life

| Problem | Likely Cause |
|------|-------------|
| Can’t reach DB | Wrong subnet |
| App can’t access internet | Missing NAT |
| Server unreachable | Public IP missing |
| Only some services talk | Routing issue |

Most “network issues” are subnet or NAT misconfigurations.

---

### 11. Mini hands-on mental exercise

Ask yourself:
- Is this service public or private?
- Does it need inbound access?
- Does it need outbound internet?

Those answers decide:
- Subnet
- NAT
- Security rules

---

### 12. Mental model upgrade

Your networking stack now looks like:

```
Internet
  ↓
Public Subnet
  ↓
Private Subnet
  ↓
Service
```

Subnets decide **who can talk to whom**.

---
