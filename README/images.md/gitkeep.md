# SadServers: "Saint John"

---

## 1. The Problem

A developer made a test program. The program keeps writing to this file:

```
/var/log/bad.log
``` 

The log file keeps growing. It is filling up the disk. Nobody needs this program anymore.

**My task:** Find the process that writes to the log file, and stop it.

---

## 2. Steps I Took

### Step 1: Confirm the problem

I watched the log file to see the new lines.

```bash
tail -f /var/log/bad.log
```

**What I saw:**


*![Problem Screenshot](README/images.md/1st.png).*
```

New lines kept coming. This confirmed the problem.

---

### Step 2: Look around

I checked the files in my home folder.

```bash
ls -l
```

**What I saw:**


*![Diagnosis Screenshot](README/images.md/2nd.png).*
```

I found a Python file called `badlog.py`. It looked like the program that makes the logs.

---

### Step 3: Find the process that writes to the file

I did **not** just delete `badlog.py`. Deleting a file does not stop a program that is already running. So I used `lsof` ("list open files") to see who has the log file open.

```bash
lsof /var/log/bad.log
```

**What I saw:**


*![Root cause Screenshot](README/images.md/3rd.png).*
```

From this output, I got the **PID** (process ID) of the program.

---

### Step 4: Check the process

I checked the process before I killed it. I wanted to be sure it was the right one.

```bash
ps -p <PID> -o pid,ppid,user,cmd
```

**What I saw:**


*![Right problem Screenshot](README/images.md/4th.png).*
```

It was the `badlog.py` script. This was the right process.

---

### Step 5: Stop the process

```bash
kill <PID>

**What I saw:**

*![Right problem Screenshot](README/images.md/5th.png).*


 ### Step 6: Check that it worked

I checked three things:

```bash
# 1. Is the process gone?
ps -p <PID>

# 2. Does anyone still have the file open?
lsof /var/log/bad.log

# 3. Run the scenario check
./check.sh
```

**What I saw:**


*![ensuring troubleshooting Screenshot](README/images.md/6th.png).*

```

The file stopped growing. The check passed.

---

## 3. Root Cause

A test script (`badlog.py`) was still running in the background. It wrote to `/var/log/bad.log` all the time and never stopped. Nobody rotated or limited the log, so it filled the disk.

---

## 4. Key Lessons

- **Deleting a file does not stop a running program.** The program lives in memory. It keeps running after you delete the script.
- **Deleting the log file does not help.** The program can create it again and keep writing.
- **`lsof` is the fastest way to find who uses a file.**
- **Always check a process before you kill it.** Use `ps` first.

---

## 6. Commands Used

| Command   | What it does |
|---------  |--------------|
| `tail -f` | Shows new lines of a file as they are added |
| `ls -l`   | Lists files with details |
| `lsof <file>` | Shows which processes have a file open |
| `ps -p <PID>` | Shows details about one process |
| `kill <PID>` | Asks a process to stop |
| `kill -9 <PID>` | Forces a process to stop |

---

## 7. How I Would Prevent This in Real Life

- Use **logrotate** to limit the size of log files.
- Set **disk usage alerts** (for example at 80%).
- Do not leave test programs running on a server.
- Run programs as **systemd services** so they are easy to find, stop, and limit.

---

## 8. Time Taken

*![ensuring troubleshooting Screenshot](README/images.md/7th.png).*
