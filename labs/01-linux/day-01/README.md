# Day 1 — Shell, Paths, and Files

**Time:** Self-paced, over several sessions. The completed core takes about 60–90 minutes; work through the extensions in additional sessions, pausing whenever needed.

Commands start with a `$` prompt marker: type only the text after that leading marker. Expected-result transcripts repeat commands and show representative output; read them rather than pasting them. Silent commands have no output line. Names such as `learner`, timestamps, listings, and pager displays are illustrative.

## Progress

- [x] Exercise 1 — Locate the project.
- [x] Exercise 2 — Create and inspect files.
- [x] Exercise 3 — Pipe and save command output.
- [x] Check your understanding — Answer the three questions below.
- [x] Core Day 1 complete — Exercises 1–3 and their answers reviewed.
- [ ] Extension 4 — Terminal controls, navigation, arguments, and quoting.
- [ ] Extension 5 — Text, wildcards, append, paging, and cancellation.
- [ ] Extension 6 — Copy, move, and rename.
- [ ] Extension 7 — Streams, counts, sorting, uniqueness, and manuals.
- [ ] Extension 8 — Hidden files and filename case.
- [ ] Extension 9 — Inspect and clean up extension files.
- [ ] Extension 10 — Explain administrator access and Ubuntu software tools.
- [ ] Extension checks — Answer and review the new questions.
- [ ] Expanded Day 1 complete — Core and all extensions practiced and reviewed.

## Why this matters

When a service fails, you often need to locate its configuration, inspect a log, and save evidence before changing anything. The shell gives you a repeatable way to do that. Today's commands work in a macOS terminal and in common Linux shells; later days will introduce Linux-specific behavior.

## Learning goals

By the end of this session, you should be able to:

1. Tell the difference between the current directory, an absolute path, and a relative path.
2. Navigate and inspect files with `pwd`, `cd`, `ls`, `find`, and `cat`.
3. Explain what a pipe (`|`), output redirection (`>`), and input redirection (`<`) do.
4. Save command output in a file and inspect it afterward.
5. Quote paths, copy/move files, inspect hidden entries, and clean up named lab files.
6. Use append redirection, wildcards, pagers, manuals, and text pipelines.
7. Distinguish terminal controls and Ubuntu administrator concepts from macOS practice.

## Reading

Read the whole official [Ubuntu: The Linux command line for beginners](https://ubuntu.com/desktop/docs/en/latest/tutorial/the-linux-command-line-for-beginners/), one section at a time alongside this lab. The original practice below covers its full topic range with different sample data and isolated repository workspaces. Ubuntu examples using administrator access or package installation are conceptual here.

### Tutorial coverage and suggested sessions

| Official tutorial topic | Where to study here |
|---|---|
| Opening a terminal, location, absolute/relative paths | Core 1; extension orientation and 4 |
| Creating folders/files, options, quoting | Core 2; Extension 4 |
| Redirection, cancellation, copy/paste, adding/linking text | Core 2–3; orientation and Extension 5 |
| Case and moving/manipulating/deleting files | Extensions 6, 8, and 9 |
| Text plumbing, counts, sorting, uniqueness, manuals | Core 3; Extension 7 |
| Superuser access, sensitive files, installing software | Conceptual Extension 10 |
| Hidden files, cleanup, further reading | Extensions 8–9; further reading below |

One possible pace is Extensions 4–5 in one session, 6–8 in another, then 9–10 and a review. Take more sessions if needed. Each practice extension returns to the checkout and creates its own workspace; cleanup assumes the earlier extensions have been completed.

## Core Day 1 — Already completed

Exercises 1–3 and their answers remain complete. The extensions below add work; you do not need to repeat the core to retain its progress.

## Exercise 1 — Locate the project

Open a terminal and enter the project directory. The path below matches this laptop; adjust it if you use a different checkout.

Before running the commands, know what each one does:

- `cd /Users/citradigital/Desktop/learn/platform-engineer` changes to the project directory using an **absolute path**. A path beginning with `/` starts at the filesystem root.
- `pwd` prints the current working directory, so you can confirm where you are.
- `ls -la` lists entries with details (`-l`) and includes hidden entries (`-a`), such as `.git`.
- `cd labs/01-linux` changes directory using a **relative path**. It is resolved from your current directory.
- `cd ../..` moves up two parent directories; each `..` means one parent.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ pwd
$ ls -la
$ cd labs/01-linux
$ pwd
$ cd ../..
$ pwd
```

### Expected result

The configured checkout path is repeated exactly from the exercise. Listing metadata and entries vary; `learner` is a generic sample owner. The three `pwd` results show root, lab directory, then root again.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ pwd
/Users/citradigital/Desktop/learn/platform-engineer
$ ls -la
total 0
drwxr-xr-x  8 learner staff 256 Oct  9 10:00 .
drwxr-xr-x  3 learner staff  96 Oct  9 09:00 ..
drwxr-xr-x 12 learner staff 384 Oct  9 09:00 .git
drwxr-xr-x  4 learner staff 128 Oct  9 09:00 labs
drwxr-xr-x  3 learner staff  96 Oct  9 09:00 roadmap
[other entries omitted]
$ cd labs/01-linux
$ pwd
/Users/citradigital/Desktop/learn/platform-engineer/labs/01-linux
$ cd ../..
$ pwd
/Users/citradigital/Desktop/learn/platform-engineer
```

## Exercise 2 — Create and inspect files

Run these commands from the project root:

- `mkdir -p .../workspace` creates the target directory and any missing parent directories. Repeating it is safe when the directories already exist.
- `printf 'service=go-api\nport=8080\n'` produces two lines of text. Each `\n` means a new line.
- `>` sends that text to `sample.conf`. It creates the file if needed and overwrites existing contents.
- `cat .../sample.conf` prints the file's contents; `cat` expects a file path here, not a directory path.
- `ls -la .../workspace` confirms the file exists and shows its details.

```console
$ mkdir -p labs/01-linux/day-01/workspace
$ printf 'service=go-api\nport=8080\n' > labs/01-linux/day-01/workspace/sample.conf
$ cat labs/01-linux/day-01/workspace/sample.conf
$ ls -la labs/01-linux/day-01/workspace
```

### Expected result

The file contains 25 bytes of sample configuration; it does not start a service. Permission metadata and timestamps vary.

```console
$ mkdir -p labs/01-linux/day-01/workspace
$ printf 'service=go-api\nport=8080\n' > labs/01-linux/day-01/workspace/sample.conf
$ cat labs/01-linux/day-01/workspace/sample.conf
service=go-api
port=8080
$ ls -la labs/01-linux/day-01/workspace
total 8
drwxr-xr-x 3 learner staff 96 Oct  9 10:00 .
drwxr-xr-x 3 learner staff 96 Oct  9 10:00 ..
-rw-r--r-- 1 learner staff 25 Oct  9 10:00 sample.conf
```

## Exercise 3 — Pipe and save command output

Run these commands from the project root:

- `find roadmap -type f` lists regular files under `roadmap`, including files in subdirectories.
- `| sort` passes those filenames to `sort`, which puts them in alphabetical order.
- `>` saves the sorted list to `roadmap-files.txt`, overwriting any previous contents.
- `cat .../roadmap-files.txt` displays the saved list.
- `wc -l` counts lines. `< .../roadmap-files.txt` supplies the file as input, so the output is only the number.

```console
$ find roadmap -type f | sort > labs/01-linux/day-01/workspace/roadmap-files.txt
$ cat labs/01-linux/day-01/workspace/roadmap-files.txt
$ wc -l < labs/01-linux/day-01/workspace/roadmap-files.txt
```

### Expected result

The first command is silent because output is saved to a file. The saved list contains filenames, and its line count is one in the current repository; adding roadmap files changes both.

```console
$ find roadmap -type f | sort > labs/01-linux/day-01/workspace/roadmap-files.txt
$ cat labs/01-linux/day-01/workspace/roadmap-files.txt
roadmap/12-week.md
$ wc -l < labs/01-linux/day-01/workspace/roadmap-files.txt
       1
```

## Core check your understanding

Answer in your own words:

1. Why can `cd labs/01-linux` fail when run from a different directory?
2. What is the difference between `|` and `>` in Exercise 3?
3. What happens to `sample.conf` if you repeat the `printf ... > sample.conf` command?

### Answer key

1. `labs/01-linux` is relative to the current directory. From another directory, that relative path may name a different location or not exist.
2. `|` passes one command’s output to another command as input. `>` writes output to a file and overwrites its previous contents.
3. The file is overwritten with the same two lines; they are not appended.

**Review status:** The command outputs and answers for Exercises 1–3 have been reviewed. Core Day 1 is complete. The expanded Day 1 remains incomplete until the extensions are practiced and reviewed.

## Core completion criteria

- [x] You can navigate from the project root into `labs/01-linux` and back.
- [x] You created and inspected `sample.conf`.
- [x] You saved a sorted file list and counted its lines.
- [x] You can explain relative paths, pipes, and redirection without looking at the commands.

## Extension orientation — Terminal, shell, and keys

A **terminal** presents text input/output in a window; a **shell** interprets what you type there. macOS commonly starts zsh, while Ubuntu commonly uses Bash. Programs such as `cat` run through the shell; the terminal itself does not decide how a pipe or quote works. Prompts are configurable and may display the user, computer, and directory. A prompt's `$` is a convention, not a character to enter.

Open Terminal on macOS through Spotlight or Applications → Utilities. On Ubuntu, use the application launcher; Ctrl-Alt-T commonly opens a terminal. Keyboard shortcuts depend on the terminal and its settings:

| Action | macOS Terminal | Common Ubuntu terminal |
|---|---|---|
| Copy selected text | Command-C | Ctrl-Shift-C |
| Paste clipboard | Command-V | Ctrl-Shift-V |
| Interrupt foreground command | Ctrl-C | Ctrl-C |
| Recall/edit commands | Up/Down history; Left/Right to move; Backspace to erase | Same common shell controls |
| Complete a path or command | Tab, when a completion is available | Tab, when a completion is available |
| Send end-of-input | Ctrl-D on an empty input line | Ctrl-D on an empty input line |

Ctrl-C sends SIGINT to the foreground job; a program may handle it, so cancellation is not a universal cleanup guarantee. Ctrl-D sends end-of-input rather than a termination signal; at an empty shell prompt it may exit the shell. `exit` also ends a shell session; a terminal tab may close depending on settings. Use these only after saving notes and finishing cleanup. Do not paste output, prompt markers, or pager instructions as shell commands. Read and paste one command at a time; a pasted newline can run it immediately.

Paths use `/` as a separator. `/` names the filesystem root directory; **root the user** is a privileged account. Navigating to `/` does not change your user. Linux commonly stores home directories under `/home`; macOS uses `/Users`. The `~` prefix expands to your home directory when unquoted; `.` is the current directory and `..` its parent. A bare `cd` goes home. The exercises below return explicitly to this checkout before writing files.

## Extension 4 — Navigation, arguments, options, and quoting

**Progress:** Leave Extension 4 unchecked until you practice and review it. This extension uses only `day-01/extensions/paths` for created directories.

Before the commands:

- `cd /` visits the root directory; `pwd` prints the current location. `cd` without an argument and `cd ~` both go home. The final absolute `cd` returns to the checkout.
- `whoami` prints your effective username; the example output uses `learner`.
- `mkdir -p` creates missing parent directories and permits existing directories. Multiple paths after it are separate arguments. `branch/leaf` creates nesting; `inbox archive` creates sibling directories.
- `"release notes"` and `'team notes'` each keep a space inside a single argument. `escaped\ notes` protects just that space with a backslash. Unquoted `release notes` would instead be two arguments. Single quotes preserve literal text; double quotes still allow shell expansions such as `$variable`.
- `cd labs/01-linux/day-01/extensions/paths` uses a relative path from the checkout. `ls -1` uses the digit one to list one entry per line, making output easy to compare.

Options often use a short form (`-p`), separate short forms (`-l -a`), or combined flags (`-la`). GNU tools on Ubuntu also commonly provide long forms such as `--parents` for `mkdir -p`; macOS tools do not necessarily accept GNU long options. Command documentation determines which forms are valid. `--` often ends option parsing when a filename begins with `-`, but check the specific command. A missing argument can print usage; not every shell accepts the same `cd` arguments. These are syntax errors, not requests for administrator access.

```console
$ cd /
$ pwd
$ cd
$ pwd
$ cd ~
$ whoami
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-01/extensions/paths/branch/leaf
$ cd labs/01-linux/day-01/extensions/paths
$ mkdir -p inbox archive "release notes" 'team notes' escaped\ notes
$ ls -1
```

### Expected result

Home paths and usernames vary. Directory names below are the intended first-run entries; later repeats may show additional work you have created.

```console
$ cd /
$ pwd
/
$ cd
$ pwd
/Users/learner
$ cd ~
$ whoami
learner
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-01/extensions/paths/branch/leaf
$ cd labs/01-linux/day-01/extensions/paths
$ mkdir -p inbox archive "release notes" 'team notes' escaped\ notes
$ ls -1
archive
branch
escaped notes
inbox
release notes
team notes
```

### Check and answer key

1. **Check:** Does `cd /` make you root? **Answer:** No; it changes the working directory, not your identity.
2. **Check:** Why is `release notes` one directory here? **Answer:** Quotes protect the space so the shell passes one pathname argument.
3. **Check:** Can you assume `--parents` works on macOS? **Answer:** No. This practice uses the portable `-p` form; consult the installed command's manual for supported options.

## Extension 5 — Text, wildcards, overwrite, append, and paging

This extension is independent of Extension 4. All files go in `extensions/text`. Before the commands:

- The absolute `cd` returns to the checkout; `mkdir -p` creates a dedicated workspace; the next `cd` enters it.
- `echo '...'` prints simple literal text and a newline. For predictable formatting and escape sequences, use `printf`, as in the core exercises. No `echo` options are needed here.
- `>` creates or truncates the destination before writing. `>>` creates or appends, preserving earlier text. The second write to `session.log` deliberately replaces its first line.
- `cat` with multiple paths concatenates their contents in argument order. The shell expands unquoted `*` to zero or more filename characters and `?` to exactly one. Here `part-*.txt` and `part-?.txt` select the two files you create. Avoid output filenames matching the input pattern: redirection can truncate a file before it is read.
- A quoted wildcard stays literal rather than expanding. Wildcards normally omit leading-dot files. When nothing matches, zsh typically reports `no matches found`; Bash's defaults may pass the pattern to the command unchanged.
- `less combined.txt` opens a pager without altering the file. Use arrows or Space to move, `/name` to search, `n` for another match, and **q to quit before the next command**. `less` can read piped input too. `more` is another pager you may encounter; this practice uses `less`.
- The final bare `cat` reads **standard input** instead of a file and waits for keyboard input. Press **Ctrl-C immediately** to interrupt it; do not type `quit` as a shell command. The transcript's `^C` represents that key action.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-01/extensions/text
$ cd labs/01-linux/day-01/extensions/text
$ echo 'north' > part-a.txt
$ echo 'south' > part-b.txt
$ cat part-a.txt part-b.txt
$ cat part-*.txt > combined.txt
$ cat part-?.txt
$ echo 'first attempt' > session.log
$ echo 'replacement' > session.log
$ echo 'follow-up' >> session.log
$ cat session.log
$ less combined.txt
$ cat
```

### Expected result

The `less` display is shown as text for reading; its status line and screen layout vary. Press q to leave it, then Ctrl-C after the final bare `cat`.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-01/extensions/text
$ cd labs/01-linux/day-01/extensions/text
$ echo 'north' > part-a.txt
$ echo 'south' > part-b.txt
$ cat part-a.txt part-b.txt
north
south
$ cat part-*.txt > combined.txt
$ cat part-?.txt
north
south
$ echo 'first attempt' > session.log
$ echo 'replacement' > session.log
$ echo 'follow-up' >> session.log
$ cat session.log
replacement
follow-up
$ less combined.txt
north
south
(END)
$ cat
^C
```

### Check and answer key

1. **Check:** Why is `first attempt` absent? **Answer:** The second `>` truncated `session.log`; `>>` subsequently added only `follow-up`.
2. **Check:** Who expands `part-*.txt`? **Answer:** The shell expands it to matching pathnames before starting `cat`.
3. **Check:** Why does bare `cat` wait, and how do you leave each program? **Answer:** It reads keyboard input through standard input; interrupt it with Ctrl-C. Quit `less` with q.

## Extension 6 — Copy, move, and rename

This extension creates its own files in `extensions/moves`. Before the commands:

- `cd`, `mkdir -p`, and `echo ... > ...` perform the same navigation, workspace creation, and text writing explained above.
- `cp draft.txt backup.txt` creates a separate copy, leaving `draft.txt` intact. Existing destinations can be overwritten, so these destination names belong only to this lab. `cp -R` copies directory trees, but is unnecessary here.
- `mv backup.txt reviewed.txt` renames the copy. `mv draft.txt destination/` moves the original into a directory. Source and destination arguments must be separated by a space; `.`, used later as a destination, means the current directory.
- `ls -1 destination` lists the destination; `cat reviewed.txt destination/draft.txt` reads both files in order.
- `mv destination/draft.txt .` returns that named file to this directory. `mv -i` and `cp -i` request confirmation before overwriting; our new first-run destinations do not need prompts. If repeating this extension, inspect any existing files before approving an overwrite.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-01/extensions/moves/destination
$ cd labs/01-linux/day-01/extensions/moves
$ echo 'review me' > draft.txt
$ cp draft.txt backup.txt
$ mv backup.txt reviewed.txt
$ mv draft.txt destination/
$ ls -1 destination
$ cat reviewed.txt destination/draft.txt
$ mv destination/draft.txt .
$ ls -1
```

### Expected result

A first run leaves two equal-content files plus the empty destination directory. Copying retained the original; renaming and moving changed its location/name rather than its text.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-01/extensions/moves/destination
$ cd labs/01-linux/day-01/extensions/moves
$ echo 'review me' > draft.txt
$ cp draft.txt backup.txt
$ mv backup.txt reviewed.txt
$ mv draft.txt destination/
$ ls -1 destination
draft.txt
$ cat reviewed.txt destination/draft.txt
review me
review me
$ mv destination/draft.txt .
$ ls -1
destination
draft.txt
reviewed.txt
```

### Check and answer key

1. **Check:** Which operation leaves a second file? **Answer:** `cp`; `mv` changes the source's name or location.
2. **Check:** What does the final dot mean? **Answer:** Move into the current directory.
3. **Check:** Where is `backup.txt` afterward? **Answer:** It has been renamed to `reviewed.txt`.

## Extension 7 — Standard streams, pipelines, counts, and manuals

A process normally has standard input (stdin), standard output (stdout), and standard error (stderr). Without redirection, terminal input/output are common defaults. `<` supplies file input, `>`/`>>` redirect stdout, and `|` connects stdout to the next command's stdin. Errors normally stay on stderr rather than traveling through a plain pipe. In a shell, `2>` redirects stderr separately. Pipes transfer bytes through operating-system buffers; they do not require a named intermediate file.

Before the commands:

- `cd` and `mkdir -p` create and enter the separate `extensions/streams` workspace.
- `printf 'blue\nred\nblue\n'` writes three newline-terminated records to `events.txt` via `>`. Here `\n` is interpreted by `printf`'s format string.
- `wc events.txt` reports lines, words, and bytes for this ASCII file. `wc -l < events.txt` counts newlines using file input, so it does not print a filename. `-w` selects words and `-c` selects bytes if needed; `-m` counts characters, which can differ from bytes for non-ASCII text.
- `uniq events.txt` collapses only **adjacent** equal lines; the repeated `blue` lines here are separated. `sort events.txt` arranges the lines so equal ones are adjacent. Sorting can depend on locale; these simple names have the shown common order.
- `sort events.txt | uniq` streams the sorted lines to `uniq`; adding `| wc -l` counts the distinct records. `> unique.txt` saves that same distinct list, and `cat` inspects it.
- `man uniq` opens the locally installed manual, commonly through a pager. Read its NAME, SYNOPSIS, and description of adjacent matches, then press **q** to return. `[argument]` in a synopsis commonly means optional; the brackets are not literal command syntax. Manuals can differ across BSD/macOS and GNU/Linux. `man sort` or `man wc` can clarify their supported options later.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-01/extensions/streams
$ cd labs/01-linux/day-01/extensions/streams
$ printf 'blue\nred\nblue\n' > events.txt
$ wc events.txt
$ wc -l < events.txt
$ uniq events.txt
$ sort events.txt | uniq
$ sort events.txt | uniq | wc -l
$ sort events.txt | uniq > unique.txt
$ cat unique.txt
$ man uniq
```

### Expected result

Whitespace in counts and manual wording vary. The manual display below is abbreviated, not text to type. Quit its pager with q.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-01/extensions/streams
$ cd labs/01-linux/day-01/extensions/streams
$ printf 'blue\nred\nblue\n' > events.txt
$ wc events.txt
       3       3      14 events.txt
$ wc -l < events.txt
       3
$ uniq events.txt
blue
red
blue
$ sort events.txt | uniq
blue
red
$ sort events.txt | uniq | wc -l
       2
$ sort events.txt | uniq > unique.txt
$ cat unique.txt
blue
red
$ man uniq
UNIQ(1)
[manual sections and installed option descriptions displayed in pager]
```

### Check and answer key

1. **Check:** Why does plain `uniq` still output three lines? **Answer:** Its matching records are not adjacent until sorted.
2. **Check:** What does the second pipe carry? **Answer:** `uniq`'s stdout becomes `wc`'s stdin, so `wc` counts distinct lines without a temporary file.
3. **Check:** Why are bytes and characters not always equal? **Answer:** Some characters use multiple bytes in encodings such as UTF-8; the sample uses only ASCII.

## Extension 8 — Hidden entries and case

A leading dot is a naming convention that makes an entry omitted from ordinary listings, not a security control. Home-directory dotfiles often configure individual programs; `/etc` commonly contains system configuration. Merely hiding a file does not deny access. Do not edit existing personal or system configuration for this exercise.

Linux filesystems commonly distinguish `Report.txt` and `report.txt`. Typical macOS volumes preserve case but compare filenames without regard to case; case-sensitive macOS volumes also exist. Avoid creating two files whose names differ only in case. Commands, variables, and options have their own case rules regardless of filesystem behavior: type the exact spelling shown.

Before the commands:

- `cd`, `mkdir -p`, and `echo ... > ...` enter a dedicated workspace and create a normal lab file.
- `mv status.txt .status.txt` gives it a hidden name. `ls -1` omits it; `ls -1a` combines one-entry-per-line (`-1`) and all entries (`-a`), including `.` and `..`.
- `cat .status.txt` can still read it by its explicit pathname. Hidden does not mean unreadable.
- `mkdir -p .settings` creates a hidden directory; `mv .status.txt .settings/` moves the hidden file inside. `ls -1 .settings` omits hidden children, while `ls -1a .settings` reveals them.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-01/extensions/hidden
$ cd labs/01-linux/day-01/extensions/hidden
$ echo 'local setting' > status.txt
$ mv status.txt .status.txt
$ ls -1
$ ls -1a
$ cat .status.txt
$ mkdir -p .settings
$ mv .status.txt .settings/
$ ls -1 .settings
$ ls -1a .settings
```

### Expected result

The ordinary listings are silent because all the entries being considered are hidden. Extra entries would indicate other work in this workspace; the first-run sample is:

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ mkdir -p labs/01-linux/day-01/extensions/hidden
$ cd labs/01-linux/day-01/extensions/hidden
$ echo 'local setting' > status.txt
$ mv status.txt .status.txt
$ ls -1
$ ls -1a
.
..
.status.txt
$ cat .status.txt
local setting
$ mkdir -p .settings
$ mv .status.txt .settings/
$ ls -1 .settings
$ ls -1a .settings
.
..
.status.txt
```

### Check and answer key

1. **Check:** Does renaming to a dotfile protect its contents? **Answer:** No; permissions and access controls determine access, not the leading dot.
2. **Check:** Why not create both `Report.txt` and `report.txt` on this laptop? **Answer:** A case-insensitive volume may treat them as the same file and overwrite data.
3. **Check:** What do the `.` and `..` entries represent? **Answer:** The current directory and its parent.

## Extension 9 — Inspect and clean up named lab files

Do this after Extensions 4–8 and after saving observations outside the extension workspace. The checked core's `day-01/workspace` stays available. Cleanup targets only files/directories created by the extensions.

Before the commands:

- The absolute `cd` and `pwd` establish the checkout. `find .../extensions -print` lists the workspace tree, including hidden entries. Inspect it before deleting anything. If it contains your own additional files, preserve them and adapt the cleanup; do not broaden the deletion command.
- `rmdir .../moves` initially fails **as expected** because the directory has contents. `rmdir` removes only empty directories.
- `rm` removes each explicitly named extension file; it does not send them to Trash. No wildcard, force flag, or recursive deletion is used here. If a listed file is already absent, a missing-file error means that target needs no further deletion; inspect the remaining tree.
- The subsequent `rmdir` calls remove empty children before parents. Directory names with spaces remain quoted. `rmdir -p` can also try to remove empty ancestor directories; this exercise uses explicit paths to bound the cleanup.
- `rm -r` means recursively remove a directory and contents; `-f` suppresses many prompts/errors. They are conceptual only here because broad recursive removal can destroy unrelated work. `rm -i` offers per-file confirmation, but still deletes the approved file rather than moving it to Trash.
- Finally `ls -1 labs/01-linux/day-01` confirms that this day’s `README.md` and the original `workspace` remain. `exit` or Ctrl-D can close your shell afterward; they are optional keyboard/session actions, not part of this cleanup transcript.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ pwd
$ find labs/01-linux/day-01/extensions -print
$ rmdir labs/01-linux/day-01/extensions/moves
$ rm labs/01-linux/day-01/extensions/text/part-a.txt labs/01-linux/day-01/extensions/text/part-b.txt labs/01-linux/day-01/extensions/text/combined.txt labs/01-linux/day-01/extensions/text/session.log
$ rm labs/01-linux/day-01/extensions/moves/draft.txt labs/01-linux/day-01/extensions/moves/reviewed.txt
$ rm labs/01-linux/day-01/extensions/streams/events.txt labs/01-linux/day-01/extensions/streams/unique.txt
$ rm labs/01-linux/day-01/extensions/hidden/.settings/.status.txt
$ rmdir labs/01-linux/day-01/extensions/moves/destination labs/01-linux/day-01/extensions/hidden/.settings
$ rmdir labs/01-linux/day-01/extensions/paths/branch/leaf
$ rmdir labs/01-linux/day-01/extensions/paths/branch labs/01-linux/day-01/extensions/paths/inbox labs/01-linux/day-01/extensions/paths/archive "labs/01-linux/day-01/extensions/paths/release notes" "labs/01-linux/day-01/extensions/paths/team notes" "labs/01-linux/day-01/extensions/paths/escaped notes"
$ rmdir labs/01-linux/day-01/extensions/paths labs/01-linux/day-01/extensions/text labs/01-linux/day-01/extensions/moves labs/01-linux/day-01/extensions/streams labs/01-linux/day-01/extensions/hidden
$ rmdir labs/01-linux/day-01/extensions
$ ls -1 labs/01-linux/day-01
```

### Expected result

The tree's traversal order and error wording vary. The listing below is abbreviated; inspect the full actual output before removing the named files. The first `rmdir` failure is intentional; later cleanup succeeds once the directories are empty.

```console
$ cd /Users/citradigital/Desktop/learn/platform-engineer
$ pwd
/Users/citradigital/Desktop/learn/platform-engineer
$ find labs/01-linux/day-01/extensions -print
labs/01-linux/day-01/extensions
labs/01-linux/day-01/extensions/paths
labs/01-linux/day-01/extensions/text
labs/01-linux/day-01/extensions/moves
labs/01-linux/day-01/extensions/streams
labs/01-linux/day-01/extensions/hidden
[all nested directories and files also listed; omitted here for space]
$ rmdir labs/01-linux/day-01/extensions/moves
rmdir: labs/01-linux/day-01/extensions/moves: Directory not empty
$ rm labs/01-linux/day-01/extensions/text/part-a.txt labs/01-linux/day-01/extensions/text/part-b.txt labs/01-linux/day-01/extensions/text/combined.txt labs/01-linux/day-01/extensions/text/session.log
$ rm labs/01-linux/day-01/extensions/moves/draft.txt labs/01-linux/day-01/extensions/moves/reviewed.txt
$ rm labs/01-linux/day-01/extensions/streams/events.txt labs/01-linux/day-01/extensions/streams/unique.txt
$ rm labs/01-linux/day-01/extensions/hidden/.settings/.status.txt
$ rmdir labs/01-linux/day-01/extensions/moves/destination labs/01-linux/day-01/extensions/hidden/.settings
$ rmdir labs/01-linux/day-01/extensions/paths/branch/leaf
$ rmdir labs/01-linux/day-01/extensions/paths/branch labs/01-linux/day-01/extensions/paths/inbox labs/01-linux/day-01/extensions/paths/archive "labs/01-linux/day-01/extensions/paths/release notes" "labs/01-linux/day-01/extensions/paths/team notes" "labs/01-linux/day-01/extensions/paths/escaped notes"
$ rmdir labs/01-linux/day-01/extensions/paths labs/01-linux/day-01/extensions/text labs/01-linux/day-01/extensions/moves labs/01-linux/day-01/extensions/streams labs/01-linux/day-01/extensions/hidden
$ rmdir labs/01-linux/day-01/extensions
$ ls -1 labs/01-linux/day-01
README.md
workspace
```

### Check and answer key

1. **Check:** Why did the first `rmdir` fail? **Answer:** Its directory contained files and a subdirectory; `rmdir` refuses nonempty directories.
2. **Check:** Why inspect with `find` first? **Answer:** Confirm the exact scope and preserve anything beyond the files created by this lab.
3. **Check:** Can `rm` be undone by looking in Trash? **Answer:** Normally no; recovery would require a separate backup or recovery mechanism.

## Extension 10 — Administrator access and Ubuntu software (conceptual only)

This is a written task; **do not execute administrator, package-installation, or sensitive-file commands on this Mac**. Ordinary lab files belong to you and require no elevation.

The root account usually has UID 0 and can change system-wide resources. Older workflows use `su` to switch identity and open a privileged shell, or log in directly as root. Every subsequent command in that shell can then affect the system. Ubuntu commonly disables direct root-password login and grants selected users controlled elevation through `sudo`; policy, rather than the command name alone, decides what is allowed.

`sudo` executes a specified command under an authorized identity, normally root. It commonly authenticates using your own password, may remember that authentication briefly, and records activity according to system policy. Password entry may show no characters. A shell obtained via `sudo su` or `sudo -i` keeps later commands privileged. A `#` prompt often indicates privilege but is configurable; it is not reliable proof of identity. Elevation increases authority, not the correctness of a command or trustworthiness of downloaded code.

On Linux, `/etc/shadow` commonly stores password hashes and account-aging data, with access restricted to privileged users/services. A password hash is not plaintext, but it is still sensitive. A normal-user read may be denied; that restriction is expected and is not something to bypass for practice. macOS manages accounts differently, so a Linux path/example is not a macOS diagnostic procedure. Avoid collecting or sharing protected account data.

Ubuntu's `apt` manages packages from configured repositories. In the conceptual expression `sudo apt install tree`, `sudo` requests elevation, `apt` is the package tool, `install` is its action, and `tree` is a package name. `apt-get` is a related package interface; `apt` does not ship as macOS's system package manager. Installing packages changes the machine and may require downloads. Repository additions, PPAs, and downloaded installation scripts introduce trust decisions; confirm the publisher/source before granting permissions. Ubuntu also supports Snap packages, whose confinement depends on packaging and options. None needs installation for this lab.

`tree` displays nested paths visually; its `-a` option commonly includes hidden entries. It may not be installed on this laptop. Our existing `find ... -print` practice covers tree inspection without adding software. System settings often live under `/etc`, while individual dotfiles can hold user preferences; readable configuration does not automatically require privileged access. Editing system settings may.

**Written task:** Explain to a teammate why none of the extension exercises requires `sudo`, how package installation differs from making a local directory, and why you would not copy an unfamiliar privileged command into your terminal.

### Expected result — Sample written response

“All practice uses my own isolated workspace. Creating its directories changes local files, while Ubuntu package installation changes system-managed software using repositories and authorized privileges. `sudo` can give a command root authority; it does not validate what that command does. I would first identify the action, source, intended changes, and target environment. I would not inspect or share `/etc/shadow` for this exercise. `/` is a directory and root is an account.”

### Check and answer key

1. **Check:** Does every permission error need `sudo`? **Answer:** No. Confirm ownership, intended access, and the target path first; protected system files may deliberately deny access.
2. **Check:** Does Ubuntu's root-login policy eliminate privileged execution? **Answer:** No. Authorized `sudo` execution can still obtain root powers, including a lasting privileged shell.
3. **Check:** Should `apt` examples be run on macOS? **Answer:** No. They illustrate Ubuntu package management; no software installation is required here.

## Expanded Day 1 completion criteria

- [x] Core Exercises 1–3 and their answers remain reviewed.
- [ ] Extensions 4–8 practiced, including pager/terminal controls and new checks answered.
- [ ] Extension 9 cleanup completed or documented if additional files need preservation.
- [ ] Extension 10's conceptual task and answers reviewed without privileged execution.
- [ ] You can explain options, quoting, globbing, streams, copying/moving, hidden names, case caveats, and least-privilege use.
- [ ] Expanded Day 1 complete — Update the day checkbox only after all new work is reviewed.

**Share back:** Send extension observations and answers, noting any commands you need to repeat. The completed core remains complete while these extensions are pending.

## Further written reading

Return to the complete [Ubuntu beginner tutorial](https://ubuntu.com/desktop/docs/en/latest/tutorial/the-linux-command-line-for-beginners/) to compare Ubuntu behavior with these macOS adaptations. Its introductory terminal history and linked resources are optional context. For deeper study, [The Linux Command Line by William Shotts](https://linuxcommand.org/tlcl.php) provides a longer written course. For precise behavior on your machine, use the installed manuals for `ls`, `mkdir`, `cp`, `mv`, `rm`, `rmdir`, `less`, `wc`, `sort`, and `uniq`; GNU and BSD option sets can differ.
