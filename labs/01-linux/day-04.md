# Day 4 — DNS, HTTP, and TLS

**Time:** About 60–90 minutes.

## Progress

- [ ] Exercise 1 — Inspect DNS records.
- [ ] Exercise 2 — Read an HTTP exchange over HTTPS.
- [ ] Exercise 3 — Describe the request path from your observations.
- [ ] Check your understanding — Answer the three questions below.
- [ ] Day 4 complete — All exercises and answers reviewed.

## Why this matters

A request to a hostname involves several steps: name resolution, a network connection, TLS for HTTPS, and an HTTP exchange. Knowing which step failed helps you choose the next diagnostic command. Today uses public read-only requests and requires internet access; no local Go API is needed.

## Learning goals

1. Explain DNS records and why a hostname can have multiple addresses.
2. Identify an HTTP method, path, status code, headers, and body.
3. Explain TLS encryption, integrity, and certificate-based server authentication.

## Reading

Read [Cloudflare: What is DNS?](https://www.cloudflare.com/learning/dns/what-is-dns/), [MDN: Overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview), and [Cloudflare: Transport Layer Security](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/). Focus on DNS resolution, HTTP message structure, and the TLS handshake. These are written assignments; no video is required.

## Exercise 1 — Inspect DNS records

- `dig` queries DNS through your configured resolver. `example.com` is the hostname being queried.
- `A` asks for IPv4 addresses; `AAAA` asks for IPv6 addresses.
- `+noall` hides the usual output sections; `+answer` re-enables the answer section. Each answer shows a name, TTL (cache lifetime in seconds), class, record type, and record value.

```sh
dig example.com A +noall +answer
dig example.com AAAA +noall +answer
```

Record one address, its type, and TTL. Addresses and TTLs can change, so there is no fixed expected value. Multiple answers are normal. An empty answer is not by itself proof that all DNS is broken; inspect the full response next.

Before the optional diagnostic command: plain `dig example.com A` includes status, question, answer, and resolver details. Look for `status` and the `SERVER` line. A timeout suggests your query received no usable reply.

```sh
dig example.com A
```

If DNS or internet access is unavailable, record that observation and finish the written model; repeat the network observations when access returns.

## Exercise 2 — Read an HTTP exchange over HTTPS

- `curl` uses GET by default and makes a request to `https://example.com/`. `https://` enables TLS and uses port 443 by default. `/` is the resource path.
- `-v` prints diagnostic details, including the connection, TLS information when available, and request/response headers. Lines marked `>` describe the request; lines marked `<` describe the response.
- `-i` includes response headers in the normal output before the body.
- `--connect-timeout 5` limits connection establishment to five seconds; `--max-time 15` caps the entire transfer at fifteen seconds.
- `-I` requests headers using the HTTP HEAD method, so the second request does not download the response body.

```sh
curl -v -i --connect-timeout 5 --max-time 15 https://example.com/
curl -I --connect-timeout 5 --max-time 15 https://example.com/
```

Find the requested method and path, response status, one response header, and the GET response body. Find TLS/certificate details in the verbose output if shown by your curl build. HTTP versions, TLS versions, and certificate details can vary. Curl verifies the certificate by default; record any verification error as an observation instead of disabling verification.

If your network uses an HTTP proxy, verbose output may also show a proxy connection or CONNECT exchange. Identify that extra step in your notes. This lab does not require changing proxy settings.

## Exercise 3 — Describe the request path from your observations

No terminal commands are needed. In a personal notes file, draw or write these steps and attach evidence from Exercises 1–2:

1. DNS maps the hostname to address records. DNS does not itself check whether an HTTP service is healthy.
2. The client connects to an address and the chosen port. HTTPS commonly uses TCP followed by TLS; newer HTTP/3 uses QUIC over UDP, so TCP is not universal across all HTTP versions.
3. TLS authenticates the server using its certificate and protects data in transit against reading and modification. It does not guarantee that the application is correct or that stored data is protected.
4. The client sends an HTTP request and the server returns an HTTP response with a status, headers, and possibly a body.

Compare this to Day 3's `http://127.0.0.1:8765/`: the numeric address needs no hostname lookup, the port is explicit, and that plaintext local server does not use TLS. You do not need to restart it for this exercise.

## Check your understanding

1. What do A, AAAA, and TTL tell you, and why is a DNS answer not a health check?
2. What changed between curl's GET and HEAD requests?
3. What does TLS protect, and what does it leave for the application to handle?

**Share back:** Send DNS observations, HTTP/TLS observations, your request-path notes, and answers. Note any network-dependent exercise you need to repeat.

## Completion criteria

- [ ] You identified a DNS record's type, address, and TTL.
- [ ] You located the method, path, status, headers, and body in an HTTP exchange.
- [ ] You explained how TLS fits between connection establishment and HTTP.
- [ ] You wrote a request-path explanation supported by your observations.
