# Day 3 — Ports and TCP Connections

**Time:** Self-paced, over additional sessions. The completed core took about 60–90 minutes; work through the extensions in separate sessions, pausing whenever needed.

In the command blocks below, `$` is a prompt marker, not part of the command. Type the text after `$` and press Enter once per line, using Terminal A or B as instructed.

Flag meanings in parentheses are memory aids, not invented long-option aliases. BSD (**Berkeley Software Distribution**)/macOS tools may not support GNU (**GNU’s Not Unix**, a recursive acronym) long options; use only the options documented for the installed command.

TCP means **Transmission Control Protocol**; HTTP means **Hypertext Transfer Protocol**. IP means **Internet Protocol**; IPv4 and IPv6 are **Internet Protocol version 4** and **version 6**. Command names, abbreviations, and option meanings are also collected in the [glossary](../../../notes/glossary.md).

## Progress

- [x] Exercise 1 — Start a local HTTP listener.
- [x] Exercise 2 — Inspect the listener and make requests.
- [x] Exercise 3 — Stop the listener and observe a connection failure.
- [x] Check your understanding — Answer the three questions below.
- [x] Core Day 3 complete — Exercises 1–3 and their answers reviewed.
- [ ] Extension 4 — Observe a deliberate port collision.
- [ ] Extension 5 — Compare IPv4 and IPv6 loopback.
- [ ] Extension checks — Practice and review both new question sets.
- [ ] Expanded Day 3 complete — Core and extensions reviewed.

## Why this matters

A running process does not necessarily accept network traffic. A client needs an address and a port where a process is listening. Testing the listener separately from the HTTP response helps distinguish a connection failure from an application error.

## Learning goals

1. Explain loopback addresses, ports, and a TCP listener.
2. Connect a listening socket to its process with `lsof`.
3. Distinguish HTTP 404 from failure to establish a connection.

## Reading

Read [Cloudflare: What is TCP/IP?](https://www.cloudflare.com/learning/ddos/glossary/tcp-ip/). In the [curl manual](https://curl.se/docs/manpage.html), read `--include`, `--verbose`, `--connect-timeout`, and `--max-time`. TCP provides an ordered byte stream; HTTP gives those bytes request and response meaning.

## Core Day 3 — Already completed

Exercises 1–3 and their checked criteria remain complete. The new extensions below are pending; complete and review them before marking the expanded day complete.

## Exercise 1 — Start a local HTTP listener

Open two terminal tabs, called **Terminal A** and **Terminal B**. Run this block in **Terminal A**:

- `cd` (**change directory**) enters the project root; `mkdir` (**make directory**) with `-p` uses `-p` (parents) to create an isolated workspace and missing parents.
- `printf` (**formatted print**) produces a line ending in `\n`; `>` creates or overwrites the lab's `index.html`. HTML means **Hypertext Markup Language**; this sample file contains plain greeting text.
- `python3 -m http.server` uses `-m` (module) to run Python's built-in HTTP server module. `8765` is the TCP port.
- `--bind 127.0.0.1` accepts connections only through this computer's IPv4 loopback address.
- `--directory` serves only the specified lab workspace. This is a static file server, not the future Go application programming interface (**API**).

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-03/workspace
$ printf 'Hello from Day 3\n' > labs/01-linux/day-03/workspace/index.html
$ python3 -m http.server 8765 --bind 127.0.0.1 --directory labs/01-linux/day-03/workspace
```

Leave Terminal A running while you use Terminal B. Expect a serving message. If you see “Address already in use,” stop this attempt; do not stop an unknown process. Pick an unused port such as `8766` and substitute it in every command today. HTTP uses the port after the colon in the URL (**Uniform Resource Locator**, the resource address).

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

- `lsof` means **list open files**; it lists open files and sockets. `-nP` keeps IP addresses numeric (`-n`) and ports numeric (`-P`).
- `-iTCP:8765` uses `-i` to select Internet sockets, restricted here to TCP on this port; `-sTCP:LISTEN` uses `-s` to select a socket state, restricted here to listening TCP sockets. Read the COMMAND, PID (**process identifier**, process ID), and NAME columns.
- `curl` is a **URL transfer tool**; here it requests a URL. `-i` (`--include`) includes HTTP response headers alongside the body. `--connect-timeout 2` limits connection establishment to two seconds; `--max-time 5` caps the whole operation at five seconds.
- `http://` selects plaintext HTTP; `127.0.0.1` selects this computer; `8765` selects the listener; `/` requests the root resource.
- `/missing-file` requests a file you did not create. HTTP 404 is an expected application response; a connection succeeded even though that resource is absent.

```console
$ lsof -nP -iTCP:8765 -sTCP:LISTEN
$ curl -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/
$ curl -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8765/missing-file
```

Expect a Python listener on `127.0.0.1:8765`, a 200 response with your greeting, and a 404 response for the absent file. Look at Terminal A: each request should produce a log line. By default, curl can exit successfully after receiving a 404 because it completed the HTTP exchange.

### Expected result

This is Terminal B. PID 12345, socket metadata, Python version, timestamps, and headers are samples. FD means **file descriptor**, an open handle owned by the process; `3u` is descriptor 3 opened for reading and writing. GMT means **Greenwich Mean Time**, the time reference shown in these HTTP date headers. UTF-8 means **Unicode Transformation Format, 8-bit**, the text encoding shown in the error response. Output omissions are labeled in brackets. The 404 is expected: the connection and HTTP exchange succeeded, but the resource is missing. Terminal A also logs each request.

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

In **Terminal A**, press **Ctrl-C** once. This sends SIGINT (**signal interrupt**) to the foreground server and returns you to the shell. No command needs to be pasted for this step.

Then run in **Terminal B**:

- `lsof` uses the same filters as Exercise 2. With the server stopped, no listener output is expected, and the command can exit with a nonzero status.
- `curl -v` uses `-v` (`--verbose`) to print connection and protocol details. The timeout options retain the same bounds as above.

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

## Core completion criteria

- [x] You reached your greeting through a loopback TCP listener.
- [x] You associated the listener with its PID.
- [x] You explained HTTP 404 versus a connection failure.
- [x] The foreground server is stopped with Ctrl-C.

## Extension 4 — Observe a deliberate port collision

A process can fail to start because its requested address/port is already occupied. Here both servers belong to you: the second startup fails, while the first should keep serving. This separates a startup error from an outage of the existing listener.

**Written reading:** [Python: http.server command-line interface](https://docs.python.org/3/library/http.server.html#command-line-interface) and [Apple: bind(2), EADDRINUSE](https://developer.apple.com/library/archive/documentation/System/Conceptual/ManPages_iPhoneOS/man2/bind.2.html). `EADDRINUSE` denotes an address already in use; Python's error text/number varies by operating system (**OS**).

Use Terminal A for the first server and Terminal B for preparation, the second attempt, and requests. Before the commands:

- `cd` independently puts each terminal at the project root. `lsof -nP -iTCP:8875 -sTCP:LISTEN` lists TCP listeners on the chosen port without resolving address (`-n`) or port (`-P`) names. Start only if this initial check is empty. An empty result can return nonzero; if occupied, choose an unused port and substitute it everywhere in this extension. Never stop an unknown listener.
- `mkdir -p` creates distinct `extensions/collision/first` and `second` serving directories. `printf` and `>` write different lab greetings with a newline so the response identifies which server answered.
- `python3 -m http.server 8875` runs the built-in static server on port 8875. `--bind 127.0.0.1` limits it to IPv4 loopback; `--directory` specifies the isolated serving folder. Leave Terminal A running during the Terminal B steps.
- `curl --noproxy '*'` bypasses proxy routing for this invocation; quoting prevents the shell from expanding `*`. `-i` includes headers. `--connect-timeout 2` and `--max-time 5` bound connection establishment and the entire request in seconds.
- The second Python command asks for exactly the same loopback address and port. Its failure is intentional. It should return to Terminal B's prompt; do not try a different port for just the second command, as that would remove the collision.
- Compare `lsof` PIDs before and after the failed attempt, and compare both curl bodies. To clean up, press **Ctrl-C in Terminal A** only after verification, then run Terminal B's final listener check. `rm` (**remove**) deletes the two explicitly named fixture files; `rmdir` (**remove directory**) removes only their empty directories.

**Prepare in Terminal B:**

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ lsof -nP -iTCP:8875 -sTCP:LISTEN
$ mkdir -p labs/01-linux/day-03/extensions/collision/first labs/01-linux/day-03/extensions/collision/second
$ printf 'first server\n' > labs/01-linux/day-03/extensions/collision/first/index.html
$ printf 'second server\n' > labs/01-linux/day-03/extensions/collision/second/index.html
```

**Start the first server in Terminal A:**

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ python3 -m http.server 8875 --bind 127.0.0.1 --directory labs/01-linux/day-03/extensions/collision/first
```

**Verify and attempt the collision in Terminal B:**

```console
$ lsof -nP -iTCP:8875 -sTCP:LISTEN
$ curl --noproxy '*' -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8875/
$ python3 -m http.server 8875 --bind 127.0.0.1 --directory labs/01-linux/day-03/extensions/collision/second
$ lsof -nP -iTCP:8875 -sTCP:LISTEN
$ curl --noproxy '*' -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8875/
```

**After Ctrl-C in Terminal A, clean up from Terminal B:**

```console
$ lsof -nP -iTCP:8875 -sTCP:LISTEN
$ rm labs/01-linux/day-03/extensions/collision/first/index.html labs/01-linux/day-03/extensions/collision/second/index.html
$ rmdir labs/01-linux/day-03/extensions/collision/first labs/01-linux/day-03/extensions/collision/second
$ rmdir labs/01-linux/day-03/extensions/collision
```

### Expected result

PID 52083, metadata, response versions/headers, and error wording are representative. The second Python command's nonzero status and address-in-use error are **expected**, while the unchanged first PID and repeated greeting demonstrate continued service.

**Terminal B preparation:**

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ lsof -nP -iTCP:8875 -sTCP:LISTEN
$ mkdir -p labs/01-linux/day-03/extensions/collision/first labs/01-linux/day-03/extensions/collision/second
$ printf 'first server\n' > labs/01-linux/day-03/extensions/collision/first/index.html
$ printf 'second server\n' > labs/01-linux/day-03/extensions/collision/second/index.html
```

**Terminal A startup:**

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ python3 -m http.server 8875 --bind 127.0.0.1 --directory labs/01-linux/day-03/extensions/collision/first
Serving HTTP on 127.0.0.1 port 8875 (http://127.0.0.1:8875/) ...
```

**Terminal B collision and verification:**

```console
$ lsof -nP -iTCP:8875 -sTCP:LISTEN
COMMAND   PID USER    FD TYPE DEVICE SIZE/OFF NODE NAME
Python  52083 learner  3u IPv4 0x1234      0t0  TCP 127.0.0.1:8875 (LISTEN)
$ curl --noproxy '*' -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8875/
HTTP/1.0 200 OK
[response headers omitted]

first server
$ python3 -m http.server 8875 --bind 127.0.0.1 --directory labs/01-linux/day-03/extensions/collision/second
Traceback (most recent call last):
[stack frames omitted]
OSError: [Errno 48] Address already in use
$ lsof -nP -iTCP:8875 -sTCP:LISTEN
COMMAND   PID USER    FD TYPE DEVICE SIZE/OFF NODE NAME
Python  52083 learner  3u IPv4 0x1234      0t0  TCP 127.0.0.1:8875 (LISTEN)
$ curl --noproxy '*' -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8875/
HTTP/1.0 200 OK
[response headers omitted]

first server
```

**Terminal B after Ctrl-C in Terminal A:**

```console
$ lsof -nP -iTCP:8875 -sTCP:LISTEN
$ rm labs/01-linux/day-03/extensions/collision/first/index.html labs/01-linux/day-03/extensions/collision/second/index.html
$ rmdir labs/01-linux/day-03/extensions/collision/first labs/01-linux/day-03/extensions/collision/second
$ rmdir labs/01-linux/day-03/extensions/collision
```

### Checks and answer key

1. **Check:** Which server failed, and why? **Answer:** The second could not bind the occupied loopback address/port; the first was already listening there.
2. **Check:** What shows that the failed startup did not stop the first server? **Answer:** Its listener PID stayed the same and both requests returned `first server` with 200.
3. **Check:** Would merely seeing a Python process prove this endpoint works? **Answer:** No. The listener identifies the address/port, and the request verifies the intended HTTP response.

**Cleanup and rerun:** Stop only your foreground first server with Ctrl-C and confirm no lab listener remains before removing the named fixtures. If startup behaved differently or another listener appears, inspect it and choose a new unused port for a fresh attempt; do not terminate it. Keep extra notes elsewhere so `rmdir` can remove the empty fixture directories. A nonempty-directory error is a reason to inspect contents, not to broaden deletion.

## Extension 5 — Compare IPv4 and IPv6 loopback

`127.0.0.1` is IPv4 loopback; `::1` is IPv6 loopback. A listener bound specifically to one address does not establish that the other address can reach it. Use explicit numeric addresses to observe the family difference without hostname resolution.

**Written reading:** [Python: http.server command-line interface](https://docs.python.org/3/library/http.server.html#command-line-interface), including IPv6 support for `--bind`. This requires local IPv6 support and a Python version that accepts the IPv6 binding (Python 3.8 or newer). If unavailable, record the actual error and leave this extension unchecked until successful practice; do not alter system networking to force it.

Before the commands:

- In Terminal B, `cd` enters the checkout; `lsof -nP -iTCP:8876 -sTCP:LISTEN` checks listeners on the chosen port across both families, keeping addresses and ports numeric. If occupied, choose another unused port and update every command in this extension.
- `mkdir -p` creates `extensions/ipv6`; `printf ... > ...` writes a newline-terminated greeting only in its `index.html`.
- In Terminal A, the absolute `cd` sets the working directory independently. `python3 -m http.server 8876 --bind ::1 --directory ...` starts a static server restricted to IPv6 loopback and the isolated serving directory. Leave it running.
- In Terminal B, `lsof` should identify an IPv6 socket on `[::1]:8876`. `curl --noproxy '*'` bypasses proxies only for that invocation; the quoted asterisk remains literal.
- `-g` (`--globoff`) disables curl's own URL range/glob interpretation. The single-quoted URL `'http://[::1]:8876/'` also prevents zsh (**Z shell**) glob interpretation of square brackets. URL brackets enclose the IPv6 literal so its internal colons can be distinguished from the colon before the port.
- `-i` includes headers; `--connect-timeout 2` and `--max-time 5` limit connection establishment and the full request. The request to `127.0.0.1` uses the same port but an IPv4 destination. With only this IPv6-loopback listener, that IPv4 request should fail **as expected**.
- Press **Ctrl-C in Terminal A** to stop your server after both observations, then check for a remaining listener in Terminal B. `rm` removes the single named fixture; `rmdir` removes its empty directory.

**Prepare in Terminal B:**

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ lsof -nP -iTCP:8876 -sTCP:LISTEN
$ mkdir -p labs/01-linux/day-03/extensions/ipv6
$ printf 'IPv6 loopback\n' > labs/01-linux/day-03/extensions/ipv6/index.html
```

**Start in Terminal A:**

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ python3 -m http.server 8876 --bind ::1 --directory labs/01-linux/day-03/extensions/ipv6
```

**Compare from Terminal B:**

```console
$ lsof -nP -iTCP:8876 -sTCP:LISTEN
$ curl --noproxy '*' -g -i --connect-timeout 2 --max-time 5 'http://[::1]:8876/'
$ curl --noproxy '*' -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8876/
```

**After Ctrl-C in Terminal A, clean up from Terminal B:**

```console
$ lsof -nP -iTCP:8876 -sTCP:LISTEN
$ rm labs/01-linux/day-03/extensions/ipv6/index.html
$ rmdir labs/01-linux/day-03/extensions/ipv6
```

### Expected result

This is a successful local IPv6 example; PIDs, metadata, serving-message formatting, response headers, and diagnostic timing vary. The IPv4 connection failure is expected. If IPv6 startup fails with an address-family or address-availability error, save that observation, skip claiming a successful IPv6 response, and leave practice pending. If the IPv4 request unexpectedly succeeds, inspect listeners for another process before interpreting it.

**Terminal B preparation:**

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ lsof -nP -iTCP:8876 -sTCP:LISTEN
$ mkdir -p labs/01-linux/day-03/extensions/ipv6
$ printf 'IPv6 loopback\n' > labs/01-linux/day-03/extensions/ipv6/index.html
```

**Terminal A startup:**

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ python3 -m http.server 8876 --bind ::1 --directory labs/01-linux/day-03/extensions/ipv6
Serving HTTP on ::1 port 8876 (http://[::1]:8876/) ...
```

**Terminal B comparison:**

```console
$ lsof -nP -iTCP:8876 -sTCP:LISTEN
COMMAND   PID USER    FD TYPE DEVICE SIZE/OFF NODE NAME
Python  52084 learner  3u IPv6 0x5678      0t0  TCP [::1]:8876 (LISTEN)
$ curl --noproxy '*' -g -i --connect-timeout 2 --max-time 5 'http://[::1]:8876/'
HTTP/1.0 200 OK
[response headers omitted]

IPv6 loopback
$ curl --noproxy '*' -i --connect-timeout 2 --max-time 5 http://127.0.0.1:8876/
curl: (7) Failed to connect to 127.0.0.1 port 8876 after 0 ms: Could not connect to server
```

**Terminal B after Ctrl-C in Terminal A:**

```console
$ lsof -nP -iTCP:8876 -sTCP:LISTEN
$ rm labs/01-linux/day-03/extensions/ipv6/index.html
$ rmdir labs/01-linux/day-03/extensions/ipv6
```

### Checks and answer key

1. **Check:** Are `::1` and `127.0.0.1` interchangeable destinations for this listener? **Answer:** No. They are loopback addresses in different families, and this server binds specifically to `::1`.
2. **Check:** Why both URL quotes and brackets? **Answer:** Brackets delimit the IPv6 literal in URL syntax; quotes prevent zsh from treating them as a shell pattern. `-g` separately disables curl's URL globbing.
3. **Check:** Does an unavailable local IPv6 stack mean you completed this experiment? **Answer:** No. Documenting that limitation is useful, but the extension remains pending until you can observe the successful IPv6 listener/request and the family comparison.

**Cleanup and rerun:** If startup failed, there is no owned running server to interrupt; remove only the named fixture and its empty directory. If it succeeded, stop it in Terminal A and verify cleanup from B. Do not kill an unknown PID or change networking settings. Check a fresh unused port before rerunning and keep both terminal sessions at the project root.

## Expanded Day 3 completion criteria

- [x] Core Exercises 1–3 and their original criteria remain reviewed.
- [ ] Extension 4 practiced: intentional collision observed while the first server continued serving.
- [ ] Extension 5 successfully practiced: IPv6 listener/request and IPv4 comparison observed.
- [ ] Both owned servers stopped and named extension fixtures cleaned up.
- [ ] Both extension question sets reviewed, including address/port ownership and URL syntax.
- [ ] Expanded Day 3 complete — All extension practice and cleanup reviewed.
