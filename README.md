# Apache Forward & Reverse Proxy

An Apache HTTP Server configuration designed to act as both a **forward proxy** (for reaching external remote resources/internet) and a **reverse proxy** (for routing traffic to local development applications).

## Installation

Install Apache HTTP Server (`httpd`).

* **macOS (Homebrew):** `brew install httpd`
* **Ubuntu/Debian:** `sudo apt install apache2`
* **CentOS/RHEL:** `sudo yum install httpd`

---

## Configuration

### 1. Configure `httpd.conf`

Locate and open Apache’s main configuration file (`httpd.conf`).

#### Enable Proxy Modules
Uncomment (or add) the required proxy modules:

```apache
LoadModule proxy_module lib/httpd/modules/mod_proxy.so
LoadModule proxy_connect_module lib/httpd/modules/mod_proxy_connect.so
LoadModule proxy_http_module lib/httpd/modules/mod_proxy_http.so
```

#### Include Proxy Configuration
Include your custom proxy configuration file by adding the following directive:

```apache
Include /opt/homebrew/etc/httpd/extra/proxy.conf
```

---

### 2. Configure `proxy.conf`

Create or update the proxy configuration file at the path specified in your `Include` directive above.

#### Global Settings

Set Apache to listen on your designated proxy port:

```apache
Listen 8888
```

#### Reverse Proxy (Local Resources)

Add a VirtualHost block to route incoming requests to a local application (e.g., a web app running on your dev machine):

```apache
<VirtualHost *:8888>
    ServerName reverse.io
    ServerAlias local.myapp.com myapp.com
    
    ProxyRequests Off
    
    <Proxy *>
        Order deny,allow
        Deny from all
        Allow from all
    </Proxy>
    
    ProxyPass        / "http://local.myapp.com:3000/"
    ProxyPassReverse / "http://local.myapp.com:3000/"
    ProxyTimeout 300
    
    ErrorLog "/var/log/httpd/myproxy-reverse-error_log"
    CustomLog /var/log/httpd/myproxy-reverse-access_log" common
</VirtualHost>
```

#### Forward Proxy (Remote Resources / Internet)

Add a VirtualHost block to handle forward-proxied requests to external domains, APIs, and websites:

```apache
<VirtualHost *:8888>
    ServerName forward.io
    ServerAlias *
    
    ProxyRequests On
    ProxyVia On
    
    <Proxy *>
        Require ip 192.168.0
    </Proxy>
    
    ErrorLog "/var/log/httpd/myproxy-forward-error_log"
    CustomLog "/var/log/httpd/myproxy-forward-access_log" common
</VirtualHost>
```

---

## Usage & Testing

Restart Apache to apply changes:
* **macOS (Homebrew):** `brew services restart httpd`
* **Linux:** `sudo systemctl restart apache2` (or `httpd`)

### Testing the Reverse Proxy
Send a request with the configured host header to target your local app:
```bash
curl -H "Host: local.myapp.com" http://localhost:8888/
```

### Testing the Forward Proxy
Pass the proxy flag via `curl` to route an outbound HTTP request through Apache:
```bash
curl -x http://localhost:8888 http://example.com
```

---

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss the proposed changes.

## License

[GNU GPL 3.0](https://www.gnu.org/licenses/gpl-3.0.en.html)