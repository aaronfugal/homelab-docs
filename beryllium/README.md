# Beryllium

**Beryllium** is the container that runs Apache Guacamole (v1.5.5), providing web-based remote access to machines in my homelab through a hardened, database-backed, and MFA-secured setup. It is reverse proxied through Boron and protected by Cloudflare Access.

---

## 🧹 System Overview

- **OS**: Ubuntu 22.04 LXC (Container ID 1004 on Proxmox `tungsten`)
- **guacd**: v1.5.5, built from source, bound to `127.0.0.1:4822`
- **Tomcat 9**: Hosts the Guacamole web application (`guacamole.war`)
- **MariaDB**: Stores authentication data and user permissions (`guacamole_db`)
- **Nginx**: Reverse proxy on Boron (`apache.protonsystems.org`)
- **Cloudflare Access**: Gatekeeper for external web access
- **Authentication**: JDBC + TOTP (multi-factor)

---

## 🔐 Security and Hardening

- `guacd` runs in the background with logs to `/var/log/guacd.log`, bound only to localhost.
- All connections to Guacamole are routed through NGINX → Cloudflare Access → Tomcat.
- User access is managed with database roles, and **TOTP** is required for all logins.

---

## 🛠️ Configuration Summary

### guacd

- **Binary**: `/usr/local/sbin/guacd`
- **Systemd ExecStart**:
  ```bash
  /usr/local/sbin/guacd -f -p /run/guacd.pid -b 127.0.0.1 -l 4822 -L info
  ```

### Tomcat + Guacamole

- `guacamole.war` deployed to `/var/lib/tomcat9/webapps/`
- `GUACAMOLE_HOME=/etc/guacamole`
- `guacamole.properties` configures:
  ```properties
  guacd-hostname: localhost
  guacd-port: 4822
  mysql-hostname: localhost
  mysql-database: guacamole_db
  totp-issuer: ProtonSystems
  totp-required: true
  ```

### Database

- Schema created using Guacamole JDBC extension (`guacamole-auth-jdbc-1.5.5.tar.gz`)

- SQL setup:

  ```sql
  CREATE DATABASE guacamole_db CHARACTER SET utf8mb4;
  CREATE USER 'aaron'@'localhost' IDENTIFIED BY '<secure>';
  GRANT SELECT, INSERT, UPDATE, DELETE ON guacamole_db.* TO 'aaron'@'localhost';
  FLUSH PRIVILEGES;
  ```

- Admin Users:

  - `guacadmin`: default admin, password updated
  - `aaron`: permanent admin
  - `juniordev`: read-only SSH access to `tungsten`

---

## 🔒 Multi-Factor Authentication (TOTP)

- TOTP extension (`guacamole-auth-totp-1.5.5.jar`) extracted to `/etc/guacamole/extensions`

- Enforced on all users via:

  ```properties
  totp-required: true
  ```

- Users are prompted with a QR code at first login

---

## 🌐 Reverse Proxy: Boron → Guacamole

Nginx server block (on `boron`):

```nginx
server {
  listen 443 ssl http2;
  server_name apache.protonsystems.org;

  ssl_certificate     /etc/nginx/certs/origin.crt;
  ssl_certificate_key /etc/nginx/certs/origin.key;

  limit_req zone=one burst=10 nodelay;

  location = / {
    return 302 /guacamole/;
  }

  location /guacamole/ {
    proxy_pass         http://192.168.0.248:8080/guacamole/;
    proxy_http_version 1.1;
    proxy_buffering    off;
    proxy_read_timeout 60s;

    proxy_set_header   Upgrade           $http_upgrade;
    proxy_set_header   Connection        $connection_upgrade;
    proxy_set_header   Host              $host;
    proxy_set_header   X-Real-IP         $remote_addr;
    proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
  }
}
```

- `$connection_upgrade` mapping handled in global `http {}` block
- Access protected by **Cloudflare Access**, prior to any login prompt

---

## 🧑‍💻 User Access

| Username    | Role         | Notes                                            |
| ----------- | ------------ | ------------------------------------------------ |
| `guacadmin` | Legacy Admin | Default account with updated password            |
| `aaron`     | Admin        | Full permissions, created via UI                 |
| `juniordev` | Limited User | Access only to SSH into `tungsten`, MFA enforced |

---

## 📌 Notes

- This setup provides full browser-based SSH/RDP/VNC access to internal machines.
- All external access passes through Cloudflare + NGINX before reaching Beryllium.
- The Guacamole UI runs on port `8080` but is **never** exposed directly.
- All users must register a TOTP app (e.g., Authy, Google Authenticator) on first login.

