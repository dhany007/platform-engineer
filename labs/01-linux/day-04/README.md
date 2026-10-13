# Day 4 — DNS, HTTP, and TLS

**Time:** About 60–90 minutes for the core exercises. The extensions are additional self-paced sessions.

In the command blocks below, `$` is a prompt marker, not part of the command. Type the text after `$` and press Enter once per line.

Command names, abbreviations, and option meanings are also collected in the [glossary](../../../notes/glossary.md).

## Progress

- [x] Exercise 1 — Inspect DNS records.
- [x] Exercise 2 — Read an HTTP exchange over HTTPS.
- [x] Exercise 3 — Describe the request path from your observations.
- [x] Check your understanding — Answer the three questions below.
- [x] Core Day 4 complete — Exercises 1–3 and answers reviewed.
- [x] Extension 4 — Measure request timing milestones.
- [x] Extension 5 — Test an observed address with the hostname preserved.
- [x] Extension checks — Practice and review both new question sets.
- [x] Expanded Day 4 complete — Core work and both extensions reviewed.

## Why this matters

DNS means Domain Name System, HTTP means Hypertext Transfer Protocol, TLS means Transport Layer Security, and HTTPS means Hypertext Transfer Protocol Secure (HTTP protected by TLS). A request to a hostname involves several steps: name resolution, a network connection, TLS for HTTPS, and an HTTP exchange. Knowing which step failed helps you choose the next diagnostic command. Today uses public read-only requests and requires internet access; no local Go API (Application Programming Interface) is needed.

## Learning goals

1. Explain DNS records and why a hostname can have multiple addresses.
2. Identify an HTTP method, path, status code, headers, and body.
3. Explain TLS encryption, integrity, and certificate-based server authentication.

## Reading

Read [Cloudflare: What is DNS?](https://www.cloudflare.com/learning/dns/what-is-dns/), [MDN: Overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview), and [Cloudflare: Transport Layer Security](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/). Focus on DNS resolution, HTTP message structure, and the TLS handshake. These are written assignments; no video is required.

## Exercise 1 — Inspect DNS records

- `dig` is a DNS lookup tool that queries DNS through your configured resolver. `example.com` is the hostname being queried.
- `A` (address) records contain IPv4 (Internet Protocol version 4) addresses; `AAAA` records contain IPv6 (Internet Protocol version 6) addresses. `AAAA` is a record-type name, not an abbreviation to expand: IPv6 addresses have four times as many bits as IPv4 addresses (128 versus 32).
- `+noall` hides the usual output sections; `+answer` re-enables the answer section. Each answer shows a name, TTL (time to live: cache lifetime in seconds), class, record type, and record value. `IN` means the Internet class.

```console
$ dig example.com A +noall +answer
$ dig example.com AAAA +noall +answer
```

Record one address, its type, and TTL. Addresses and TTLs can change, so there is no fixed expected value. Multiple answers are normal. An empty answer is not by itself proof that all DNS is broken; inspect the full response next.

Before the optional diagnostic command: plain `dig example.com A` includes status, question, answer, and resolver details. Look for `status` and the `SERVER` line. In the response flags, `qr` marks a response (query/response), `rd` means recursion desired, and `ra` means recursion available. A timeout suggests your query received no usable reply.

```console
$ dig example.com A
```

If DNS or internet access is unavailable, record that observation and finish the written model; repeat the network observations when access returns.

### Expected result

These DNS addresses are documentation-only sample values, not claimed current records for example.com. Real addresses, TTLs, resolver addresses, identifiers (IDs), timings, and record counts vary. The final command includes the optional full-response check; bracketed notes mark omitted sections. DNS answers establish name resolution, not service health.

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

- `curl` is a URL (Uniform Resource Locator) transfer tool; its name has several documented wordplays. It uses the HTTP GET method (retrieve a resource) by default and makes a request to `https://example.com/`. `https://` enables TLS and uses port 443 by default. `/` is the resource path.
- `-v` is short for `--verbose`: it prints diagnostic details, including the connection, TLS information when available, and request/response headers. Lines marked `>` describe the request; lines marked `<` describe the response. See the [curl verbose reference](https://curl.se/docs/manpage.html#-v).
- `-i` corresponds to `--include`, a supported alias: it includes response headers in normal output before the body, while the request remains GET. The current [curl header-output reference](https://curl.se/docs/manpage.html#-i) names it `--show-headers`; older versions may only support the long name `--include`.
- `--connect-timeout 5` limits connection establishment to five seconds; `--max-time 15` caps the entire transfer at fifteen seconds.
- `-I` is short for `--head`: it uses the HTTP HEAD method, so the second request returns headers without a response body. See the [curl head reference](https://curl.se/docs/manpage.html#-I). Options are case-sensitive: lowercase `-i` includes headers with the GET body; uppercase `-I` makes a HEAD request.

The response body can contain HTML (Hypertext Markup Language), the markup used for web pages. Curl may label TLS diagnostics as SSL (Secure Sockets Layer), the older protocol name; the reported TLS version identifies the protocol actually used.

Combining `-v -i` displays response headers twice: `-v` writes diagnostic headers (prefixed with `<`) to standard error (stderr), and `-i` includes headers with the response body on standard output (stdout). These are two displays of the same response, not two requests.

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
2. The client connects to an address and the chosen port. HTTPS commonly uses TCP (Transmission Control Protocol) followed by TLS; newer HTTP/3 uses QUIC (a protocol name, not a current acronym) over UDP (User Datagram Protocol), so TCP is not universal across all HTTP versions.
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

## Core completion criteria

- [x] You identified a DNS record's type, address, and TTL.
- [x] You located the method, path, status, headers, and body in an HTTP exchange.
- [x] You explained how TLS fits between connection establishment and HTTP.
- [x] You wrote a request-path explanation supported by your observations.

## Extension 4 — Measure request timing milestones

**Context:** A request can succeed yet feel slow. Measure where elapsed time accumulates before deciding which part of the request path to investigate. Read the [curl write-out reference](https://curl.se/docs/manpage.html#-w).

- `--silent --show-error` hides the progress meter while retaining error messages. `--output /dev/null` discards the downloaded body; curl still transfers it.
- `--connect-timeout 5` bounds connection establishment and `--max-time 15` bounds the entire operation, in seconds.
- `--write-out` prints the quoted format after the transfer. Curl substitutes `%{...}` variables; `\n` adds a newline. Single quotes preserve this format for curl.
- The leading `$` below is only a prompt marker. Type the complete command on one line and press Enter once.

```console
$ curl --silent --show-error --output /dev/null --connect-timeout 5 --max-time 15 --write-out 'lookup=%{time_namelookup}\nconnect=%{time_connect}\ntls=%{time_appconnect}\nfirst_byte=%{time_starttransfer}\ntotal=%{time_total}\n' https://example.com/
```

For a successful direct HTTPS connection, these values are cumulative milestones measured in seconds from the start:

| Field | Milestone |
| --- | --- |
| `time_namelookup` | Name lookup finished. |
| `time_connect` | Connection to the remote host or proxy established. |
| `time_appconnect` | TLS handshake finished. |
| `time_starttransfer` | First response byte arrived, including earlier setup and server processing. |
| `time_total` | Transfer finished. |

Do not add the five values together. For this simple direct connection, subtract adjacent milestones to estimate intervals: `time_appconnect - time_connect` estimates TLS setup, and `time_total - time_starttransfer` estimates the remaining transfer. Time to first byte also reflects network travel and server work, so it does not isolate application processing.

### Expected result

Illustrative seconds only; network load, DNS caches, server behavior, proxies, and curl's protocol choice change the numbers. A proxy can introduce its own lookup, connection, and tunnel setup. Failed requests may leave later milestones zero or incomplete; record errors before interpreting them.

```console
$ curl --silent --show-error --output /dev/null --connect-timeout 5 --max-time 15 --write-out 'lookup=%{time_namelookup}\nconnect=%{time_connect}\ntls=%{time_appconnect}\nfirst_byte=%{time_starttransfer}\ntotal=%{time_total}\n' https://example.com/
lookup=0.012000
connect=0.045000
tls=0.091000
first_byte=0.140000
total=0.142000
```

### Check your understanding

1. In the sample, what is the approximate TLS setup interval?
2. Why would adding all five numbers overstate the request duration?
3. Does a larger first-byte time prove the application itself became slower?

### Answer key

1. `0.091 - 0.045 = 0.046` seconds for this direct connection.
2. Earlier intervals are already included in later cumulative milestones; total duration is `0.142` seconds.
3. No. Lookup, connection/TLS setup, network travel, proxies, and server processing can all contribute.

**Cleanup and safe rerun:** No files or services are created. Repeat the bounded command a few times and retain the observations in your notes; treat variation as evidence, not a guaranteed performance threshold.

## Extension 5 — Connect to an observed address while preserving HTTPS identity

**Context:** Test one DNS answer while still asking for the same HTTPS site. Read [everything curl: Name resolve tricks](https://everything.curl.dev/usingcurl/connections/name.html).

- `dig example.com A +short` asks for concise IPv4 answers. `awk` is a pattern/action text-processing tool named after its authors Aho, Weinberger, and Kernighan. Here its pattern selects the first line shaped like an IPv4 address, and its actions print that line and exit; any alias lines are skipped.
- `example_address=$(...)` assigns that observed address to a shell variable; type the literal `$()` so the shell runs the enclosed pipeline. `printf` (print formatted) displays the selection. If it is blank, investigate DNS before continuing.
- `--resolve` supplies curl's hostname-and-port mapping for this invocation. Double quotes expand the variable while keeping the mapping one argument. The literal `${example_address:?No IPv4 A answer found}` expands to its value, or stops the command with an error if it is empty or unset. The URL remains `https://example.com/`, so the HTTP hostname, TLS SNI (Server Name Indication) name, and certificate hostname validation remain `example.com`.
- `-v` shows connection and TLS diagnostics; `--output /dev/null` discards the body. Connection and total timeouts remain five and fifteen seconds. Certificate verification remains enabled.
- `$` at the beginning is a prompt marker. Enter one command per line. Type the internal `$` characters in `$(...)`, `"$example_address"`, and `${example_address:?...}` because they are shell syntax.

```console
$ example_address=$(dig example.com A +short | awk '/^[0-9]+[.][0-9]+[.][0-9]+[.][0-9]+$/ {print; exit}')
$ printf 'Selected address: %s\n' "$example_address"
$ curl -v --output /dev/null --connect-timeout 5 --max-time 15 --resolve "example.com:443:${example_address:?No IPv4 A answer found}" https://example.com/
```

Compare curl's attempted remote address with the selected DNS answer. A configured HTTP proxy may resolve the origin itself, so `--resolve` does not necessarily control the origin address reached through that proxy. Inspect verbose output for proxy/CONNECT details and record this limitation; do not change required proxy settings. An observed DNS address can also stop working later, so refresh the answer before retrying.

### Expected result

This transcript illustrates a bounded connection failure using an address observed during validation. The address is not a fixed answer or a value to copy: your command selects its own current DNS answer. A DNS answer does not guarantee that the selected address is reachable from your network. A successful connection instead shows connection/TLS diagnostics and an HTTP response; verify those from your own output. Bracketed notes omit output.

```console
$ example_address=$(dig example.com A +short | awk '/^[0-9]+[.][0-9]+[.][0-9]+[.][0-9]+$/ {print; exit}')
$ printf 'Selected address: %s\n' "$example_address"
Selected address: 104.20.23.154
$ curl -v --output /dev/null --connect-timeout 5 --max-time 15 --resolve "example.com:443:${example_address:?No IPv4 A answer found}" https://example.com/
[progress meter and some diagnostics omitted]
* Added example.com:443:104.20.23.154 to DNS cache
*   Trying 104.20.23.154:443...
curl: (28) Failed to connect to example.com port 443 after 5006 ms: Timeout was reached
```

### Check your understanding

1. Where does `example_address` come from, and what happens if it is empty?
2. Why keep the hostname in the URL instead of replacing it with the IP address?
3. Why might this command fail to test the chosen origin address through a proxy?

### Answer key

1. The pipeline selects an observed IPv4 `A` answer from `dig`. If the selection is empty, the shell expansion stops the curl command with an error instead of attempting a request.
2. The hostname supplies the HTTP identity and TLS SNI and is checked against the certificate; `--resolve` changes address selection for that hostname/port.
3. The proxy may perform origin resolution and connection itself. Curl's connection diagnostics must be interpreted in that context.

**Cleanup and safe rerun:** The shell variable remains in this terminal session, while the mapping is local to the curl invocation and creates no files or persistent DNS changes. Refresh DNS and repeat with a currently observed address. Record certificate or connection errors without disabling verification.

## Expanded Day 4 completion criteria

- [x] You recorded timing milestones and explained cumulative values and network/proxy variability.
- [x] You tested an observed A record with `--resolve` and explained hostname identity and proxy limitations.
- [x] You answered both extensions’ checks and reviewed their evidence.
- [x] Expanded Day 4 complete — Core criteria and all extension criteria reviewed.
