# Week 1 — Linux and Networking

This week builds a troubleshooting model from files and processes up to ports, DNS, HTTP, and logs. Work through one day at a time. Each lab explains its commands before asking you to run them. Keep your command output and short answers in a personal notes file; the exercises should be performed by you in a terminal.

In command examples, a `$` at the start of a line marks a shell prompt. Do not type that leading `$`; type the command after it and press Enter once per line. Other `$` characters inside a command, such as `$!` or `$lab_sleep_pid`, are shell syntax and must be typed. Keep using the same terminal unless an exercise explicitly names Terminal A or Terminal B.

Each exercise includes an **Expected result** subsection, and each question set includes an **Answer key**. Expected results repeat the exercise commands as reading transcripts: `$` marks a command, and the lines immediately below it show representative output. Consecutive command lines mean the earlier command normally prints nothing. Bracketed notes mark omitted output. These examples are for reading, not for pasting as commands, and do not record completed exercises. Sample PIDs, UIDs, timestamps, permissions, DNS addresses, headers, and diagnostics vary; `learner` is a generic sample user. Terminal A/B labels show where each transcript belongs. A written exercise has a labeled sample response instead of a terminal transcript. Reading an example does not mark an exercise complete.

## Prerequisites

Days 2–5 work on this macOS laptop with a shell and `python3`, `curl`, `dig`, and `lsof`. Day 4 requires internet access. The exercises use a harmless `sleep` process and a loopback Python HTTP server; a Go API is not required for Week 1. Days 3 and 5 use two terminal tabs, with Ctrl-C in the server tab for cleanup.

## Lab locations

Each day folder contains its study guide as `README.md` alongside any `workspace` data. From the project root, `cd labs/01-linux/day-01` enters Day 1's folder; open its `README.md` in your editor or use `less README.md` (press q to quit). Substitute another day number for that day's guide. To return to the project root from a day folder, use `cd ../../..`.

The exercise commands intentionally use paths relative to the project root. Follow the working-directory instruction in each exercise, including any explicit `cd`, before running its commands. Opening a guide from its day folder does not change the directory those commands expect.

## Day progress

- [ ] Day 1 — Shell, paths, files, pipes, and redirection; completed core plus pending extensions: [lab](day-01/README.md)
- [x] Day 2 — Users, permissions, and processes: [lab](day-02/README.md)
- [ ] Day 3 — Ports and TCP connections: [lab](day-03/README.md)
- [ ] Day 4 — DNS, HTTP, and TLS: [lab](day-04/README.md)
- [ ] Day 5 — Logs and first-response troubleshooting: [lab](day-05/README.md)

Day 1 now covers the entire Ubuntu beginner tutorial over multiple self-paced sessions. Its original Core Day 1 exercises and answers remain checked; the expanded day is pending until the new extensions are practiced and reviewed. All Day 1 material stays in its lab file.

Mark a day complete after its exercises and answers have been reviewed.

The full goals and reading list are in the [12-week course](../../roadmap/12-week.md#week-1--linux-and-networking).
