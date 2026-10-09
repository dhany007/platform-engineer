# Day 1 — Shell, Paths, and Files

**Time:** About 60–90 minutes.

## Progress

- [x] Exercise 1 — Locate the project.
- [x] Exercise 2 — Create and inspect files.
- [x] Exercise 3 — Pipe and save command output.
- [x] Check your understanding — Answer the three questions below.
- [x] Day 1 complete — All exercises and answers reviewed.

## Why this matters

When a service fails, you often need to locate its configuration, inspect a log, and save evidence before changing anything. The shell gives you a repeatable way to do that. Today's commands work in a macOS terminal and in common Linux shells; later days will introduce Linux-specific behavior.

## Learning goals

By the end of this session, you should be able to:

1. Tell the difference between the current directory, an absolute path, and a relative path.
2. Navigate and inspect files with `pwd`, `cd`, `ls`, `find`, and `cat`.
3. Explain what a pipe (`|`), output redirection (`>`), and input redirection (`<`) do.
4. Save command output in a file and inspect it afterward.

## Reading

Read the sections on navigation, paths, files, and pipes in [Ubuntu's Linux command-line tutorial](https://documentation.ubuntu.com/desktop/en/latest/tutorial/the-linux-command-line-for-beginners/). You can read the remaining sections later.

## Exercise 1 — Locate the project

Open a terminal and enter the project directory. The path below matches this laptop; adjust it if you use a different checkout.

Before running the commands, know what each one does:

- `cd /Users/citradigital/Desktop/learn/platform-engineer` changes to the project directory using an **absolute path**. A path beginning with `/` starts at the filesystem root.
- `pwd` prints the current working directory, so you can confirm where you are.
- `ls -la` lists entries with details (`-l`) and includes hidden entries (`-a`), such as `.git`.
- `cd labs/01-linux` changes directory using a **relative path**. It is resolved from your current directory.
- `cd ../..` moves up two parent directories; each `..` means one parent.

```sh
cd /Users/citradigital/Desktop/learn/platform-engineer
pwd
ls -la
cd labs/01-linux
pwd
cd ../..
pwd
```

Check that the three `pwd` results show the project root, then `labs/01-linux`, then the project root again.

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

```sh
mkdir -p labs/01-linux/day-01/workspace
printf 'service=go-api\nport=8080\n' > labs/01-linux/day-01/workspace/sample.conf
cat labs/01-linux/day-01/workspace/sample.conf
ls -la labs/01-linux/day-01/workspace
```

Expected file contents: `service=go-api` on the first line and `port=8080` on the second.

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

```sh
find roadmap -type f | sort > labs/01-linux/day-01/workspace/roadmap-files.txt
cat labs/01-linux/day-01/workspace/roadmap-files.txt
wc -l < labs/01-linux/day-01/workspace/roadmap-files.txt
```

This repository currently has one file under `roadmap`, so the expected count is `1`.

### Expected result

The first command is silent because output is saved to a file. The saved list contains filenames, and its line count is one in the current repository; adding roadmap files changes both.

```console
$ find roadmap -type f | sort > labs/01-linux/day-01/workspace/roadmap-files.txt
$ cat labs/01-linux/day-01/workspace/roadmap-files.txt
roadmap/12-week.md
$ wc -l < labs/01-linux/day-01/workspace/roadmap-files.txt
       1
```

## Check your understanding

Answer in your own words:

1. Why can `cd labs/01-linux` fail when run from a different directory?
2. What is the difference between `|` and `>` in Exercise 3?
3. What happens to `sample.conf` if you repeat the `printf ... > sample.conf` command?

### Answer key

1. `labs/01-linux` is relative to the current directory. From another directory, that relative path may name a different location or not exist.
2. `|` passes one command’s output to another command as input. `>` writes output to a file and overwrites its previous contents.
3. The file is overwritten with the same two lines; they are not appended.

**Review status:** The command outputs and answers for Exercises 1–3 have been reviewed. Day 1 is complete.

## Completion criteria

- [x] You can navigate from the project root into `labs/01-linux` and back.
- [x] You created and inspected `sample.conf`.
- [x] You saved a sorted file list and counted its lines.
- [x] You can explain relative paths, pipes, and redirection without looking at the commands.
