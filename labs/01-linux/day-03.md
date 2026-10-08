# Day 3 — Ports and TCP Connections

**Time:** About 60–90 minutes.

## Progress

- [ ] Exercise 1 — Start a local HTTP listener.
- [ ] Exercise 2 — Inspect the listener and make requests.
- [ ] Exercise 3 — Stop the listener and observe a connection failure.
- [ ] Check your understanding — Answer the three questions below.
- [ ] Day 3 complete — All exercises and answers reviewed.

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

```sh
cd /Users/citradigital/Desktop/learn/platform-engineer
mkdir -p labs/01-linux/day-03/workspace
printf 'Hello from Day 3\n' > labs/01-linux/day-03/workspace/index.html
python3 -m http.server 8765 --bind 127.0.0.1 --directory labs/01-linux/day-03/workspace
```

Leave Terminal A running while you use Terminal B. Expect a serving message. If you see “Address already in use,” stop this attempt; do not stop an unknown process. Pick an unused port such as `8766` and substitute it in every command today. HTTP uses the port after the colon in the URL.

## Exercise 2 — Inspect the listener and make requests

Run in **Terminal B**:

- `lsof` lists open files and sockets. `-nP` keeps IP addresses numeric (`-n`) and ports numeric (`-P`).
- `-iTCP:8765` selects TCP sockets on this port; `-sTCP:LISTEN` restricts them to listening sockets. Read the COMMAND, PID, and NAME columns.
- `curl` requests a URL. `-i` includes HTTP response headers alongside the body. `--connect-timeout 2` limits connection establishment to two seconds; `--max-time 5` caps the whole operation at five seconds.
- `http://` selects plaintext HTTP; `127.0.0.1` selects this computer; `8765` selects the listener; `/` requests the root resource.
- `/missing-file` requests a file you did not create. HTTP 404 is an expected application response; a connection succeeded even though that resource is absent.

```sh
lsof -nP -iTCP:8765 -sTCP:LISTEN
curl -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/
curl -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/missing-file
```

Expect a Python listener on `127.0.0.1:8765`, a 200 response with your greeting, and a 404 response for the absent file. Look at Terminal A: each request should produce a log line. By default, curl can exit successfully after receiving a 404 because it completed the HTTP exchange.

## Exercise 3 — Stop the listener and observe a connection failure

In **Terminal A**, press **Ctrl-C** once. This sends SIGINT to the foreground server and returns you to the shell. No command needs to be pasted for this step.

Then run in **Terminal B**:

- `lsof` uses the same filters as Exercise 2. With the server stopped, no listener output is expected, and the command can exit with a nonzero status.
- `curl -v` prints connection and protocol details. The timeout options retain the same bounds as above.

```sh
lsof -nP -iTCP:8765 -sTCP:LISTEN
curl -v --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/
```

A curl connection failure and nonzero exit status are **expected observations** here. There should be no HTTP status code because the client could not reach an HTTP server. If a response appears, inspect the listener again before drawing a conclusion; another process may be using the port.

## Check your understanding

1. Why does binding to `127.0.0.1` keep this lab local to your computer?
2. What evidence connects the Python process to port 8765?
3. How does a 404 response differ from the failure after Ctrl-C?

**Share back:** Send listener output, the two HTTP status codes, the final connection error, and your answers.

## Completion criteria

- [ ] You reached your greeting through a loopback TCP listener.
- [ ] You associated the listener with its PID.
- [ ] You explained HTTP 404 versus a connection failure.
- [ ] The foreground server is stopped with Ctrl-C.
