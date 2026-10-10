# Diagnostic approach

The key lesson was to separate the network and application layers instead of treating every failed page load as a JavaScript bug.

## Identify the failure stage

1. **DNS:** did the device resolve the intended host?
2. **TCP:** was a connection established to the server?
3. **TLS:** did the handshake finish, and was the certificate accepted?
4. **HTTP:** did the server receive a request and return a status and body?
5. **Browser rendering:** did the document and all required resources load and execute?

These stages are not interchangeable. A timeout during TLS cannot be caused by React rendering. Conversely, an HTTP 200 does not prove that JavaScript, CSS, images, or the final page rendered successfully.

## Make comparisons useful

- Change one variable at a time.
- Prefer a paired comparison on the same device, same cellular connection, same URL, and close in time.
- Record whether VPN is off, whether the device is on cellular or Wi-Fi, the negotiated HTTP version, status, and rough time-to-load.
- Treat an alternate hostname or certificate as a confounder, even when it points to the same server and content.
- Use HTTP/2 and HTTP/1.1 command-line requests as protocol checks, not as substitutes for an iPhone browser test.
- Correlate a server-side packet capture with an exact test time. Minimize capture duration and redact client identifiers before sharing.

## Useful checks

Check that the installed curl has HTTP/2 support with `curl -V`, then compare the protocol paths:

```sh
curl --http2 --compressed -sS -o /dev/null \
  -w 'status=%{http_code} version=%{http_version} tls_verify=%{ssl_verify_result} total=%{time_total}s\n' \
  https://polishelp.ru/

curl --http1.1 --compressed -sS -o /dev/null \
  -w 'status=%{http_code} version=%{http_version} tls_verify=%{ssl_verify_result} total=%{time_total}s\n' \
  https://polishelp.ru/
```

A command-line success from a server or computer on another route does not prove that an iPhone on cellular can reach the same service. The user-visible browser result remains important.

## Avoid

- Repeatedly changing JavaScript, compression, TLS ciphers, and server settings at the same time.
- Weakening TLS or enabling obsolete protocols as a diagnostic shortcut.
- Publishing raw packet captures, client IPs, cookies, authorization headers, or credentials.
- Declaring the root cause before a result is repeatable and competing explanations are considered.
