# Learning Glossary

Use this reference when an abbreviation, command name, or short option appears in a lab. The first explanatory use in each day guide also gives its meaning. An option letter is local to its command: `curl -i` and `lsof -i` have different meanings, and `curl -i` differs from `curl -I`.

Some command names are shortened words or historical names rather than acronyms. In those cases, the description below explains what the command does without inventing an expansion. Commands and flags can differ between macOS/BSD and GNU/Linux; run the version printed in your lab.

## Networking and web

| Term | Full form or meaning |
|---|---|
| DNS | Domain Name System; translates names into records, including address records. |
| IP / IPv4 / IPv6 | Internet Protocol / Internet Protocol version 4 / Internet Protocol version 6. |
| TCP | Transmission Control Protocol; carries a reliable byte stream between endpoints. |
| HTTP | Hypertext Transfer Protocol; defines requests and responses between clients and servers. |
| HTTPS | HTTP over Transport Layer Security (TLS); the `S` commonly indicates “secure.” |
| TLS | Transport Layer Security; protects the connection and authenticates the server certificate. |
| URL | Uniform Resource Locator; identifies where and how to access a resource. |
| A / AAAA | DNS record types holding IPv4 / IPv6 addresses. They are type names, not words to expand. |
| TTL | Time to live; in a DNS answer, a cache lifetime measured in seconds. |
| IN | The Internet DNS record class. |
| GET / HEAD | HTTP method names. GET requests a representation; HEAD requests its response headers without a response body. They are not acronyms. |
| HTML | Hypertext Markup Language; the markup often returned in an HTTP response body. |
| SNI | Server Name Indication; the TLS extension that sends the requested server name during the handshake. |

## Processes, files, and terminal output

| Term | Full form or meaning |
|---|---|
| PID / PPID | Process identifier / parent process identifier; `ps` uses these output fields. |
| UID / GID | User identifier / group identifier; numeric account and group IDs. |
| FD | File descriptor; a process's handle for an open file or socket. |
| STAT | Process state/status in `ps` output; individual letters vary by system. |
| SIGTERM / SIGINT / SIGTSTP | Termination signal / interrupt signal / terminal stop signal. Their effects depend on the receiving process and its signal handling. |
| stdin / stdout / stderr | Standard input / standard output / standard error. |
| ASCII | American Standard Code for Information Interchange; an older character encoding. |
| UTF-8 | Unicode Transformation Format, 8-bit code units; a Unicode encoding with variable bytes per character. |
| ACL | Access control list; additional file-permission entries beyond basic mode bits. |
| GMT | Greenwich Mean Time; a time zone label in example HTTP output. |
| BSD | Berkeley Software Distribution; macOS command implementations often differ from GNU versions. |
| GNU | GNU's Not Unix; the recursive name of the GNU project. |
| APT / PPA | Advanced Package Tool / Personal Package Archive; Ubuntu package-management terms. |

## Course and platform terms

| Term | Full form or meaning |
|---|---|
| API | Application programming interface. |
| CLI | Command-line interface. |
| CI | Continuous integration; automated checks on code changes. |
| CD | Continuous delivery or continuous deployment, depending on whether release to production is automatic. |
| IaC | Infrastructure as code; infrastructure configuration stored and reviewed as code. |
| CPU | Central processing unit. |
| SDK | Software development kit. |
| SRE | Site reliability engineering; “site reliability engineer” when referring to a person. |
| SLI / SLO | Service level indicator / service level objective; a measurement and its target. |
| PromQL | Prometheus Query Language. |
| CNCF | Cloud Native Computing Foundation. |
| OWASP | Open Worldwide Application Security Project. |
| OTel | Short name for OpenTelemetry; OpenTelemetry itself is a project name. |
| kind | Kubernetes in Docker; a local Kubernetes cluster tool. |
| README / `.md` | “Read me” document / Markdown filename extension. |

## Command names

| Command | Name or purpose |
|---|---|
| `cd`, `pwd`, `ls` | Change directory; print working directory; list directory contents. |
| `mkdir`, `rmdir`, `rm` | Make directory; remove directory; remove files or directories as allowed by the chosen options. |
| `cat`, `cp`, `mv` | Concatenate file contents; copy; move or rename. |
| `printf`, `echo` | Print formatted text; print text. |
| `find`, `sort`, `uniq`, `wc` | Find filesystem entries; sort lines; filter adjacent duplicate lines; word count (also counts lines and bytes with options). |
| `less`, `man` | Page through text; open a manual page. |
| `id`, `whoami`, `ps` | Show identity; show current username (“who am I”); process status. |
| `chmod`, `chown`, `umask` | Change file mode bits; change ownership; set the file-creation permission mask. |
| `mktemp` | Make a temporary file or directory. |
| `bg`, `fg`, `kill`, `wait` | Resume a job in the background; bring it to the foreground; send a signal; wait for a child job. |
| `lsof` | List open files, including network sockets. |
| `dig` | DNS lookup utility. A historical name expansion is sometimes given, but the official manual describes its function. |
| `curl` | URL transfer tool. Its project describes several wordplays for the name, so treat this as a purpose rather than a unique acronym expansion. |
| `tail` | Print the end of a file or stream. |
| `awk` | Pattern-and-action text processor named for its authors Aho, Weinberger, and Kernighan. |
| `su`, `sudo` | Switch user in a shell; run a command with another user's privileges (usually the administrator by default). |
| `sh`, `zsh` | Shell; Z shell. |
| `kubectl` | Kubernetes command-line control tool. |

## Option examples from Week 1

Options belong to a particular command. The day guides explain each option next to the exercise that uses it.

| Option | Meaning |
|---|---|
| `curl -i` | Include/show response headers with the body; `--include` works on older curl, and current curl documentation calls this `--show-headers`. |
| `curl -I` | `--head`: make an HTTP HEAD request. Capital `I` matters. |
| `curl -v` | `--verbose`: show connection and transfer details. |
| `ps -p`, `ps -o` | Select a process by PID; choose output columns. |
| `lsof -n`, `lsof -P` | Keep network addresses numeric; keep port numbers numeric. |
| `lsof -i`, `lsof -s` | Select network files; select a socket state with a value such as `TCP:LISTEN`. |
| `wc -l`, `wc -c` | Count newline characters as lines; count bytes. `-c` is not a character count. |
| `ls -1`, `ls -a` | One entry per line; include hidden entries. `-1` is the digit one. |

## Written references

- [Ubuntu Linux command line tutorial](https://documentation.ubuntu.com/desktop/en/latest/tutorial/the-linux-command-line-for-beginners/) and [GNU Coreutils manual](https://www.gnu.org/software/coreutils/manual/) for file and text commands.
- [curl manual](https://curl.se/docs/manpage.html), [curl FAQ](https://curl.se/docs/faq.html), [lsof documentation](https://github.com/lsof-org/lsof/blob/master/docs/index.md), and [BIND `dig` manual](https://bind9.readthedocs.io/en/latest/manpages.html#dig-dns-lookup-utility) for command names and options.
- [IETF DNS terminology](https://www.rfc-editor.org/rfc/rfc9499.html), [TCP specification](https://www.rfc-editor.org/rfc/rfc9293.html), [TLS specification](https://www.rfc-editor.org/rfc/rfc8446.html), and [HTTP semantics](https://www.rfc-editor.org/rfc/rfc9110.html) for networking terms.
- [Google SRE: Service Level Objectives](https://sre.google/sre-book/service-level-objectives/), [OpenTelemetry overview](https://opentelemetry.io/docs/what-is-opentelemetry/), and [OWASP Foundation](https://owasp.org/about/) for platform terms.
