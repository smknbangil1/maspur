# Konfigurasi Nginx-Varnish-Apache2
## nginx ssl termination reverse proxy
```bash
server {
    listen 443 ssl http2;
    server_name domain.my.id;
    ssl_certificate           /etc/letsencrypt/live/domain.my.id/fullchain.pem;
    ssl_certificate_key       /etc/letsencrypt/live/domain.my.id/privkey.pem;

    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_protocols TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    ssl_prefer_server_ciphers on;

    access_log            /var/log/nginx/domain.my.id_access.log;
    error_log             /var/log/nginx/domain.my.id_error.log;

        location / {
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
            proxy_pass_request_headers on;
            proxy_pass http://127.0.0.1:80;
        }

    }
```
## Varnis Cache Reverse Proxy 
```bash 
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}

sub vcl_recv {
    # Happens before we check if we have this in cache already.
    #
    # Typically you clean up the request here, removing cookies you don't need,
    # rewriting the request, etc.
    if (client.ip != "127.0.0.1" && req.http.host ~ "^(www.)?domain.my.id$") {
       set req.http.x-redir = "https://domain.my.id" + req.url;
       return(synth(850, ""));
    }
}

# tambahkan pada baris bawah sendiri
sub vcl_synth {
    if (resp.status == 850) {
       set resp.http.Location = req.http.x-redir;
       set resp.status = 301;
       return (deliver);
    }
}
```
### virtualhost apache 
```bash
<VirtualHost *:8080>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/moodle

        <FilesMatch \.php$>
                SetHandler "proxy:unix:/var/run/php/php7.4-fpm.sock|fcgi://localhost"
        </FilesMatch>
        
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
## Cara Mengatasi Apache2 [mpm_event:notice] [pid 677:tid 139638138522688] AH00491: caught SIGTERM, shutting down 
```bash
apache2ctl -S
```
perintah diatas akan menunjukkan kesalahan konfig apache2
apache2ctl -S will show the vhost config files. It helps to find the vhost that causes the 404 / file not found.
