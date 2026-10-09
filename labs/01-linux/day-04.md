# Day 4 — DNS, HTTP, and TLS

**Time:** About 60–90 minutes.

In the command blocks below, `$` is a prompt marker, not part of the command. Type the text after `$` and press Enter once per line.

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

```console
$ dig example.com A +noall +answer
$ dig example.com AAAA +noall +answer
```

Record one address, its type, and TTL. Addresses and TTLs can change, so there is no fixed expected value. Multiple answers are normal. An empty answer is not by itself proof that all DNS is broken; inspect the full response next.

Before the optional diagnostic command: plain `dig example.com A` includes status, question, answer, and resolver details. Look for `status` and the `SERVER` line. A timeout suggests your query received no usable reply.

```console
$ dig example.com A
```

If DNS or internet access is unavailable, record that observation and finish the written model; repeat the network observations when access returns.

### Expected result

These DNS addresses are documentation-only sample values, not claimed current records for example.com. Real addresses, TTLs, resolver addresses, IDs, timings, and record counts vary. The final command includes the optional full-response check; bracketed notes mark omitted sections. DNS answers establish name resolution, not service health.

```console
$ dig example.com A +noall +answer
example.com. 300 IN A 192.0.2.10
$ dig example.com AAAA +noall +answer
example.com. 300 IN AAAA 2001:db8::10
$ dig example.com A
[dig version banner omitted]
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42100
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
[other sections omitted]
;; QUESTION SECTION:
;example.com. IN A

;; ANSWER SECTION:
example.com. 300 IN A 192.0.2.10

;; Query time: 12 msec
;; SERVER: 192.0.2.53#53(192.0.2.53) (UDP)
;; WHEN: Fri Oct 09 10:00:00 WIB 2026
[message size omitted]
```

## Exercise 2 — Read an HTTP exchange over HTTPS

- `curl` uses GET by default and makes a request to `https://example.com/`. `https://` enables TLS and uses port 443 by default. `/` is the resource path.
- `-v` prints diagnostic details, including the connection, TLS information when available, and request/response headers. Lines marked `>` describe the request; lines marked `<` describe the response.
- `-i` includes response headers in the normal output before the body.
- `--connect-timeout 5` limits connection establishment to five seconds; `--max-time 15` caps the entire transfer at fifteen seconds.
- `-I` requests headers using the HTTP HEAD method, so the second request does not download the response body.

```console
$ curl -v -i --connect-timeout 5 --max-time 15 https://example.com/
$ curl -I --connect-timeout 5 --max-time 15 https://example.com/
```

Find the requested method and path, response status, one response header, and the GET response body. Find TLS/certificate details in the verbose output if shown by your curl build. HTTP versions, TLS versions, and certificate details can vary. Curl verifies the certificate by default; record any verification error as an observation instead of disabling verification.

If your network uses an HTTP proxy, verbose output may also show a proxy connection or CONNECT exchange. Identify that extra step in your notes. This lab does not require changing proxy settings.

### Expected result

This shows an illustrative direct HTTP/1.1 exchange. The documentation-only address, TLS details, headers, curl version, and diagnostics vary. Bracketed notes mark omitted output. `-v` prints response headers as diagnostics and `-i` also includes them with the body, so they can appear twice. The second command uses HEAD and returns no body. Proxies or HTTP/2 can change the displayed exchange.

```console
$ curl -v -i --connect-timeout 5 --max-time 15 https://example.com/
[progress meter and some diagnostics omitted]
* Host example.com:443 was resolved.
*   Trying 192.0.2.10:443...
* Connected to example.com (192.0.2.10) port 443
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* SSL certificate verify ok.
> GET / HTTP/1.1
> Host: example.com
> User-Agent: curl/8.7.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Content-Type: text/html
< [other response headers omitted]
<
HTTP/1.1 200 OK
Content-Type: text/html
[other response headers omitted]

<!doctype html>
<html><head><title>Example Domain</title></head>
[remaining HTML omitted]
$ curl -I --connect-timeout 5 --max-time 15 https://example.com/
HTTP/1.1 200 OK
Content-Type: text/html
[other response headers omitted]
```

## Exercise 3 — Describe the request path from your observations

No terminal commands are needed. In a personal notes file, draw or write these steps and attach evidence from Exercises 1–2:

1. DNS maps the hostname to address records. DNS does not itself check whether an HTTP service is healthy.
2. The client connects to an address and the chosen port. HTTPS commonly uses TCP followed by TLS; newer HTTP/3 uses QUIC over UDP, so TCP is not universal across all HTTP versions.
3. TLS authenticates the server using its certificate and protects data in transit against reading and modification. It does not guarantee that the application is correct or that stored data is protected.
4. The client sends an HTTP request and the server returns an HTTP response with a status, headers, and possibly a body.

Compare this to Day 3's `http://127.0.0.1:8765/`: the numeric address needs no hostname lookup, the port is explicit, and that plaintext local server does not use TLS. You do not need to restart it for this exercise.

### Expected result

**Sample written response (no terminal commands):**

“DNS returned an address for `example.com`. In the sample HTTPS exchange, curl connected to that address on port 443, validated the server certificate during TLS setup, and sent `GET /`. The server returned HTTP 200, headers, and an HTML body. DNS resolution alone would not prove that the server was healthy.

“For Day 3's `http://127.0.0.1:8765/`, the numeric address needs no hostname lookup, the port is explicit, and plaintext HTTP does not use TLS. HTTP/3 or a configured proxy can introduce a different connection path.”

Attach your own DNS and HTTP/TLS evidence when doing the exercise; this written example is illustrative.

## Check your understanding

1. What do A, AAAA, and TTL tell you, and why is a DNS answer not a health check?
2. What changed between curl's GET and HEAD requests?
3. What does TLS protect, and what does it leave for the application to handle?

### Answer key

1. A records contain IPv4 addresses, AAAA records contain IPv6 addresses, and TTL limits caching time. DNS can return an address even when its service is unavailable.
2. GET requests the resource and receives its body. HEAD requests the corresponding response headers without a response body; it does not change the resource.
3. TLS protects data in transit against reading and alteration and authenticates the server through certificate validation. Application correctness, authorization, and protection of stored data still need application controls.

**Share back:** Send DNS observations, HTTP/TLS observations, your request-path notes, and answers. Note any network-dependent exercise you need to repeat.

## Completion criteria

- [ ] You identified a DNS record's type, address, and TTL.
- [ ] You located the method, path, status, headers, and body in an HTTP exchange.
- [ ] You explained how TLS fits between connection establishment and HTTP.
- [ ] You wrote a request-path explanation supported by your observations.
