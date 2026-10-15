# Day 2 — Users, Permissions, and Processes

**Time:** Self-paced, over additional sessions. The completed core took about 60–90 minutes; work through the extensions in separate sessions, pausing whenever needed.

In the command blocks below, the `$` at the start of each line is a prompt marker, not part of the command. Type the text after it and press Enter once per line. Other `$` characters inside a command, such as `$!` and `$lab_sleep_pid`, must be typed. Use the same terminal throughout this day.

Flag meanings in parentheses are memory aids, not invented long-option aliases. BSD (**Berkeley Software Distribution**)/macOS tools may not support GNU (**GNU’s Not Unix**, a recursive acronym) long options; use only the options documented for the installed command.

Command names, abbreviations, and option meanings are also collected in the [glossary](../../../notes/glossary.md).

## Progress

- [x] Exercise 1 — Inspect your identity and a file's permissions.
- [x] Exercise 2 — Start and inspect a harmless process.
- [x] Exercise 3 — Request termination and restart the process.
- [x] Check your understanding — Answer the three questions below.
- [x] Core Day 2 complete — Exercises 1–3 and their answers reviewed.
- [x] Extension 4 — Default permissions with umask.
- [ ] Extension 5 — Suspend, resume, and stop an owned job.
- [ ] Extension checks — Practice and review both new question sets.
- [ ] Expanded Day 2 complete — Core and extensions reviewed.

## Why this matters

A service runs as a user, needs permission to read its files, and has a process identifier (**PID**, process ID). Inspecting those facts helps explain permission errors and failed startups. This lab uses your own files and a harmless `sleep` process on macOS; no Go application programming interface (**API**) is needed.

## Learning goals

1. Identify your user and group memberships.
2. Read owner, group, and other permissions and change only a lab file.
3. Find a process by PID and distinguish a signal from application cleanup.

## Reading

Read [Ubuntu: Welcome to the terminal](https://ubuntu.com/server/docs/tutorial/welcome-to-the-terminal/) for terminal and permission basics. In the [Linux signal manual](https://man7.org/linux/man-pages/man7/signal.7.html), read signal dispositions and the entries for SIGINT (**signal interrupt**) and SIGTERM (**signal terminate**). The signal concepts apply here; some Linux commands and output differ from macOS.

## Core Day 2 — Already completed

Exercises 1–3 and their checked criteria remain complete. The new extensions below are pending; complete and review them before marking the expanded day complete.

## Exercise 1 — Inspect your identity and a file's permissions

Before running the commands:

- `cd` (**change directory**) enters this checkout using its absolute path. Use the same terminal for all today's exercises.
- `id` (**identity**) prints your user identifier (**UID**, user ID), primary group identifier (**GID**, group ID), and group memberships.
- `mkdir` (**make directory**) with `-p` uses `-p` (parents) to create the workspace, including missing parents, and tolerates an existing directory.
- `printf` (**formatted print**) writes sample text; `\n` adds a newline. `>` creates or overwrites only the named lab file.
- `chmod` means **change mode** (permission bits); `chmod 600` gives the owner read (4) plus write (2) permission and gives the group and others no permissions (0). These three digits represent owner, group, and others.
- `ls` (**list**) with `-l` uses `-l` (long listing) to show file details. The leading `-` means a regular file; `rw-------` means owner read/write only. macOS may append an extra marker for metadata.
- `cat` (**concatenate**) joins file contents in sequence; here it reads the single file. Your user owns it, so reading should succeed.
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
- `ps` means **process status**; `ps -p` uses `-p` (select PID) to select that process. `-o pid,ppid,user,stat,command` uses `-o` (output format) to select columns for PID, parent process ID (**PPID**), user, process state (**STAT**, status), and command. State details vary by operating system (**OS**); a waiting process commonly shows `S`.

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
- `kill` sends a signal to a process; `-TERM` selects the termination signal. `kill -TERM` sends SIGTERM to that PID. SIGTERM requests termination; its default action is to terminate. Graceful cleanup requires the application to handle the signal and finish work. Sending SIGTERM alone does not prove that cleanup happened.
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

## Core completion criteria

- [x] You identified your user and explained the lab file's permission changes.
- [x] You inspected a process you started and observed its replacement PID.
- [x] Both `sleep` processes are stopped.
- [x] You can explain signals and the application's role in graceful cleanup.

## Extension 4 — Default permissions with umask

`umask` is the **file-creation permission mask**; it controls which permission bits are cleared when files or directories are created. A file's initial permissions depend on the creating program and its process mask. Changing the mask affects later creations, while `chmod` changes an existing file. This explains why an application's new logs can have a different mode from a file you edited yesterday.

**Written reading:** [Apple: umask(2)](https://developer.apple.com/library/archive/documentation/System/Conceptual/ManPages_iPhoneOS/man2/umask.2.html) and [zsh: umask builtin](https://zsh.sourceforge.io/Doc/Release/Shell-Builtin-Commands.html#index-umask). Ordinary shell redirection requests mode `666` for a new regular file; a mask clears selected bits. Mask `077` normally yields `600`, and `022` yields `644`. This is bit masking, not general decimal subtraction. A mask never adds execute permission. Access control lists (**ACLs**) and filesystem policies can affect observed access; record any extra permission markers.

Think of `666` as the permissions requested for each **new** regular file: owner, group, and others each start with read/write. The mask removes permissions from that request:

| Creating shell's mask | Bits cleared | New file mode in this lab |
| --- | --- | --- |
| `077` | Group and others lose read/write. | `600` (`rw-------`) |
| `022` | Group and others lose write, but keep read. | `644` (`rw-r--r--`) |

Each `zsh -f -c '...'` runs in a separate **child shell**, so its `umask` change affects only files that child creates; your original terminal retains its own mask. `mktemp -d` gives each run a fresh directory, and the two file names do not exist yet. This matters because `>` on an **existing** file replaces its contents but normally keeps its existing permission mode; that would hide the effect of the creation mask. After creation, changing a mask does not alter either file. Use `chmod` to change an existing file's mode.

Before running the commands from the project root:

- `cd` enters the checkout. `mkdir -p` creates the extension parent and tolerates its existence.
- `umask` without arguments prints the current shell's mask. Compare its first and last output; the extension does not change it.
- `mktemp` means **make temporary**; it creates a file or directory with a unique name. `lab_mask_dir=$(mktemp -d .../run.XXXXXX)` stores a newly created directory path. `$(...)` captures command output, `-d` (directory) requests a directory, and `XXXXXX` supplies a unique suffix template. Each run therefore creates genuinely new files instead of overwriting files with existing modes. If `mktemp` reports an error, stop here and resolve it before continuing; do not run file commands with an empty directory variable.
- `zsh` means **Z shell**. Each `zsh -f -c '...'` invokes a separate child shell (a subshell environment). `-f` means skip ordinary startup files; `-c` means command string and executes the quoted script. `mask-lab` supplies its `$0` label; the following directory argument becomes `$1` inside that child. Single quotes keep the parent from expanding `$1`.
- Inside each child, `umask 077` or `umask 022` sets only that child's mask. A semicolon separates its two commands. `printf` writes text ending with `\n`; `>` creates the named new file inside the quoted `$1` directory. Both children exit afterward.
- `ls -l` shows file modes. `chmod 640` changes the already-created private file to owner read/write, group read, and no other access. The other file remains `644`.
- `rm` (**remove**) removes only the two explicitly named lab files; `rmdir` (**remove directory**) removes their now-empty unique directory. Neither operates on the core workspace.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-02/extensions/umask
$ umask
$ lab_mask_dir=$(mktemp -d labs/01-linux/day-02/extensions/umask/run.XXXXXX)
$ zsh -f -c 'umask 077; printf "private\n" > "$1/private.txt"' mask-lab "$lab_mask_dir"
$ zsh -f -c 'umask 022; printf "public\n" > "$1/public.txt"' mask-lab "$lab_mask_dir"
$ ls -l "$lab_mask_dir/private.txt" "$lab_mask_dir/public.txt"
$ chmod 640 "$lab_mask_dir/private.txt"
$ ls -l "$lab_mask_dir/private.txt" "$lab_mask_dir/public.txt"
$ umask
$ rm "$lab_mask_dir/private.txt" "$lab_mask_dir/public.txt"
$ rmdir "$lab_mask_dir"
```

### Expected result

The starting mask, random directory suffix, owner, metadata markers, and timestamps vary. Here the parent starts and ends at `022`; the creation modes differ as intended.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-02/extensions/umask
$ umask
022
$ lab_mask_dir=$(mktemp -d labs/01-linux/day-02/extensions/umask/run.XXXXXX)
$ zsh -f -c 'umask 077; printf "private\n" > "$1/private.txt"' mask-lab "$lab_mask_dir"
$ zsh -f -c 'umask 022; printf "public\n" > "$1/public.txt"' mask-lab "$lab_mask_dir"
$ ls -l "$lab_mask_dir/private.txt" "$lab_mask_dir/public.txt"
-rw-------  1 learner staff 8 Oct 12 10:00 labs/01-linux/day-02/extensions/umask/run.A1b2C3/private.txt
-rw-r--r--  1 learner staff 7 Oct 12 10:00 labs/01-linux/day-02/extensions/umask/run.A1b2C3/public.txt
$ chmod 640 "$lab_mask_dir/private.txt"
$ ls -l "$lab_mask_dir/private.txt" "$lab_mask_dir/public.txt"
-rw-r-----  1 learner staff 8 Oct 12 10:00 labs/01-linux/day-02/extensions/umask/run.A1b2C3/private.txt
-rw-r--r--  1 learner staff 7 Oct 12 10:00 labs/01-linux/day-02/extensions/umask/run.A1b2C3/public.txt
$ umask
022
$ rm "$lab_mask_dir/private.txt" "$lab_mask_dir/public.txt"
$ rmdir "$lab_mask_dir"
```

### Checks and answer key

1. [x] **Check:** Why did the new files start at different modes? **Answer:** Each creating child shell used a different mask to clear bits from the requested creation mode.
2. [x] **Check:** Does a later `umask 077` change `public.txt` to `600`? **Answer:** No. Existing-file mode changes require an operation such as `chmod`; changing creation defaults is separate.
3. [x] **Check:** Why use separate children and fresh filenames? **Answer:** Children isolate the mask change from your interactive shell, while fresh files let you observe creation rather than truncation of an existing file.

**Cleanup and rerun:** The last two commands remove only this run's files/directory. Keep the empty extension parent if desired. If interrupted, inspect the saved directory path and finish those named cleanup commands in the same shell before repeating. A rerun gets a fresh suffix; do not reuse a path from another session or remove a directory you did not create.

## Extension 5 — Suspend, resume, and stop an owned job

Suspending a job leaves it alive but stopped. Running it in the background lets the shell accept commands, and bringing it forward gives it control of terminal input. This differs from terminating a process.

**Written reading:** [zsh: Jobs and Signals](https://zsh.sourceforge.io/Doc/Release/Jobs-_0026-Signals.html). A shell job can contain several processes; the job number is local to that shell, while a PID identifies an OS process. This practice has one process per job.

Use an interactive terminal, not a script runner. Before the commands:

- `cd` enters the checkout and `mkdir -p` establishes the isolated `extensions/jobs` location. No file is required for this process exercise.
- `zsh -f` opens a nested interactive zsh with ordinary startup configuration skipped and a new job table. Keep using this nested shell until `exit`; all sample job numbers below refer to it.
- `sleep 600` waits up to ten minutes in the foreground. While it runs, press **Ctrl-Z** once. This normally sends SIGTSTP (**signal terminal stop**) and stops the job; it does not kill it. The displayed `^Z` is a keypress representation, not a command.
- `jobs -l` uses `-l` (long listing) to list this shell's jobs with their PIDs. Confirm the stopped entry is your `sleep 600` before proceeding.
- `bg` means **background**; `bg %1` resumes job 1 in the background; `%1` names a shell job, not PID 1. If the job number differs, replace `%1` in both `bg` and `fg` with the observed number.
- `fg` means **foreground**; `fg %1` brings that job into the foreground. Press **Ctrl-C** once while it is running to interrupt this owned `sleep` with SIGINT. Then `jobs -l` should have no remaining lab job.
- `exit` leaves the nested shell and returns to the original shell. Do this only after stopping the lab job.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-02/extensions/jobs
$ zsh -f
$ sleep 600
$ jobs -l
$ bg %1
$ jobs -l
$ fg %1
$ jobs -l
$ exit
```

### Expected result

Read the key actions above before following the commands: Ctrl-Z after starting sleep, then Ctrl-C after `fg`. Job numbers, PID 30790, prompts, and notification timing are illustrative. No output from the final `jobs` means the lab job is gone.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-02/extensions/jobs
$ zsh -f
$ sleep 600
^Z
zsh: suspended  sleep 600
$ jobs -l
[1]  + 30790 suspended  sleep 600
$ bg %1
[1]  + continued  sleep 600
$ jobs -l
[1]  + 30790 running    sleep 600
$ fg %1
[1]  + running    sleep 600
^C
$ jobs -l
$ exit
```

### Checks and answer key

1. **Check:** What distinguishes job 1 from PID 30790? **Answer:** Job 1 is an entry in this shell's job table; 30790 identifies the process at the OS level.
2. **Check:** Did Ctrl-Z terminate the process? **Answer:** No. It stopped execution until a resume operation such as `bg` or `fg`.
3. **Check:** How did cleanup differ from backgrounding? **Answer:** `bg` continued the job; foreground Ctrl-C interrupted it so it exited, leaving no lab entry in `jobs`.

**Cleanup and rerun:** If interrupted while the job is stopped or backgrounded, inspect `jobs -l`, select only your lab `sleep` with `fg %<observed-job-number>`, then Ctrl-C. This placeholder is not a literal command to paste. Do not act on unrelated jobs. If ten minutes pass and sleep exits, start a fresh one and repeat the key actions. Rerun only after the previous job is gone; the empty extension directory can remain. A shell-close warning about jobs means cleanup still needs attention.

## Expanded Day 2 completion criteria

- [x] Core Exercises 1–3 and their original criteria remain reviewed.
- [x] Extension 4 practiced: creation masks compared, parent mask preserved, and named files cleaned up.
- [ ] Extension 5 practiced: owned job suspended, resumed, foregrounded, and stopped.
- [ ] Both extension question sets reviewed, including creation versus chmod and job number versus PID.
- [ ] Expanded Day 2 complete — All extension practice and cleanup reviewed.
