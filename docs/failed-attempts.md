# Failed and inconclusive attempts

These experiments helped narrow the problem, but most were not clean, repeatable controls. Results below are observations from this incident, not universal conclusions.

| Experiment | Observation | What it did and did not show |
|---|---|---|
| Copy on a second server | The copied site also hung in the reported iPhone test. | A simple move to another host did not resolve the symptom. It did not isolate server software, network path, or timing. |
| Another hostname on the original server | It opened at first, then later failed. | A different host name alone was not a stable fix. |
| Minimal HTML or no JavaScript | Some minimal responses opened, but results varied; disabling JavaScript on the iPhone did not solve the full-site problem. | Application rendering was not a sufficient explanation for every failure. |
| Gzip disabled | Some attempts opened, others did not. | No repeatable gzip-only cause was established. |
| Static HTML, CSS, and images | A reduced page could open in some trials. | Smaller/simple responses were useful controls, but did not reliably reproduce the full-site failure. |
| Main JavaScript chunks | Eleven checked files loaded successfully in one test. | A single successful asset sweep did not prove that every page load was healthy. |
| Next.js prefetch disabled | The change was deployed, but the reported production symptom remained. | This specific application-level change was not a fix. |
| Browser and private-mode variations | The problem was reported across Safari and other browsers, including private browsing. | Browser cache alone was not a sufficient explanation. |
| TLS, certificates, and session cache | Certificate validation and TLS 1.2/1.3 checks succeeded; the shared session cache was present. | No obvious TLS configuration fault was found. Successful command-line handshakes did not reproduce every iPhone failure. |
| Packet capture | A capture was made, but no decisive cause was identified. | Capturing traffic is useful only when correlated to a precise, reproducible attempt and interpreted at the correct layer. |
| Command-line HTTP request | One request returned HTTP 200; another attempt timed out during TLS setup. | The incident had more than one observed failure mode. HTTP 200 alone did not prove browser rendering or asset completion. |

## Lessons from the failed attempts

- Do not equate a successful HTTP status with a fully rendered page.
- Do not infer a single root cause from intermittent tests with changing network, cache, hostname, or device conditions.
- Keep failures before HTTP separate from failures after receiving HTML.
- A fix to one JavaScript or compression path should not be presented as a network fix without a repeatable comparison.
- The useful discriminator was a protocol change followed by a successful canary test, then by immediate success on the same production host after enabling HTTP/2.

The HTTP/2 result is stronger than the earlier attempts, but it still does not reveal the precise mechanism.
