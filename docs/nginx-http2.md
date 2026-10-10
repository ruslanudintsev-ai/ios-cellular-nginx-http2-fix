# Nginx HTTP/2 configuration and rollback

This section documents the configuration change used in the case. It is not a universal prescription; inspect the installed Nginx build and the exact virtual hosts first.

## Version and module

The tested server ran Nginx 1.30.5 and was built with the HTTP/2 module. Nginx documents the `http2 on;` directive from version 1.25.1. The directive is valid in `http` and `server` contexts. See the [official module documentation](https://nginx.org/en/docs/http/ngx_http_v2_module.html).

HTTP/2 over TLS also requires ALPN support. Keep the existing TLS configuration and certificate handling; do not copy old cipher lists or weaken the minimum TLS version.

## Configuration pattern

On current Nginx versions, enable HTTP/2 inside the intended HTTPS server block:

```nginx
server {
    listen 443 ssl;
    http2 on;
    server_name example.com;

    # Keep the existing certificate, TLS policy, and site configuration.
}
```

In this case the directive was added only to the apex and www HTTPS virtual hosts. It was not added globally, and unrelated sites were not changed.

## Validate both protocols

Before and after the change, verify configuration syntax and that both protocol paths remain available:

```sh
nginx -t && nginx -s reload

curl --http2 --compressed -sS -o /dev/null \
  -w 'status=%{http_code} version=%{http_version} tls_verify=%{ssl_verify_result}\n' \
  https://example.com/

curl --http1.1 --compressed -sS -o /dev/null \
  -w 'status=%{http_code} version=%{http_version} tls_verify=%{ssl_verify_result}\n' \
  https://example.com/
```

Expected in this case: the first command reports `version=2`, while forced HTTP/1.1 reports `version=1.1`; both return successful responses. A 200 response without `version=2` does **not** prove that HTTP/2 was negotiated. Also test the actual client and network that showed the problem; command-line checks do not replace that test.

## Rollback

Remove only the `http2 on;` directives from the affected server blocks, run `nginx -t`, and perform a graceful reload. Preserve a copy of the exact pre-change vhost configuration before editing. Do not roll back unrelated shared TLS settings.

## Result and caveat

Enabling HTTP/2 on a separate diagnostic vhost was followed by an immediate successful iPhone load. Enabling it on the production vhosts was then followed by an immediate successful load on the same iPhone, production hostname, and cellular connection. HTTP/1.1 fallback continued to work.

The HTTP/2 change involved a graceful Nginx reload, which could also affect connection state. This is strong evidence associated with the change, but one immediate success does not fully separate the protocol effect from the reload or identify the low-level cause. The observation period was under 24 hours when documented.
