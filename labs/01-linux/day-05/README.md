# Day 5 — Logs and First-Response Troubleshooting

**Time:** About 60–90 minutes.

In the command blocks below, `$` is a prompt marker, not part of the command. Type the text after `$` and press Enter once per line, using Terminal A or B as instructed.

## Progress

- [ ] Exercise 1 — Start a local server and capture its logs.
- [ ] Exercise 2 — Diagnose an incorrect client port.
- [ ] Exercise 3 — Restore the request and write an incident note.
- [ ] Check your understanding — Answer the three questions below.
- [ ] Day 5 complete — All exercises and answers reviewed.

## Why this matters

A useful first response collects evidence before changing anything: what failed, whether a process exists, whether it listens where the client expects, and what the logs show. Today's failure is an incorrect client port. The server needs no database or Go API, so you can isolate the symptom locally.

## Learning goals

1. Capture standard output and standard error in a lab log.
2. Correlate client results, process information, listeners, and log lines.
3. Restore a request by correcting its port and summarize evidence clearly.

## Reading

Read [The Twelve-Factor App: Logs](https://12factor.net/logs), which treats logs as an event stream. In the [curl manual](https://curl.se/docs/manpage.html), review `--include`, `--verbose`, `--connect-timeout`, and `--max-time`. Saving a log to a file here is a local learning technique; a deployed platform can collect the application's output stream centrally.

## Exercise 1 — Start a local server and capture its logs

Open **Terminal A** and **Terminal B**. In **Terminal A**, before starting the server:

- `cd` enters the project root; `mkdir -p` creates the lab workspace and missing parents.
- `printf` writes a greeting with a newline (`\n`); `>` creates or overwrites `index.html`.
- `python3 -u` disables buffering for Python's standard output and error, helping log lines appear promptly. `-m http.server` runs the built-in server module on port `8765`.
- `--bind 127.0.0.1` limits access to IPv4 loopback. `--directory` limits served files to this workspace.
- `>` redirects standard output to `server.log`, overwriting an earlier session. `2>&1` then sends standard error (descriptor 2) to the same destination as standard output (descriptor 1). The order matters.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-05/workspace
$ printf 'Hello from Day 5\n' > labs/01-linux/day-05/workspace/index.html
$ python3 -u -m http.server 8765 --bind 127.0.0.1 --directory labs/01-linux/day-05/workspace > labs/01-linux/day-05/workspace/server.log 2>&1
```

Leave Terminal A occupied by the foreground server. Its output now goes to the log, so the terminal may look blank. If it returns immediately to a prompt, inspect the log in Terminal B. If the port is in use, choose another unused port, such as `8766`, and use it for every healthy request and listener check below. Do not stop an unknown process to free a port.

In **Terminal B**:

- `cd` enters the same checkout independently; terminal tabs have separate working directories.
- `tail -n 10` prints the last ten log lines to check startup.
- `curl -i` includes response headers. `--connect-timeout 2` and `--max-time 5` bound connection establishment and the full operation in seconds.
- The second `tail` shows the successful request's log line.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ tail -n 10 labs/01-linux/day-05/workspace/server.log
$ curl -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/
$ tail -n 10 labs/01-linux/day-05/workspace/server.log
```

Expect the greeting, HTTP 200, and a log entry for `GET /`. Confirm this baseline before introducing the incorrect port.

### Expected result

The server command in Terminal A stays running and prints nothing there because both output streams are redirected. The following Terminal B transcript shows the log before and after the successful request. Versions, headers, and timestamps vary.

**Terminal A:**

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-05/workspace
$ printf 'Hello from Day 5\n' > labs/01-linux/day-05/workspace/index.html
$ python3 -u -m http.server 8765 --bind 127.0.0.1 --directory labs/01-linux/day-05/workspace > labs/01-linux/day-05/workspace/server.log 2>&1
```

**Terminal B:**

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ tail -n 10 labs/01-linux/day-05/workspace/server.log
Serving HTTP on 127.0.0.1 port 8765 (http://127.0.0.1:8765/) ...
$ curl -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/
HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.13.0
Date: Fri, 09 Oct 2026 03:00:01 GMT
Content-type: text/html
Content-Length: 17
Last-Modified: Fri, 09 Oct 2026 03:00:00 GMT

Hello from Day 5
$ tail -n 10 labs/01-linux/day-05/workspace/server.log
Serving HTTP on 127.0.0.1 port 8765 (http://127.0.0.1:8765/) ...
127.0.0.1 - - [09/Oct/2026 10:00:01] "GET / HTTP/1.1" 200 -
```

## Exercise 2 — Diagnose an incorrect client port

Use **Terminal B**. First check the deliberate wrong port:

- `lsof -nP` lists sockets with numeric IP addresses and ports. `-iTCP:8764` selects port 8764; `-sTCP:LISTEN` selects listeners.
- No output, with a possible nonzero exit status, means no visible listener matches. If a listener appears, choose a different unused port and replace `8764` in the failing curl command. Do not interact with that process.

```console
$ lsof -nP -iTCP:8764 -sTCP:LISTEN
```

Now collect evidence:

- `curl -v` shows connection diagnostics, with the same timeout bounds as before. Port 8764 is intentionally different from the server port.
- `lsof` checks the actual listener on 8765 using the same options. Record its PID.
- `tail -n 10` reads recent logs. A connection that never reaches this server cannot produce an HTTP request log here.

```console
$ curl -v --connect-timeout 2 --max-time 5 http://127.0.0.1:8764/
$ lsof -nP -iTCP:8765 -sTCP:LISTEN
$ tail -n 10 labs/01-linux/day-05/workspace/server.log
```

The curl connection failure and nonzero status are **expected observations**. There should be no HTTP response status for that attempt, while the real listener remains available.

To inspect its process, use the PID from the listener output. Before the command: `ps -p` selects a PID, and `-o pid,ppid,user,stat,command` shows identity, parent, user, state, and command. Replace `12345` below with the observed PID; it is only a placeholder.

```console
$ ps -p 12345 -o pid,ppid,user,stat,command
```

Check that it is your lab Python server. Explain why a process check alone would not reveal the incorrect client port.

### Expected result

This is Terminal B. The wrong-port connection error is expected; it adds no HTTP request to this server’s log. For exact agreement with the exercise’s placeholder command, this example uses PID 12345; use your observed PID when doing the exercise. Process/socket metadata, command paths, timings, and diagnostic wording vary.

```console
$ lsof -nP -iTCP:8764 -sTCP:LISTEN
$ curl -v --connect-timeout 2 --max-time 5 http://127.0.0.1:8764/
*   Trying 127.0.0.1:8764...
* connect to 127.0.0.1 port 8764 failed: Connection refused
* Failed to connect to 127.0.0.1 port 8764 after 0 ms: Could not connect to server
curl: (7) Failed to connect to 127.0.0.1 port 8764 after 0 ms: Could not connect to server
$ lsof -nP -iTCP:8765 -sTCP:LISTEN
COMMAND   PID USER    FD TYPE DEVICE SIZE/OFF NODE NAME
Python  12345 learner  3u IPv4 0x1234      0t0  TCP 127.0.0.1:8765 (LISTEN)
$ tail -n 10 labs/01-linux/day-05/workspace/server.log
Serving HTTP on 127.0.0.1 port 8765 (http://127.0.0.1:8765/) ...
127.0.0.1 - - [09/Oct/2026 10:00:01] "GET / HTTP/1.1" 200 -
$ ps -p 12345 -o pid,ppid,user,stat,command
  PID  PPID USER     STAT COMMAND
12345 30700 learner  S+   python3 -u -m http.server 8765 --bind 127.0.0.1 --directory labs/01-linux/day-05/workspace
```

## Exercise 3 — Restore the request and write an incident note

In **Terminal B**:

- `curl -i` uses the correct port again and includes headers; the timeouts remain two and five seconds.
- `tail -n 10` shows recent logs, including the recovered request.

```console
$ curl -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/
$ tail -n 10 labs/01-linux/day-05/workspace/server.log
```

Expect HTTP 200 and the greeting. In personal notes, write five short points: symptom, client address/port, process/listener evidence, log evidence, and the correction with its verification. State that this static server has no database dependency to investigate; future services may have dependencies even when their listener is healthy.

For cleanup, press **Ctrl-C** in **Terminal A**. It sends SIGINT to the foreground server even though output is redirected. Wait for the shell prompt. Then in **Terminal B**, `lsof` with the same filters confirms that the lab listener is gone:

```console
$ lsof -nP -iTCP:8765 -sTCP:LISTEN
```

No output is expected after cleanup. Keep the log as learning evidence. Do not mark this day complete until the observations and answers have been reviewed.

### Expected result

This is Terminal B. The corrected request returns 200 and adds a log entry. After that request and log check, press Ctrl-C in Terminal A before the final listener check. Timestamps and headers vary; the final `lsof` is silent because the listener is stopped.

```console
$ curl -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/
HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.13.0
Date: Fri, 09 Oct 2026 03:00:01 GMT
Content-type: text/html
Content-Length: 17
Last-Modified: Fri, 09 Oct 2026 03:00:00 GMT

Hello from Day 5
$ tail -n 10 labs/01-linux/day-05/workspace/server.log
Serving HTTP on 127.0.0.1 port 8765 (http://127.0.0.1:8765/) ...
127.0.0.1 - - [09/Oct/2026 10:00:01] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [09/Oct/2026 10:02:00] "GET / HTTP/1.1" 200 -
```

**After Ctrl-C in Terminal A, check from Terminal B:**

```console
$ lsof -nP -iTCP:8765 -sTCP:LISTEN
```

**Sample incident note:** Connection to port 8764 failed without an HTTP response. The Python process still listened on 8765, and its log contained the earlier successful request but no failed attempt. Correcting the client port returned 200 and added a successful request log. After Ctrl-C, no listener remained. This lab has no database dependency.

## Check your understanding

1. Why does the incorrect-port attempt produce no HTTP request line in this server's log?
2. What does a listener prove, and what still needs an HTTP request to verify?
3. What evidence showed the server stayed healthy while the client request failed?

### Answer key

1. The client contacted a different port and never reached this server, so the server had no HTTP request to log.
2. A listener proves a process has a socket accepting connections on that address/port. An HTTP request is needed to verify the requested resource and response behavior.
3. The Python process and actual listener remained present, and requests to the correct port returned 200 with matching log entries. Only the wrong-port attempt failed.

**Share back:** Send your incident note, relevant outputs and log lines, and answers.

## Completion criteria

- [ ] You captured startup and request logs in a local file.
- [ ] You diagnosed an incorrect port using client, process, listener, and log evidence.
- [ ] You restored HTTP 200 by correcting the client port.
- [ ] You wrote an incident note and stopped the server with Ctrl-C.
