---
title: "An iPhone Couldn't Load Our HTTPS Site on Cellular. Enabling HTTP/2 Helped—But Here's What We Actually Know"
published: false
description: "A real Nginx incident: Wi-Fi and VPN worked, cellular iPhone requests stalled, and the effective change was HTTP/2. What failed, what worked, and what remains unknown."
tags: nginx, http2, debugging, webdev
---

On October 10, 2026, we changed one thing on a production HTTPS website: **we enabled HTTP/2 for its Nginx virtual hosts**.

The site then loaded immediately on the same affected iPhone, using cellular data with Wi-Fi and VPN disabled. It was the first genuinely promising outcome after testing JavaScript, compression, a second server, alternate hostnames, certificates, and TLS settings.

Before anyone turns this into "HTTP/1.1 is broken on iOS": **that's not what we proved**. The low-level cause remains unknown, and the change also involved reloading Nginx. What we have is a useful production before/after result and a repeatable *method of investigation*, not a universal fix.

Here's the story.

## A very selective failure

This wasn't an all-users outage. The website, a static Next.js export served by Nginx 1.30.5, looked healthy from outside monitoring.

| Client or path | Observed behavior |
| --- | --- |
| iPhone on cellular, no VPN | Long hangs or "server stopped responding" |
| Same iPhone on Wi-Fi | Site loaded |
| Same iPhone using a VPN | Site loaded |
| Android on cellular | Reported to load |
| External desktop and server checks | HTTPS usually returned HTTP 200 |

A separate site **sharing the exact same VPS and IPv4 address** could also load on the affected iPhone's mobile connection.

That last detail made a blanket "the carrier cannot reach the server" explanation less convincing. Multiple HTTPS hosts can share an address yet differ in hostname, SNI, certificate, or virtual-host behavior. Still, our TLS configuration review did not identify a relevant misconfiguration.

## Why 200 OK kept misleading us

At times an iPhone command-line client fetched the complete HTML quickly using TLS 1.3. Another attempt, made shortly afterward with `curl --compressed`, timed out during TLS setup—**before any HTTP request**.

At other times, server logs contained HTTP 200 responses for the main document and some Next.js resources, while Safari still failed to display the page.

Those observations aren't mutually exclusive: an intermittent incident can fail at different stages. But they prevent several tempting shortcuts:

- `curl` receiving HTML doesn't mean Safari received and rendered every subresource.
- An origin access-log entry with `200` doesn't prove that the phone received the entire response.
- A request missing from an access log doesn't tell you whether DNS, TCP, TLS, a client timeout, or an intermediary stopped it.
- A carrier's shared public IPv4 (CGNAT) is not a reliable per-device identifier for packet captures.

We learned to keep **DNS → TCP → TLS → HTTP → asset loading → rendering** separate.

## The fixes that weren't fixes

### Moving the site

We built an isolated mirror on a different public IPv4. A tiny HTTPS page opened; the fuller mirrored homepage could still hang. A fresh IP was not enough.

We also used another hostname on the original VPS with the same static site content. Its first opening looked promising, but subsequent attempts stalled. A one-off successful canary run was not a durable workaround.

### Blaming JavaScript

We tested simplified pages, combinations of HTML/CSS/images, and reduced JavaScript. Disabling JavaScript on the phone did **not** make the original site load reliably.

A minimal same-origin JavaScript probe executed successfully. One carefully instrumented test loaded all **11 core Next.js homepage scripts**. A separate artificial test with many concurrent chunks stalled, but that stress pattern did not represent normal page navigation.

We even reduced background Next.js navigation prefetch on the homepage, released the change, and verified that it really reached production. **The cellular failure remained.**

### Blaming gzip

One uncompressed test page opened while a compressed one failed, which looked like a clear clue—until a compressed `curl` request succeeded on the same mobile path, and another purported "gzip failure" hadn't even reached HTTP.

Compression was worth checking, but **the experiments didn't establish gzip as the cause**.

### Blaming certificates or the TLS session cache

Nginx had valid certificates, working TLS 1.2/1.3 from external test locations, and a shared TLS session cache. There was no evidence to justify copying an old Stack Overflow workaround that changed several TLS parameters and obsolete cipher suites.

## The one major transport variable we hadn't tested

Our production HTTPS virtual hosts were still serving **HTTP/1.1**, even though the Nginx build supported HTTP/2.

The hypothesis was simple: HTTP/2's multiplexing and connection management *might* change the interaction between this specific iPhone/mobile-network path and a page with many resources. We did **not** know whether that was the actual mechanism.

We first enabled HTTP/2 on a diagnostic virtual host. It served the same static content on the original server; gzip behavior had been aligned beforehand. The test still had confounders—different hostname, SNI, and certificate.

The site operator reported that this HTTP/2 canary opened immediately on the affected iPhone using cellular data without a VPN.

After explicit approval, we enabled HTTP/2 only on the **existing production apex and www HTTPS virtual hosts**. We did not change DNS, certificate, application files, gzip policy, or TLS cipher configuration.

The result: **the same production URL opened immediately on the same affected iPhone and mobile connection**.

Server-side checks confirmed an HTTP/2 200 response. Forced HTTP/1.1 still worked, preserving compatibility with older clients.

## The Nginx configuration

For Nginx **1.25.1 or later**, when built with `ngx_http_v2_module`, the relevant pattern is:

```nginx
server {
    listen 443 ssl;
    http2 on;
    server_name example.com;

    # Keep your real certificate, TLS policy, and
    # existing application/static-site configuration.
}
```

This example is *not* a recommendation to overwrite an existing vhost. Back up the exact configuration first, use the intended HTTPS blocks only, and check that your installed Nginx supports the directive. If an admin panel manages Nginx config, follow that panel's persistence and change-control rules.

For an approved and reversible change:

```sh
nginx -t && nginx -s reload
```

Then verify the **negotiated protocol**, not merely the status:

```sh
# curl must support HTTP/2 (check with: curl -V)
curl --http2 --compressed -sS -o /dev/null \
  -w 'status=%{http_code} version=%{http_version}\n' \
  https://example.com/

curl --http1.1 --compressed -sS -o /dev/null \
  -w 'status=%{http_code} version=%{http_version}\n' \
  https://example.com/
```

In this configuration, the first request should report `version=2`, and the forced fallback should report `version=1.1`. Both should succeed.

To roll back, restore the prior saved vhost definition, run `nginx -t`, and gracefully reload. Don't weaken TLS settings or change unrelated domains to chase an unverified theory.

The directive and version requirement are documented in the [official Nginx HTTP/2 module documentation](https://nginx.org/en/docs/http/ngx_http_v2_module.html).

## What the result does—and doesn't—tell us

**We observed:**

- The affected phone repeatedly struggled with the original production HTTPS URL over cellular without VPN.
- An HTTP/2 diagnostic vhost opened promptly.
- Enabling HTTP/2 on the *same original production hostname and certificate* was followed by an immediate successful load.
- HTTP/1.1 remained available.

**We did not establish:**

- A general HTTP/1.1 bug, or a specific Safari, iOS, Nginx, carrier, DPI, MTU, or TLS implementation defect.
- That every iPhone and carrier would now work.
- Long-term stability: fewer than 24 hours had passed when the case was first documented.

There's another important confounder: enabling HTTP/2 also required a **graceful Nginx reload**. That reload might have affected existing connection state. Without a carefully controlled follow-up, we cannot separate its contribution completely from the protocol change.

It's nevertheless a stronger signal than an earlier canary success because the production hostname, IP, and certificate stayed the same.

## The debugging lesson

When a site fails on one class of device and network, don't jump straight to rewriting the frontend.

Check what protocol is **actually negotiated** at the failing HTTPS entry point. Compare the affected device on cellular, Wi-Fi, and VPN; distinguish failure before HTTP from failure during resource loading; and keep every test reversible.

Most importantly, **write down the negative results and the limitations of each experiment**. They keep the next engineer from spending days repeating an investigation that already happened.

We published the full five-language case notes, investigation timeline, inconclusive experiments, and rollback instructions in an [open GitHub case study](https://github.com/ruslanudintsev-ai/ios-cellular-nginx-http2-fix).

*Disclosure: This article was drafted with AI assistance from a real incident log and technical findings. A human operator reported the iPhone results; the underlying transport root cause remains unverified.*
