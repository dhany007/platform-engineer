# Day 2 — Users, Permissions, and Processes

**Time:** About 60–90 minutes.

In the command blocks below, the `$` at the start of each line is a prompt marker, not part of the command. Type the text after it and press Enter once per line. Other `$` characters inside a command, such as `$!` and `$lab_sleep_pid`, must be typed. Use the same terminal throughout this day.

## Progress

- [x] Exercise 1 — Inspect your identity and a file's permissions.
- [x] Exercise 2 — Start and inspect a harmless process.
- [x] Exercise 3 — Request termination and restart the process.
- [x] Check your understanding — Answer the three questions below.
- [x] Day 2 complete — All exercises and answers reviewed.

## Why this matters

A service runs as a user, needs permission to read its files, and has a process ID (PID). Inspecting those facts helps explain permission errors and failed startups. This lab uses your own files and a harmless `sleep` process on macOS; no Go API is needed.

## Learning goals

1. Identify your user and group memberships.
2. Read owner, group, and other permissions and change only a lab file.
3. Find a process by PID and distinguish a signal from application cleanup.

## Reading

Read [Ubuntu: Welcome to the terminal](https://ubuntu.com/server/docs/tutorial/welcome-to-the-terminal/) for terminal and permission basics. In the [Linux signal manual](https://man7.org/linux/man-pages/man7/signal.7.html), read signal dispositions and the entries for SIGINT and SIGTERM. The signal concepts apply here; some Linux commands and output differ from macOS.

## Exercise 1 — Inspect your identity and a file's permissions

Before running the commands:

- `cd` enters this checkout using its absolute path. Use the same terminal for all today's exercises.
- `id` prints your user ID, primary group, and group memberships.
- `mkdir -p` creates the workspace, including missing parents, and tolerates an existing directory.
- `printf` writes sample text; `\n` adds a newline. `>` creates or overwrites only the named lab file.
- `chmod 600` gives the owner read (4) plus write (2) permission and gives the group and others no permissions (0). These three digits represent owner, group, and others.
- `ls -l` shows file details. The leading `-` means a regular file; `rw-------` means owner read/write only. macOS may append an extra marker for metadata.
- `cat` reads the file. Your user owns it, so reading should succeed.
- `chmod 644` keeps owner read/write and gives group and others read permission. File execute permission (`x`, value 1) is unnecessary for this text file. Directory execute permission instead controls traversal.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ id
$ mkdir -p labs/01-linux/day-02/workspace
$ printf 'service=practice\n' > labs/01-linux/day-02/workspace/sample.conf
$ chmod 600 labs/01-linux/day-02/workspace/sample.conf
$ ls -l labs/01-linux/day-02/workspace/sample.conf
$ cat labs/01-linux/day-02/workspace/sample.conf
$ chmod 644 labs/01-linux/day-02/workspace/sample.conf
$ ls -l labs/01-linux/day-02/workspace/sample.conf
```

Record the owner and group, and compare the two permission strings. Both reads are allowed for the owner.

### Expected result

UID 501 and the user/group names are illustrative. Group memberships, timestamps, and macOS permission markers vary. The mode changes from owner-only read/write to also allowing group/other reads.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ id
uid=501(learner) gid=20(staff) groups=20(staff),12(everyone)
$ mkdir -p labs/01-linux/day-02/workspace
$ printf 'service=practice\n' > labs/01-linux/day-02/workspace/sample.conf
$ chmod 600 labs/01-linux/day-02/workspace/sample.conf
$ ls -l labs/01-linux/day-02/workspace/sample.conf
-rw-------  1 learner staff 17 Oct  9 10:00 labs/01-linux/day-02/workspace/sample.conf
$ cat labs/01-linux/day-02/workspace/sample.conf
service=practice
$ chmod 644 labs/01-linux/day-02/workspace/sample.conf
$ ls -l labs/01-linux/day-02/workspace/sample.conf
-rw-r--r--  1 learner staff 17 Oct  9 10:00 labs/01-linux/day-02/workspace/sample.conf
```

## Exercise 2 — Start and inspect a harmless process

- `sleep 600` waits for 600 seconds and then exits. `&` runs it in the background so your shell stays available.
- `lab_sleep_pid=$!` stores the PID of the most recent background process in a shell variable. No spaces go around `=`.
- `printf` prints that PID using `%s` as a text placeholder; `"$lab_sleep_pid"` expands the variable as one argument.
- `ps -p` selects that PID. `-o pid,ppid,user,stat,command` selects columns for PID, parent PID, user, state, and command. State details vary by OS; a waiting process commonly shows `S`.

```console
$ sleep 600 &
$ lab_sleep_pid=$!
$ printf 'Lab PID: %s\n' "$lab_sleep_pid"
$ ps -p "$lab_sleep_pid" -o pid,ppid,user,stat,command
```

Confirm the command is `sleep 600` and the user matches Exercise 1. If you take longer than ten minutes, repeat this exercise to create a fresh process and PID before continuing.

### Expected result

PID 30790 and parent PID 30700 are sample values. Job notifications and state flags vary. The saved PID, printed PID, and selected process row agree.

```console
$ sleep 600 &
[1] 30790
$ lab_sleep_pid=$!
$ printf 'Lab PID: %s\n' "$lab_sleep_pid"
Lab PID: 30790
$ ps -p "$lab_sleep_pid" -o pid,ppid,user,stat,command
  PID  PPID USER     STAT COMMAND
30790 30700 learner  S    sleep 600
```

## Exercise 3 — Request termination and restart the process

First inspect the saved PID again and confirm it still belongs to your `sleep 600`. If it has exited, repeat Exercise 2 before sending a signal. Never substitute a PID belonging to another process.

- `ps` uses the same selection and columns as above to confirm the target.
- `kill -TERM` sends SIGTERM to that PID. SIGTERM requests termination; its default action is to terminate. Graceful cleanup requires the application to handle the signal and finish work. Sending SIGTERM alone does not prove that cleanup happened.
- `wait` waits for this shell's background child and collects its exit status. A termination message or nonzero status is expected after the signal.
- The next `ps` should show only a header or report no matching process.
- `sleep`, `&`, and `$!` start a replacement and save its new PID. The final signal and `wait` clean up that replacement.

```console
$ ps -p "$lab_sleep_pid" -o pid,ppid,user,stat,command
$ kill -TERM "$lab_sleep_pid"
$ wait "$lab_sleep_pid"
$ ps -p "$lab_sleep_pid" -o pid,ppid,user,stat,command
$ sleep 600 &
$ lab_sleep_pid=$!
$ ps -p "$lab_sleep_pid" -o pid,ppid,user,stat,command
$ kill -TERM "$lab_sleep_pid"
$ wait "$lab_sleep_pid"
```

Record the original and replacement PIDs. Both lab processes should now be stopped. If you close the terminal before finishing, do not reuse a saved PID in a new session.

### Expected result

The sample replacement PID is 30791. `kill` normally prints nothing; termination notifications can appear before or after `wait`, which may return nonzero without printing. The header-only `ps` shows the first child is gone; the final signal and wait stop the replacement. This demonstrates termination, not application-specific graceful cleanup.

```console
$ ps -p "$lab_sleep_pid" -o pid,ppid,user,stat,command
  PID  PPID USER     STAT COMMAND
30790 30700 learner  S    sleep 600
$ kill -TERM "$lab_sleep_pid"
$ wait "$lab_sleep_pid"
[1]  + terminated  sleep 600
$ ps -p "$lab_sleep_pid" -o pid,ppid,user,stat,command
  PID  PPID USER     STAT COMMAND
$ sleep 600 &
[1] 30791
$ lab_sleep_pid=$!
$ ps -p "$lab_sleep_pid" -o pid,ppid,user,stat,command
  PID  PPID USER     STAT COMMAND
30791 30700 learner  S    sleep 600
$ kill -TERM "$lab_sleep_pid"
$ wait "$lab_sleep_pid"
[1]  + terminated  sleep 600
```

## Check your understanding

1. What do the three digits in `644` mean, and why does this file need no execute permission?
2. How does a PID differ from the command name?
3. Why does sending SIGTERM not guarantee graceful cleanup?

### Answer key

1. `6` gives the owner read/write, and each `4` gives group/others read only. This configuration is data to read, not a program to execute.
2. A PID identifies one running process instance. A command name describes what it runs, and multiple processes can share that name.
3. SIGTERM requests termination. The application must handle it to finish work or clean up gracefully; the default action simply terminates.

**Share back:** Send the permission observations, process output, and your three answers for review.

## Completion criteria

- [x] You identified your user and explained the lab file's permission changes.
- [x] You inspected a process you started and observed its replacement PID.
- [x] Both `sleep` processes are stopped.
- [x] You can explain signals and the application's role in graceful cleanup.
