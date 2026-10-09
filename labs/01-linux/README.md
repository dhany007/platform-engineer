# Week 1 — Linux and Networking

This week builds a troubleshooting model from files and processes up to ports, DNS, HTTP, and logs. Work through one day at a time. Each lab explains its commands before asking you to run them. Keep your command output and short answers in a personal notes file; the exercises should be performed by you in a terminal.

In command examples, a `$` at the start of a line marks a shell prompt. Do not type that leading `$`; type the command after it and press Enter once per line. Other `$` characters inside a command, such as `$!` or `$lab_sleep_pid`, are shell syntax and must be typed. Keep using the same terminal unless an exercise explicitly names Terminal A or Terminal B.

Each exercise includes an **Expected result** subsection, and each question set includes an **Answer key**. Expected results repeat the exercise commands as reading transcripts: `$` marks a command, and the lines immediately below it show representative output. Consecutive command lines mean the earlier command normally prints nothing. Bracketed notes mark omitted output. These examples are for reading, not for pasting as commands, and do not record completed exercises. Sample PIDs, UIDs, timestamps, permissions, DNS addresses, headers, and diagnostics vary; `learner` is a generic sample user. Terminal A/B labels show where each transcript belongs. A written exercise has a labeled sample response instead of a terminal transcript. Reading an example does not mark an exercise complete.

## Prerequisites

Days 2–5 work on this macOS laptop with a shell and `python3`, `curl`, `dig`, and `lsof`. Day 4 requires internet access. The exercises use a harmless `sleep` process and a loopback Python HTTP server; a Go API is not required for Week 1. Days 3 and 5 use two terminal tabs, with Ctrl-C in the server tab for cleanup.

## Day progress

- [x] Day 1 — Shell, paths, files, pipes, and redirection: [lab](day-01.md)
- [x] Day 2 — Users, permissions, and processes: [lab](day-02.md)
- [ ] Day 3 — Ports and TCP connections: [lab](day-03.md)
- [ ] Day 4 — DNS, HTTP, and TLS: [lab](day-04.md)
- [ ] Day 5 — Logs and first-response troubleshooting: [lab](day-05.md)

Mark a day complete after its exercises and answers have been reviewed.

The full goals and reading list are in the [12-week course](../../roadmap/12-week.md#week-1--linux-and-networking).
