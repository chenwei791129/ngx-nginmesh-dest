# NGINX Destination IP recovery module for stream and http

This dynamic module recovers original IP address and port number of the destination packet.
It is used by nginmesh sidecar where all outgoing traffic is redirect to a single port using iptable mechanism

```
iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 80 -j REDIRECT --to-ports 80
iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 443 -j REDIRECT --to-ports 443
```

## Dependencies

This module uses Linux **getsockopt** socket API.  

## Compatibility

* nginx 1.18.0
* openresty 1.19.3.1

## Synopsis

```nginx

load_module "modules/ngx_http_nginmesh_dest_module.so";

http {
    server {
        #  use iptable to capture all outgoing traffic.  see Istio design document
        listen 80;

        # turn on module for this server
        # original IP destination and port is set to variable $nginmesh_dest
        # ex: 10.31.242.228:80
        nginmesh_dest on;

        # variable can be used in valid config directive
        proxy_pass $nginmesh_dest;
    }
}

```

## Embedded Variables

The following embedded variables are provided:

* **nginmesh_dest**
  * Original IP address and port in the format:  &lt;address&gt;:&lt;port&gt;

## Directives

### nginmesh_dest

| -   | - |
| --- | --- |
| **Syntax**  | **nginmesh_dest** \<on\|off\> |
| **Default** | off |
| **Context** | stream, http, server |

`Description:` Enables or disables the nginmesh_dest module


## Installation

1. Clone the git repository

  ```bash
foo@bar:~$ git clone https://github.com/chenwei791129/ngx-nginmesh-dest.git
  ```

2. Build the dynamic module

  ```bash
foo@bar:~$ curl -L "https://openresty.org/download/openresty-1.19.3.1.tar.gz" -o openresty.tar.gz \
&& tar -xzvf openresty.tar.gz \
&& rm -f openresty.tar.gz \
&& cd openresty-1.19.3.1
foo@bar:~$ ./configure --prefix=/usr/local/openresty/nginx --with-cc-opt='-O2 -DNGX_LUA_ABORT_AT_PANIC -I/usr/local/openresty/zlib/include -I/usr/local/openresty/pcre/include -I/usr/local/openresty/openssl111/include' --with-ld-opt='-Wl,-rpath,/usr/local/openresty/luajit/lib -L/usr/local/openresty/zlib/lib -L/usr/local/openresty/pcre/lib -L/usr/local/openresty/openssl111/lib -Wl,-rpath,/usr/local/openresty/zlib/lib:/usr/local/openresty/pcre/lib:/usr/local/openresty/openssl111/lib' --with-pcre-jit --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module --with-http_v2_module --without-mail_pop3_module --without-mail_imap_module --without-mail_smtp_module --with-http_stub_status_module --with-http_realip_module --with-http_addition_module --with-http_auth_request_module --with-http_secure_link_module --with-http_random_index_module --with-http_gzip_static_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-threads --with-stream --with-debug --with-http_ssl_module --add-dynamic-module=../ngx-stream-nginmesh-dest/module
foo@bar:~$ make && make install
  ```
