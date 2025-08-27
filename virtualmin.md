# 🌐 Web Application Deployment Architecture: Integrating Apache, NGINX, SSL, and phpMyAdmin

## 📝 Abstract

This paper investigates a practical deployment architecture for web applications using **Apache** as the application server, **NGINX** as a reverse proxy, SSL certificate management, and installation of **phpMyAdmin** for database administration. The configuration targets robust security, flexible routing, and maintainability for both production and development environments. Real-world examples are provided, including handling SSL errors and adapting to local deployments.

## 1️⃣ Introduction

Modern web applications often require a multi-layered server architecture to ensure performance, scalability, and security. This paper presents a configuration methodology using Apache and NGINX, with SSL for secure communication, and outlines steps for installing phpMyAdmin outside the web root for enhanced security.

## 🗂️ 2. System Architecture

### ⚙️ 2.1. Apache Configuration

Apache listens on a non-standard port (8080), isolating application traffic from direct internet access and allowing NGINX to handle public HTTP(S) requests.

```bash
# /etc/apache2/ports.conf
Listen 8080
```

```apache
# /etc/apache2/sites-available/sampad.me.conf
<VirtualHost 192.168.0.101:8080>
    # Application configuration
</VirtualHost>
```

### 🌉 2.2. NGINX Reverse Proxy Configuration

NGINX acts as the public-facing server, listening on ports 80 (HTTP) and 443 (HTTPS). All traffic is proxied to the Apache backend using the local network IP and port.

```nginx
# /etc/nginx/sites-available/sampad.me
server {
    listen 80;
    server_name sampad.com www.sampad.com;

    location / {
        proxy_pass http://192.168.0.101:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location /.well-known/ {
        proxy_pass http://192.168.0.101:8080/.well-known/;
        proxy_set_header Host $host;
    }
}

server {
    listen 443 ssl;
    server_name sampad.com www.sampad.com;
    ssl_certificate /etc/ssl/virtualmin/172466363954357/ssl.combined;
    ssl_certificate_key /etc/ssl/virtualmin/172466363954357/ssl.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256';

    location / {
        proxy_pass http://192.168.0.101:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location /.well-known/ {
        proxy_pass http://192.168.0.101:8080/.well-known/;
        proxy_set_header Host $host;
    }
}
```

### 🔗 2.3. Enabling NGINX Configuration

Symbolic linking and configuration testing ensure NGINX reads the intended configuration.

```bash
sudo ln -s /etc/nginx/sites-available/sampad.me /etc/nginx/sites-enabled/
sudo nginx -t
```

### 🔒 2.4. SSL Certificate Management

If SSL errors occur, update the certificate paths to use Apache's SSL files and verify supported protocols and ciphers:

```nginx
ssl_certificate /etc/apache2/ssl/apache.combined;
ssl_certificate_key /etc/apache2/ssl/apache.key;
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers 'ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256';
```

**🧑‍💻 Development Note:** For local testing, replace `server_name` with the backend IP address.

## 🛠️ 3. phpMyAdmin Installation

Install phpMyAdmin outside the web root directory to reduce exposure to direct attacks.

```bash
sudo apt-get install phpmyadmin -y
```

Ensure MariaDB (or MySQL) is configured to use the UNIX socket for secure database connections.

## 🛡️ 4. Security Considerations

- **Reverse Proxy**: NGINX shields Apache from direct internet exposure, making it harder for attackers to exploit Apache vulnerabilities.
- **SSL/TLS**: Correct protocol and cipher selection mitigates risks from deprecated or weak encryption.
- **phpMyAdmin**: Installing outside the web root and limiting access (e.g., via firewall rules or VPN) further secures the database management interface.
- **.well-known Handling**: Forwarding `.well-known` requests supports Let's Encrypt and other ACME-based certificate authorities.

## 👍 5. Best Practices

- Always test configuration changes using `nginx -t`.
- Use strong SSL ciphers and protocols.
- Keep Apache and NGINX updated.
- For local development, adjust domains to IPs and use self-signed certificates.
- Restrict access to phpMyAdmin via IP whitelisting or authentication.

## 🏁 6. Conclusion

This deployment architecture leverages best practices for security, maintainability, and scalability by combining Apache and NGINX, robust SSL management, and secure database administration with phpMyAdmin. Adjustments for local and production environments ensure flexibility, while careful configuration and testing minimize risks.

## 📚 References

- Apache HTTP Server Documentation
- NGINX Official Documentation
- MariaDB and MySQL Security Best Practices
- Let's Encrypt and ACME Protocol Standards
