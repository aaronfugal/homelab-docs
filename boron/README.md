# Boron

**Boron** is the LXC container in my Proxmox environment responsible for securely exposing internal services to the public internet using **Cloudflare Tunnel** and **NGINX**. It also forms part of the reverse proxy chain to route access to Apache Guacamole.

---

## 🔧 Services Hosted

### 1. Cloudflare Tunnel (`cloudflared`)
- Connects `boron` to my Cloudflare account using the domain: **protonsystems.org**
- Routes traffic to internal services through subdomain ingress rules:
  - `plex.protonsystems.org` → Plex Server (192.168.0.253:32400)
  - `proxmox.protonsystems.org` → Proxmox Web UI (192.168.0.252:8006)
  - `apache.protonsystems.org` → Guacamole (192.168.0.248:8080/guacamole)
- Config file path: `/etc/cloudflared/config.yml`
  - Contains tunnel ID, credentials, ingress rules, and 404 fallback
- Enforces **Full (strict)** SSL

### 2. NGINX (Port 8080)
- Reverse proxy for all exposed services
- Custom welcome page replaces default index
- Hosts static web content from `/var/www/html`
- Enforces **rate limiting** via **Fail2Ban**

---

## 🔐 Cloudflare Zero Trust

Cloudflare Zero Trust secures the exposed services using layered policy enforcement.

### Applications:
- Proxmox
- Plex
- Guacamole

### Policies:
1. **IP Bypass**  
   Allows specific trusted IPs to bypass identity checks.

2. **IdP - Allow OTP (Plex only)**  
   Permits login via Google, GitHub, or OTP (One-Time Password).

3. **Geofencing (Trust US/Canada)**  
   Blocks access from all non-US/Canada IP addresses.

4. **IdP - Require GitHub (Proxmox & Guacamole)**  
   Requires GitHub authentication and approved email match; blocks Google and OTP.

---

## 🧩 Linked Services

| Container   | Purpose                        | IP               | Notes                             |
|-------------|--------------------------------|------------------|-----------------------------------|
| `aluminum`  | Plex Media Server              | 192.168.0.253    | Routed through `plex.` subdomain |
| `beryllium` | Apache Guacamole (Remote UI)   | 192.168.0.248    | Routed through `apache.` subdomain |
| `tungsten`  | Proxmox Host                   | 192.168.0.252    | Routed through `proxmox.` subdomain |

---

## 🗃️ Files of Interest

- `/etc/cloudflared/config.yml`  
  Cloudflare tunnel configuration  
- `/etc/nginx/sites-available/`  
  Reverse proxy routing rules for each app  
- `/var/www/html/`  
  NGINX root for custom index page  
- `/etc/fail2ban/`  
  Rate limiting setup with NGINX jail  

---

## 🔒 Security Practices

- All traffic goes through **Cloudflare's edge**
- DNS records are proxied (orange cloud enabled)
- All apps require MFA or SSO via Cloudflare Zero Trust
- Geofencing limits access to US & Canada only
- Fail2Ban limits brute-force attempts and bad actors

---

## 📌 Notes

- Guacamole (`beryllium`) runs **Tomcat9** on port 8080, proxied through NGINX and Cloudflare.
- All services are containerized and firewall-restricted to local IP routing behind Proxmox.

