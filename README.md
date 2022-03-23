# Konfigurasi Nginx-Varnish-Apache2
## nginx ssl termination reverse proxy
```bash
server {
    listen 443 ssl http2;
    server_name moodle30d.my.id;
    ssl_certificate           /etc/letsencrypt/live/moodle30d.my.id/fullchain.pem;
    ssl_certificate_key       /etc/letsencrypt/live/moodle30d.my.id/privkey.pem;

    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_protocols TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    ssl_prefer_server_ciphers on;

    access_log            /var/log/nginx/moodle30d.my.id_access.log;
    error_log             /var/log/nginx/moodle30d.my.id_error.log;

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
    if (client.ip != "127.0.0.1" && req.http.host ~ "domain.my.id") {
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
### virtualhost apache, default aja
