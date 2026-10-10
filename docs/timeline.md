# Investigation timeline

Dates are shown at day-level to avoid implying timestamp precision not present in the public case notes. The mobile-browser observations below were reported by the site operator.

## 2026-10-09 — symptom and initial controls

- Safari on multiple iPhones was reported to hang or fail over cellular data without a VPN.
- The same site was reported to work over Wi-Fi and with a VPN. Android over cellular was reported to work. These observations narrowed the conditions but did not isolate a cause.
- An iPhone shell client completed one HTTPS request and received HTTP 200; a later compressed request timed out during TLS setup. Browser tests also showed that an HTTP 200 response did not always lead to a rendered page.
- The team tried a second server, another hostname, simplified static responses, JavaScript and gzip variations, and browser/cache variations. Results were mixed and not reproducible enough to identify a single layer.
- A packet capture was attempted but did not reveal a conclusive root cause.

## 2026-10-10 — TLS review and protocol comparison

- Read-only review found valid certificates and working TLS 1.2/1.3 handshakes. The production vhost negotiated HTTP/1.1. The shared TLS session cache was present; no clear cache misconfiguration was identified.
- A separate diagnostic vhost served the same static site content from the same server. HTTP/2 was enabled only on that vhost. It used a different hostname and certificate, so it was a useful signal but not a perfect control.
- The operator reported that the HTTP/2 canary opened immediately on the affected iPhone over cellular data without VPN.
- After explicit approval, HTTP/2 was enabled on the production apex and www HTTPS vhosts. TLS policy, application code, DNS, and site files were left unchanged.
- The operator then reported that the production URL opened immediately on the same iPhone and cellular connection. Server-side checks confirmed HTTP/2 200, valid TLS, gzip, and continued HTTP/1.1 fallback.

## Status at publication

The same-production-host before/after observation strongly implicates the protocol change or an interaction with the previous HTTP/1.1 path. The change also involved a graceful Nginx reload, so the result cannot fully isolate the effect of the negotiated protocol from a change in connection state. It does not reveal the underlying mechanism. Less than 24 hours of post-change observation had elapsed when this case was written; stability follow-up remained pending.
