# Day 3 — Ports and TCP Connections

**Time:** About 60–90 minutes.

In the command blocks below, `$` is a prompt marker, not part of the command. Type the text after `$` and press Enter once per line, using Terminal A or B as instructed.

## Progress

- [x] Exercise 1 — Start a local HTTP listener.
- [x] Exercise 2 — Inspect the listener and make requests.
- [x] Exercise 3 — Stop the listener and observe a connection failure.
- [x] Check your understanding — Answer the three questions below.
- [x] Day 3 complete — All exercises and answers reviewed.

## Why this matters

A running process does not necessarily accept network traffic. A client needs an address and a port where a process is listening. Testing the listener separately from the HTTP response helps distinguish a connection failure from an application error.

## Learning goals

1. Explain loopback addresses, ports, and a TCP listener.
2. Connect a listening socket to its process with `lsof`.
3. Distinguish HTTP 404 from failure to establish a connection.

## Reading

Read [Cloudflare: What is TCP/IP?](https://www.cloudflare.com/learning/ddos/glossary/tcp-ip/). In the [curl manual](https://curl.se/docs/manpage.html), read `--include`, `--verbose`, `--connect-timeout`, and `--max-time`. TCP provides an ordered byte stream; HTTP gives those bytes request and response meaning.

## Exercise 1 — Start a local HTTP listener

Open two terminal tabs, called **Terminal A** and **Terminal B**. Run this block in **Terminal A**:

- `cd` enters the project root; `mkdir -p` creates an isolated workspace and missing parents.
- `printf` produces a line ending in `\n`; `>` creates or overwrites the lab's `index.html`.
- `python3 -m http.server` runs Python's built-in HTTP server module. `8765` is the TCP port.
- `--bind 127.0.0.1` accepts connections only through this computer's IPv4 loopback address.
- `--directory` serves only the specified lab workspace. This is a static file server, not the future Go API.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-03/workspace
$ printf 'Hello from Day 3\n' > labs/01-linux/day-03/workspace/index.html
$ python3 -m http.server 8765 --bind 127.0.0.1 --directory labs/01-linux/day-03/workspace
```

Leave Terminal A running while you use Terminal B. Expect a serving message. If you see “Address already in use,” stop this attempt; do not stop an unknown process. Pick an unused port such as `8766` and substitute it in every command today. HTTP uses the port after the colon in the URL.

### Expected result

This is Terminal A. The last command remains running in the foreground; the shell prompt does not return. Startup wording varies by Python version.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-03/workspace
$ printf 'Hello from Day 3\n' > labs/01-linux/day-03/workspace/index.html
$ python3 -m http.server 8765 --bind 127.0.0.1 --directory labs/01-linux/day-03/workspace
Serving HTTP on 127.0.0.1 port 8765 (http://127.0.0.1:8765/) ...
```

## Exercise 2 — Inspect the listener and make requests

Run in **Terminal B**:

- `lsof` lists open files and sockets. `-nP` keeps IP addresses numeric (`-n`) and ports numeric (`-P`).
- `-iTCP:8765` selects TCP sockets on this port; `-sTCP:LISTEN` restricts them to listening sockets. Read the COMMAND, PID, and NAME columns.
- `curl` requests a URL. `-i` includes HTTP response headers alongside the body. `--connect-timeout 2` limits connection establishment to two seconds; `--max-time 5` caps the whole operation at five seconds.
- `http://` selects plaintext HTTP; `127.0.0.1` selects this computer; `8765` selects the listener; `/` requests the root resource.
- `/missing-file` requests a file you did not create. HTTP 404 is an expected application response; a connection succeeded even though that resource is absent.

```console
$ lsof -nP -iTCP:8765 -sTCP:LISTEN
$ curl -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/
$ curl -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/missing-file
```

Expect a Python listener on `127.0.0.1:8765`, a 200 response with your greeting, and a 404 response for the absent file. Look at Terminal A: each request should produce a log line. By default, curl can exit successfully after receiving a 404 because it completed the HTTP exchange.

### Expected result

This is Terminal B. PID 12345, socket metadata, Python version, timestamps, and headers are samples. Output omissions are labeled in brackets. The 404 is expected: the connection and HTTP exchange succeeded, but the resource is missing. Terminal A also logs each request.

```console
$ lsof -nP -iTCP:8765 -sTCP:LISTEN
COMMAND   PID USER    FD TYPE DEVICE SIZE/OFF NODE NAME
Python  12345 learner  3u IPv4 0x1234      0t0  TCP 127.0.0.1:8765 (LISTEN)
$ curl -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/
HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.13.0
Date: Fri, 09 Oct 2026 03:00:01 GMT
Content-type: text/html
Content-Length: 17
Last-Modified: Fri, 09 Oct 2026 03:00:00 GMT

Hello from Day 3
$ curl -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/missing-file
HTTP/1.0 404 File not found
Server: SimpleHTTP/0.6 Python/3.13.0
Date: Fri, 09 Oct 2026 03:00:02 GMT
Content-Type: text/html;charset=utf-8
[remaining headers omitted]

<!DOCTYPE HTML>
[HTML error body omitted: Error code: 404; Message: File not found.]
```

## Exercise 3 — Stop the listener and observe a connection failure

In **Terminal A**, press **Ctrl-C** once. This sends SIGINT to the foreground server and returns you to the shell. No command needs to be pasted for this step.

Then run in **Terminal B**:

- `lsof` uses the same filters as Exercise 2. With the server stopped, no listener output is expected, and the command can exit with a nonzero status.
- `curl -v` prints connection and protocol details. The timeout options retain the same bounds as above.

```console
$ lsof -nP -iTCP:8765 -sTCP:LISTEN
$ curl -v --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/
```

A curl connection failure and nonzero exit status are **expected observations** here. There should be no HTTP status code because the client could not reach an HTTP server. If a response appears, inspect the listener again before drawing a conclusion; another process may be using the port.

### Expected result

First press Ctrl-C in Terminal A as instructed; then read this Terminal B transcript. No `lsof` output means no matching listener. Curl’s connection error and nonzero status are expected; there is no HTTP response. Diagnostic wording and elapsed time vary.

```console
$ lsof -nP -iTCP:8765 -sTCP:LISTEN
$ curl -v --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/
*   Trying 127.0.0.1:8765...
* connect to 127.0.0.1 port 8765 failed: Connection refused
* Failed to connect to 127.0.0.1 port 8765 after 0 ms: Could not connect to server
curl: (7) Failed to connect to 127.0.0.1 port 8765 after 0 ms: Could not connect to server
```

## Check your understanding

1. Why does binding to `127.0.0.1` keep this lab local to your computer?
2. What evidence connects the Python process to port 8765?
3. How does a 404 response differ from the failure after Ctrl-C?

### Answer key

1. `127.0.0.1` is loopback, so the listener accepts connections through this computer’s local networking interface rather than its external interfaces.
2. The `lsof` row combines the Python command and PID with `127.0.0.1:8765 (LISTEN)`.
3. A 404 means a server accepted the connection and returned an HTTP response for a missing resource. After Ctrl-C, the connection fails before any HTTP response.

**Share back:** Send listener output, the two HTTP status codes, the final connection error, and your answers.

## Completion criteria

- [x] You reached your greeting through a loopback TCP listener.
- [x] You associated the listener with its PID.
- [x] You explained HTTP 404 versus a connection failure.
- [x] The foreground server is stopped with Ctrl-C.
