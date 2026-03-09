# HAProxy Server Setup Guide

Complete step-by-step guide for setting up HAProxy on Ubuntu/Linux server.

## Table of Contents
- [Installation](#installation)
- [Basic Configuration](#basic-configuration)
- [Monitoring](#monitoring)
- [Testing Setup](#testing-setup)
- [Production Enhancements](#production-enhancements)

---

## Installation

### 1. Update System
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install HAProxy
```bash
sudo apt install haproxy -y
```

Verify installation:
```bash
haproxy -v
```

Expected output:
```
HAProxy version 2.x.x
```

### 3. Enable and Start Service
```bash
sudo systemctl enable haproxy
sudo systemctl start haproxy
sudo systemctl status haproxy
```

Expected status: `active (running)`

---

## Basic Configuration

### 4. Backup Default Config
```bash
sudo cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
```

### 5. Edit Configuration
```bash
sudo nano /etc/haproxy/haproxy.cfg
```

Basic working configuration:
```haproxy
global
    log /dev/log local0
    log /dev/log local1 notice
    maxconn 2000
    daemon

defaults
    log global
    mode http
    option httplog
    timeout connect 5s
    timeout client 50s
    timeout server 50s

frontend http_front
    bind *:80
    default_backend web_servers

backend web_servers
    balance roundrobin
    server web1 10.0.0.11:80 check
    server web2 10.0.0.12:80 check
```

> **Note:** Replace backend IPs with your actual web servers.

### 6. Validate Configuration
**Always validate before restarting:**
```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
```

Expected output:
```
Configuration file is valid
```

### 7. Apply Changes
```bash
sudo systemctl restart haproxy
```

Verify listening ports:
```bash
ss -tulnp | grep haproxy
```

Expected: `*:80`

### 8. Test Load Balancer
Open browser:
```
http://<haproxy-server-ip>
```

Traffic will be distributed between backend servers.

---

## Monitoring

### 9. Enable Stats Dashboard
Add to configuration:
```haproxy
listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 10s
```

Restart service:
```bash
sudo systemctl restart haproxy
```

Access dashboard:
```
http://<SERVER-IP>:8404/stats
```

---

## Testing Setup

### 10. Local Testing with Docker
If you don't have backend servers, use Docker:

**Start test servers:**
```bash
# Server 1
docker run -d -p 8081:80 nginx

# Server 2
docker run -d -p 8082:80 nginx
```

**Update HAProxy config:**
```haproxy
backend web_servers
    balance roundrobin
    server web1 127.0.0.1:8081 check
    server web2 127.0.0.1:8082 check
```

Refresh browser to see traffic switching between servers.

---

## Production Enhancements

After basic setup, consider adding:

- **SSL/TLS Termination** - Secure traffic with HTTPS
- **Advanced Health Checks** - Custom health check endpoints
- **Rate Limiting** - Protect against abuse
- **Enhanced Logging** - Detailed access and error logs
- **High Availability** - Keepalived for failover
- **Path-Based Routing** - Route by URL patterns
- **ACLs & Security** - Access control lists
- **Kubernetes Integration** - Ingress controller style routing