# iPhone Cannot Open an HTTPS Site on Cellular Data? A Real Nginx HTTP/2 Case Study

**Languages:** [English](README.md) · [Русский](README.ru.md) · [Español](README.es.md) · [Português (Brasil)](README.pt-BR.md) · [简体中文](README.zh-CN.md)

**Case status:** enabling HTTP/2 on the production HTTPS virtual hosts was followed by an immediate successful load on the same iPhone over cellular data, with VPN off. The observation was recorded on 2026-10-10; less than 24 hours had elapsed when this report was prepared, so long-term stability had not yet been established.

This case concerns [polishelp.ru](https://polishelp.ru/). It is a field report based on operator observations, not a controlled laboratory study.

## Short version

During the investigation, Safari on iPhones intermittently failed to display the site over mobile data without a VPN. Wi-Fi and VPN access worked, Android access was reported to work, and other checks produced mixed results. Some attempts stopped before an HTTP request; other attempts returned HTML with HTTP 200 while the page still did not render correctly.

The investigation tested alternate hosts, static responses, JavaScript, gzip, Next.js prefetch, TLS configuration, and packet capture. These did not produce a reliable fix. Nginx was serving HTTP/1.1 on the affected production host.

A diagnostic vhost was then enabled for HTTP/2. The operator reported that it opened immediately on the affected iPhone over cellular data. After that signal, HTTP/2 was enabled on the production apex and www HTTPS vhosts. The operator reported that the same production URL then opened immediately on the same iPhone and mobile connection. HTTP/1.1 fallback remained available.

## What the evidence says

The before-and-after result strongly implicates the HTTP/2 change or an interaction with the previous HTTP/1.1 path. The production verification used the same hostname and certificate before and after the change, which is stronger evidence than the canary comparison.

It does **not** prove a specific Safari, Nginx, TLS, carrier, or HTTP/1.1 defect. The mobile tests were operator-reported and were not instrumented with a timestamped packet capture. Network conditions and browser state can change between attempts. Enabling HTTP/2 also required a graceful Nginx reload, which may have affected connection state. A single successful load does not isolate the protocol switch from the reload. The underlying mechanism remains unknown.

No TLS security settings, cipher policy, application code, DNS, or site content were changed as part of the production HTTP/2 switch. The change was limited to the relevant HTTPS virtual hosts.

## Contents

- [Timeline](docs/timeline.md)
- [Failed and inconclusive attempts](docs/failed-attempts.md)
- [Diagnostic approach](docs/diagnostics.md)
- [Nginx HTTP/2 configuration and rollback](docs/nginx-http2.md)

## Important limitation

This repository records an early successful outcome, not a guarantee that HTTP/2 is a universal fix for iPhone cellular failures. A site returning HTTP 200 does not prove that all assets loaded or that a browser rendered the page. Likewise, a successful test on one phone and route does not establish behavior for every iPhone, carrier, or region.

## Privacy

This report intentionally omits server IP addresses, internal filesystem paths, credentials, account identifiers, raw access logs, and personal network data. The internal incident tracker is not linked. Please do not add packet captures, client IP addresses, credentials, or private infrastructure details to public issues.

## References

- [Official Nginx HTTP/2 module documentation](https://nginx.org/en/docs/http/ngx_http_v2_module.html) — the `http2 on;` directive is available from Nginx 1.25.1 and requires the HTTP/2 module.
- [Official Nginx HTTPS configuration guide](https://nginx.org/en/docs/http/configuring_https_servers.html)

## License

Documentation is licensed under [CC BY 4.0](LICENSE).
