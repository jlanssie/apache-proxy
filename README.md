# Apache Gateway

An Apache gateway for reaching a local web app on a dev machine from a test machine (such as a phone) and simultaneously reaching remote APIs.

## Install Apache 

Install Apache httpd

## Configure Apache httpd.conf

Find Apache httpd's main configuration file `httpd.conf` and open the file.

Load the proxy modules by uncommenting them, e.g.

```apache
LoadModule proxy_module lib/httpd/modules/mod_proxy.so
LoadModule proxy_connect_module lib/httpd/modules/mod_proxy_connect.so
LoadModule proxy_http_module lib/httpd/modules/mod_proxy_http.so
````

Include the proxy configuration by adding a reference to that config file, e.g.

```apache
Include /opt/homebrew/etc/httpd/extra/gateway.conf
````

## Configure gateway 

Place the config file at the path specified in the Include directive. 

Add a listen directive. 

````apache
Listen 8888
````

Add a vhost config for reverse-proxied requests to your local resource, e.g. the web app on your dev machine.

````apache
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
    ProxyPassReverse / "http://local.myapp.com:3000/""
    ProxyTimeout 300
    ErrorLog "/opt/homebrew/var/log/httpd/myapp-reverse-error_log"
    CustomLog "/opt/homebrew/var/log/httpd/myapp-reverse-access_log" common
</VirtualHost>
````

Add a vhost config for forward-proxied requests to remote resources, e.g. domains, APIs, websites on the internet. 

````apache
<VirtualHost *:8888>
    ServerName forward.io
    ServerAlias *
    ProxyRequests On
    ProxyVia On
    <Proxy *>
      Require ip 192.168.0
    </Proxy>
    ErrorLog "/opt/homebrew/var/log/httpd/myapp-forward-error_log"
    CustomLog "/opt/homebrew/var/log/httpd/myapp-forward-access_log" common
</VirtualHost>
````
