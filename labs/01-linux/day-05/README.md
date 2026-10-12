# Day 5 — Logs and First-Response Troubleshooting

**Time:** About 60–90 minutes for the core exercises. The extensions are additional self-paced sessions.

In the command blocks below, `$` is a prompt marker, not part of the command. Type the text after `$` and press Enter once per line, using Terminal A or B as instructed.

## Progress

- [ ] Exercise 1 — Start a local server and capture its logs.
- [ ] Exercise 2 — Diagnose an incorrect client port.
- [ ] Exercise 3 — Restore the request and write an incident note.
- [ ] Check your understanding — Answer the three questions below.
- [ ] Core Day 5 complete — Exercises 1–3 and answers reviewed.
- [ ] Extension 4 — Compare HTTP status with curl exit status.
- [ ] Extension 5 — Compare standard output and standard error capture.
- [ ] Extension checks — Practice and review both new question sets.
- [ ] Expanded Day 5 complete — Core work and both extensions reviewed.

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

## Core completion criteria

- [ ] You captured startup and request logs in a local file.
- [ ] You diagnosed an incorrect port using client, process, listener, and log evidence.
- [ ] You restored HTTP 200 by correcting the client port.
- [ ] You wrote an incident note and stopped the server with Ctrl-C.

## Extension 4 — Compare HTTP 404 with curl's exit status

**Context:** An HTTP error response can still be a completed transfer. Use a separate local workspace to see how curl reports that distinction. Review [curl --fail](https://curl.se/docs/manpage.html#-f) and [curl exit codes](https://curl.se/docs/manpage.html#EXIT-CODES).

Finish the core server cleanup first. In **Terminal A**:

- `cd` enters the project root. `mkdir -p` creates an extension-only workspace and its `served` subdirectory. Serve this empty subdirectory so `/missing.txt` has no matching file; keep logs outside it.
- `python3 -u -m http.server` starts the Python server with unbuffered output. Port `8775` is separate from the core lab, `--bind 127.0.0.1` restricts it to loopback, and `--directory` selects the served directory.
- `> ... 2>&1` captures startup and request messages in the extension log, overwriting an earlier extension run.
- The leading `$` is a prompt marker; type one command and press Enter per line.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-05/workspace/extension-04/served
$ python3 -u -m http.server 8775 --bind 127.0.0.1 --directory labs/01-linux/day-05/workspace/extension-04/served > labs/01-linux/day-05/workspace/extension-04/server.log 2>&1
```

Leave Terminal A running. If it returns to a prompt, read the log from Terminal B. If the port is occupied, use another unused port consistently; do not stop an unknown process. Keep `served` empty, including on reruns.

In **Terminal B**:

- `curl --noproxy 127.0.0.1` keeps this loopback request direct even if proxy variables are configured. `--silent --show-error` suppresses the progress meter and preserves errors; `--output /dev/null` discards the body.
- `--write-out 'http=%{http_code}\n'` prints the HTTP response code. The two timeouts bound connection establishment and the full request to two and five seconds.
- `--fail` makes this HTTP 404 produce curl exit status `22` and suppresses the response body.
- In `printf 'exit=%s\n' "$?"`, `%s` formats its argument and `\n` adds a newline. **Type the literal `$?`**: the shell expands it to the previous command's exit status. Run this immediately after each curl command; any intervening command replaces that status. The leading `$` still remains only a prompt marker.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ curl --noproxy 127.0.0.1 --silent --show-error --output /dev/null --connect-timeout 2 --max-time 5 --write-out 'http=%{http_code}\n' http://127.0.0.1:8775/missing.txt
$ printf 'exit=%s\n' "$?"
$ curl --noproxy 127.0.0.1 --silent --show-error --fail --output /dev/null --connect-timeout 2 --max-time 5 --write-out 'http=%{http_code}\n' http://127.0.0.1:8775/missing.txt
$ printf 'exit=%s\n' "$?"
$ tail -n 10 labs/01-linux/day-05/workspace/extension-04/server.log
```

### Expected result

**Terminal A:** The server stays in the foreground with its output redirected.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-05/workspace/extension-04/served
$ python3 -u -m http.server 8775 --bind 127.0.0.1 --directory labs/01-linux/day-05/workspace/extension-04/served > labs/01-linux/day-05/workspace/extension-04/server.log 2>&1
```

**Terminal B:** Diagnostic wording and timestamps vary. Error text and write-out output use different streams, so their displayed order can vary. Both requests reach the server and receive 404.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ curl --noproxy 127.0.0.1 --silent --show-error --output /dev/null --connect-timeout 2 --max-time 5 --write-out 'http=%{http_code}\n' http://127.0.0.1:8775/missing.txt
http=404
$ printf 'exit=%s\n' "$?"
exit=0
$ curl --noproxy 127.0.0.1 --silent --show-error --fail --output /dev/null --connect-timeout 2 --max-time 5 --write-out 'http=%{http_code}\n' http://127.0.0.1:8775/missing.txt
curl: (22) The requested URL returned error: 404
http=404
$ printf 'exit=%s\n' "$?"
exit=22
$ tail -n 10 labs/01-linux/day-05/workspace/extension-04/server.log
Serving HTTP on 127.0.0.1 port 8775 (http://127.0.0.1:8775/) ...
127.0.0.1 - - [09/Oct/2026 10:05:00] code 404, message File not found
127.0.0.1 - - [09/Oct/2026 10:05:00] "GET /missing.txt HTTP/1.1" 404 -
127.0.0.1 - - [09/Oct/2026 10:05:01] code 404, message File not found
127.0.0.1 - - [09/Oct/2026 10:05:01] "GET /missing.txt HTTP/1.1" 404 -
```

### Check your understanding

1. Why does ordinary curl return exit status zero for this HTTP 404?
2. What changed when `--fail` was added?
3. Why must `printf` run immediately after curl?

### Answer key

1. Curl completed the transfer; by default, HTTP 404 alone does not make the command fail. The resource request still failed at the HTTP level.
2. Curl treats this HTTP error as a command failure, with exit status `22`; the server still responds with 404.
3. `$?` reports only the immediately preceding command. Running another command first loses curl's status.

**Cleanup and safe rerun:** Press **Ctrl-C in Terminal A** and wait for its prompt. In Terminal B, check the listener with the command below; no output is expected. Keep the extension log as evidence. A rerun overwrites only this extension log; the served directory must stay empty.

```console
$ lsof -nP -iTCP:8775 -sTCP:LISTEN
```

**Expected result after Ctrl-C:**

```console
$ lsof -nP -iTCP:8775 -sTCP:LISTEN
```

## Extension 5 — See which output stream a log captures

**Context:** A quiet log file can result from capturing the wrong stream. Python's HTTP request logging writes to standard error, as documented for [BaseHTTPRequestHandler.log_message](https://docs.python.org/3/library/http.server.html#http.server.BaseHTTPRequestHandler.log_message). Use controlled text to make the stream routing visible. Read the [zsh redirection reference](https://zsh.sourceforge.io/Doc/Release/Redirection.html).

Use either terminal after the extension server is stopped:

- `mkdir -p` creates an extension-only directory. `sh -c` runs the single quoted command string in a child shell.
- Each child runs two `printf` commands: `stdout line` goes to descriptor 1, and `stderr line` goes to descriptor 2 because of `>&2`. The semicolon inside the quoted string separates the two child commands; enter the whole displayed line once.
- The first outer `>` redirects only stdout into `stdout-only.log`. Stderr remains visible in the terminal.
- The second outer `> ... 2>&1` first sends stdout to `combined.log`, then sends stderr to the same destination. Redirections are processed left to right; reversing them would send stderr to the original stdout destination.
- `cat` prints each saved file. `>` overwrites these two extension files on a rerun, and `\n` in each `printf` format ends the line.
- The leading `$` is a prompt marker, not text to type. The quoted text contains no shell variable placeholders.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-05/workspace/extension-05
$ sh -c 'printf "stdout line\n"; printf "stderr line\n" >&2' > labs/01-linux/day-05/workspace/extension-05/stdout-only.log
$ cat labs/01-linux/day-05/workspace/extension-05/stdout-only.log
$ sh -c 'printf "stdout line\n"; printf "stderr line\n" >&2' > labs/01-linux/day-05/workspace/extension-05/combined.log 2>&1
$ cat labs/01-linux/day-05/workspace/extension-05/combined.log
```

### Expected result

The first child prints `stderr line` directly to the terminal. The second child is silent there; both lines appear when you read its file.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-05/workspace/extension-05
$ sh -c 'printf "stdout line\n"; printf "stderr line\n" >&2' > labs/01-linux/day-05/workspace/extension-05/stdout-only.log
stderr line
$ cat labs/01-linux/day-05/workspace/extension-05/stdout-only.log
stdout line
$ sh -c 'printf "stdout line\n"; printf "stderr line\n" >&2' > labs/01-linux/day-05/workspace/extension-05/combined.log 2>&1
$ cat labs/01-linux/day-05/workspace/extension-05/combined.log
stdout line
stderr line
```

### Check your understanding

1. Why is `stderr line` absent from `stdout-only.log`?
2. What does `2>&1` mean when it follows the file redirection?
3. Why could redirecting only stdout miss Python HTTP request logs?

### Answer key

1. `>` captures descriptor 1 only; the child explicitly writes that line to descriptor 2.
2. It sends descriptor 2 to descriptor 1's current destination, which is already the file.
3. `BaseHTTPRequestHandler.log_message` writes request messages to stderr, so stdout-only capture does not include them.

**Cleanup and safe rerun:** No long-running process is created. Keep the two small logs as evidence. Rerunning these commands overwrites only the named extension files; no deletion or administrator privileges are needed.

## Expanded Day 5 completion criteria

- [ ] You observed HTTP 404 with curl exit statuses 0 and 22 and read `$?` immediately.
- [ ] You stopped the extension server and confirmed its listener was gone.
- [ ] You compared stdout-only and combined logs and explained redirection order.
- [ ] You answered both extensions’ checks and reviewed their evidence.
- [ ] Expanded Day 5 complete — Core criteria and all extension criteria reviewed.
