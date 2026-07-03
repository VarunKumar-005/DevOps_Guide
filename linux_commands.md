# 🐧 The Complete Linux Command Arsenal — Interview & Core Guide
> Every command you need, basic to advanced, with real-world use-cases and sample output.

---

## 1️⃣ File & Directory Operations

| Command | Use Case |
|---|---|
| `ls` | List directory contents |
| `ls -l` | Long listing (permissions, owner, size, date) |
| `ls -la` | Include hidden files (dotfiles) |
| `ls -lh` | Human-readable sizes (K, M, G) |
| `ls -lt` | Sort by modification time (newest first) |
| `ls -lS` | Sort by file size |
| `ls -R` | List recursively |
| `pwd` | Print current working directory |
| `cd path` | Change directory |
| `cd ~` / `cd` | Go to home directory |
| `cd -` | Go to previous directory |
| `cd ..` | Go up one level |
| `mkdir dir` | Create a directory |
| `mkdir -p a/b/c` | Create nested directories |
| `rmdir dir` | Remove empty directory |
| `touch file` | Create empty file / update timestamp |
| `cp src dest` | Copy file |
| `cp -r src dest` | Copy directory recursively |
| `cp -v src dest` | Copy with verbose output |
| `cp -u src dest` | Copy only if source is newer |
| `mv src dest` | Move or rename file/directory |
| `rm file` | Delete file |
| `rm -r dir` | Delete directory recursively |
| `rm -rf dir` | Force delete recursively (no prompts) |
| `rm -i file` | Prompt before deleting |
| `ln src link` | Create hard link |
| `ln -s src link` | Create symbolic (soft) link |
| `readlink -f file` | Resolve absolute path of symlink |
| `basename path` | Extract filename from path |
| `dirname path` | Extract directory from path |
| `realpath path` | Show absolute canonical path |
| `stat file` | Detailed file metadata (inode, size, timestamps) |
| `file filename` | Determine file type |
| `tree` | Display directory structure as a tree |
| `du -sh dir` | Show total size of a directory |
| `du -h --max-depth=1` | Size of each subdirectory (1 level) |
| `shred file` | Securely overwrite & delete a file |
| `mktemp` | Create a temporary file safely |

```bash
$ ls -lh
-rw-r--r-- 1 user user 2.1K Jul  3 10:00 report.txt

$ stat report.txt
  File: report.txt
  Size: 2148       Blocks: 8  IO Block: 4096   regular file
Modify: 2026-07-03 10:00:12.000000000 +0000

$ du -sh /var/log
128M  /var/log
```

---

## 2️⃣ File Viewing & Text Processing

| Command | Use Case |
|---|---|
| `cat file` | Print full file content |
| `cat -n file` | Print with line numbers |
| `tac file` | Print file in reverse (bottom to top) |
| `less file` | Paginated view (search with `/`, quit `q`) |
| `more file` | Basic pager (older, simpler than less) |
| `head file` | First 10 lines |
| `head -n 20 file` | First 20 lines |
| `tail file` | Last 10 lines |
| `tail -n 50 file` | Last 50 lines |
| `tail -f file` | Follow file live (logs) |
| `tail -F file` | Follow + handle log rotation |
| `wc file` | Lines, words, bytes count |
| `wc -l file` | Line count only |
| `sort file` | Sort lines alphabetically |
| `sort -n file` | Sort numerically |
| `sort -r file` | Sort in reverse |
| `sort -u file` | Sort and remove duplicates |
| `uniq file` | Remove adjacent duplicate lines |
| `uniq -c file` | Count occurrences of each line |
| `cut -d, -f1 file` | Extract column 1 (comma delimited) |
| `paste file1 file2` | Merge lines of files side-by-side |
| `tr 'a-z' 'A-Z'` | Translate/transform characters |
| `diff file1 file2` | Show differences between files |
| `diff -u file1 file2` | Unified diff format (used in patches) |
| `cmp file1 file2` | Byte-by-byte file comparison |
| `comm file1 file2` | Compare sorted files line by line |
| `grep pattern file` | Search text pattern |
| `grep -i pattern file` | Case-insensitive search |
| `grep -r pattern dir/` | Recursive search in directory |
| `grep -v pattern file` | Invert match (exclude lines) |
| `grep -c pattern file` | Count matching lines |
| `grep -n pattern file` | Show line numbers of matches |
| `grep -E 'regex' file` | Extended regex search (egrep) |
| `egrep`, `fgrep` | Extended / fixed-string grep variants |
| `awk '{print $1}' file` | Print first column |
| `awk -F: '{print $1}' /etc/passwd` | Custom delimiter field extraction |
| `sed 's/old/new/' file` | Replace first occurrence per line |
| `sed 's/old/new/g' file` | Replace all occurrences |
| `sed -i 's/old/new/g' file` | Edit file in-place |
| `sed -n '5,10p' file` | Print lines 5-10 |
| `xargs` | Build & execute commands from input |
| `column -t file` | Format text into aligned columns |
| `fold -w 80 file` | Wrap lines at width 80 |
| `nl file` | Number lines of a file |
| `rev file` | Reverse each line's characters |
| `strings binaryfile` | Extract printable strings from a binary |
| `hexdump -C file` | View file in hex/ASCII |
| `xxd file` | Hex dump utility |
| `iconv -f UTF-8 -t ASCII file` | Convert text encoding |
| `base64 file` | Encode/decode base64 |
| `md5sum file` | Generate MD5 checksum |
| `sha256sum file` | Generate SHA-256 checksum |

```bash
$ grep -rn "TODO" ./src
src/app.py:42:# TODO: refactor this function

$ awk -F: '{print $1}' /etc/passwd | head -3
root
daemon
bin

$ sed -i 's/localhost/127.0.0.1/g' config.yml

$ sha256sum backup.tar.gz
a1b2c3d4e5f6...  backup.tar.gz
```

---

## 3️⃣ Permissions & Ownership

| Command | Use Case |
|---|---|
| `chmod 755 file` | Set exact permission bits |
| `chmod +x file` | Add execute permission |
| `chmod -w file` | Remove write permission |
| `chmod -R 755 dir/` | Apply recursively |
| `chmod u+x,g-w,o=r file` | Symbolic permission changes |
| `chown user file` | Change owner |
| `chown user:group file` | Change owner and group |
| `chown -R user:group dir/` | Recursive ownership change |
| `chgrp group file` | Change group only |
| `umask` | View/set default permission mask |
| `getfacl file` | View Access Control List |
| `setfacl -m u:user:rwx file` | Set fine-grained ACL permission |
| `chattr +i file` | Make file immutable (even root can't delete) |
| `lsattr file` | List special file attributes |
| `sudo command` | Run command as superuser |
| `sudo -i` | Start root interactive shell |
| `su username` | Switch user |
| `su -` | Switch to root with full environment |
| `visudo` | Safely edit `/etc/sudoers` |

```bash
$ chmod 750 deploy.sh
$ ls -l deploy.sh
-rwxr-x--- 1 user user 512 Jul  3 deploy.sh

$ sudo chown -R www-data:www-data /var/www/html
```

> 💡 **Permission math**: `r=4, w=2, x=1`. `750` → owner: rwx(7), group: r-x(5), others: none(0).

---

## 4️⃣ Process & Job Management

| Command | Use Case |
|---|---|
| `ps` | Snapshot of current shell's processes |
| `ps aux` | All processes, all users, detailed |
| `ps -ef` | Full-format process listing |
| `ps -ef --forest` | Show process tree |
| `top` | Live resource usage monitor |
| `htop` | Interactive, colorful process viewer |
| `pstree` | Visual tree of process hierarchy |
| `kill PID` | Send SIGTERM (graceful stop) |
| `kill -9 PID` | Send SIGKILL (force kill) |
| `kill -l` | List all signal names |
| `killall processname` | Kill all processes by name |
| `pkill pattern` | Kill processes matching pattern |
| `pgrep pattern` | Get PID(s) of matching processes |
| `nice -n 10 command` | Start process with lower priority |
| `renice 5 -p PID` | Change priority of running process |
| `bg` | Resume job in background |
| `fg` | Bring job to foreground |
| `jobs` | List background/stopped jobs |
| `nohup command &` | Run command immune to hangups |
| `disown` | Detach job from shell |
| `command &` | Run command in background |
| `Ctrl+Z` | Suspend current foreground process |
| `wait PID` | Wait for background process to finish |
| `lsof` | List open files by processes |
| `lsof -i :8080` | Find process using a specific port |
| `strace command` | Trace system calls of a process |
| `time command` | Measure execution time |

```bash
$ ps aux | grep python
user  4521  2.3  1.2  55432 12300 pts/0  Sl  10:00  0:05 python app.py

$ kill -9 4521

$ lsof -i :3000
COMMAND  PID USER   FD   TYPE DEVICE NODE NAME
node    1234 user   20u  IPv4  ...   TCP *:3000 (LISTEN)
```

---

## 5️⃣ Networking

| Command | Use Case |
|---|---|
| `ping host` | Test connectivity to host |
| `ping -c 4 host` | Send fixed number of pings |
| `traceroute host` | Trace route packets take to host |
| `mtr host` | Combines ping + traceroute live |
| `curl url` | Transfer data / test API endpoints |
| `curl -I url` | Fetch only HTTP headers |
| `curl -X POST -d "data" url` | Send POST request |
| `wget url` | Download files from web |
| `wget -c url` | Resume interrupted download |
| `ssh user@host` | Secure remote login |
| `ssh -p 2222 user@host` | SSH on custom port |
| `ssh-keygen` | Generate SSH key pair |
| `ssh-copy-id user@host` | Copy public key to remote server |
| `scp file user@host:/path` | Secure copy over SSH |
| `rsync -avz src dest` | Efficient file sync (delta transfer) |
| `netstat -tulnp` | List listening ports & processes (legacy) |
| `ss -tulnp` | Modern replacement for netstat |
| `ip a` | Show IP addresses (modern) |
| `ip route` | Show routing table |
| `ifconfig` | Show/configure interfaces (legacy) |
| `hostname` | Show system hostname |
| `hostname -I` | Show system IP address |
| `nslookup domain` | DNS lookup |
| `dig domain` | Detailed DNS query tool |
| `host domain` | Simple DNS lookup |
| `whois domain` | Domain registration info |
| `telnet host port` | Test raw TCP connection to port |
| `nc -zv host port` | Netcat port-scan/test connectivity |
| `nmap host` | Network/port scanning |
| `iptables -L` | List firewall rules |
| `ufw status` | Uncomplicated Firewall status (Ubuntu) |
| `firewall-cmd --list-all` | Firewall status (RHEL/CentOS) |
| `arp -a` | Show ARP table (MAC-IP mapping) |
| `route -n` | Show kernel routing table (legacy) |

```bash
$ ss -tulnp
Netid State  Local Address:Port   Process
tcp   LISTEN 0.0.0.0:22           sshd
tcp   LISTEN 0.0.0.0:80           nginx

$ curl -I https://example.com
HTTP/2 200
content-type: text/html

$ dig google.com +short
142.250.183.14
```

---

## 6️⃣ Archiving, Compression & Backup

| Command | Use Case |
|---|---|
| `tar -cvf archive.tar dir/` | Create archive |
| `tar -xvf archive.tar` | Extract archive |
| `tar -tvf archive.tar` | List archive contents without extracting |
| `tar -czvf archive.tar.gz dir/` | Create gzip-compressed archive |
| `tar -xzvf archive.tar.gz` | Extract gzip archive |
| `tar -cjvf archive.tar.bz2 dir/` | Create bzip2 archive (better ratio) |
| `gzip file` | Compress a file (.gz) |
| `gunzip file.gz` | Decompress a .gz file |
| `bzip2 file` | Compress with bzip2 |
| `xz file` | Compress with xz (best ratio, slower) |
| `zip -r archive.zip dir/` | Create zip archive |
| `unzip archive.zip` | Extract zip archive |
| `unzip -l archive.zip` | List zip contents |
| `rsync -avz --delete src/ dest/` | Mirror sync (deletes extras in dest) |
| `dd if=input of=output` | Low-level copy/clone (disks, ISOs) |
| `cpio` | Archive utility (legacy, used with find) |

```bash
$ tar -czvf project_backup.tar.gz ./project
$ tar -tzvf project_backup.tar.gz | head -5

$ dd if=/dev/sda of=disk_backup.img bs=4M status=progress
```

> ⚠️ `dd` is powerful and dangerous — wrong `of=` target can wipe a disk!

---

## 7️⃣ Package Management

### Debian/Ubuntu (APT)
| Command | Use Case |
|---|---|
| `apt-get update` | Refresh package index |
| `apt-get upgrade` | Upgrade all installed packages |
| `apt-get install pkg` | Install a package |
| `apt-get remove pkg` | Remove package (keep config) |
| `apt-get purge pkg` | Remove package + config files |
| `apt-get autoremove` | Remove unused dependencies |
| `apt list --installed` | List installed packages |
| `apt-cache search pkg` | Search for available packages |
| `dpkg -i package.deb` | Install local .deb package |
| `dpkg -l` | List all installed packages |

### RHEL/CentOS/Fedora
| Command | Use Case |
|---|---|
| `yum install pkg` | Install a package |
| `yum update` | Update all packages |
| `yum remove pkg` | Remove a package |
| `yum search pkg` | Search for a package |
| `dnf install pkg` | Modern replacement for yum |
| `rpm -ivh package.rpm` | Install local .rpm package |
| `rpm -qa` | List all installed packages |

```bash
$ sudo apt-get update && sudo apt-get install -y docker.io
$ sudo dnf install httpd -y
```

---

## 8️⃣ System Information & Monitoring

| Command | Use Case |
|---|---|
| `uname -a` | Full kernel/system info |
| `uname -r` | Kernel version only |
| `hostnamectl` | System hostname & OS details |
| `lsb_release -a` | Distro version info |
| `cat /etc/os-release` | OS identification |
| `uptime` | System uptime & load average |
| `top` | Live CPU/memory/process monitor |
| `htop` | Enhanced interactive monitor |
| `vmstat` | Virtual memory statistics |
| `iostat` | CPU & disk I/O statistics |
| `df -h` | Disk space usage (human-readable) |
| `du -sh *` | Size of files/folders in current dir |
| `free -h` | RAM & swap usage |
| `lscpu` | Detailed CPU architecture info |
| `nproc` | Number of processing units available |
| `lsblk` | List block devices (disks/partitions) |
| `fdisk -l` | List disk partitions (detailed) |
| `mount` | Show mounted filesystems |
| `umount /mnt/point` | Unmount a filesystem |
| `lsusb` | List USB devices |
| `lspci` | List PCI devices |
| `dmesg` | Kernel ring buffer / boot messages |
| `journalctl` | Query systemd logs |
| `journalctl -u service` | Logs for a specific service |
| `journalctl -f` | Follow logs live |
| `who` | Show logged-in users |
| `w` | Logged-in users + their activity |
| `last` | Login history |
| `id` | Show current user's UID/GID/groups |
| `whoami` | Print current username |

```bash
$ free -h
              total   used   free
Mem:           16Gi   4.2Gi   9Gi

$ lscpu | grep "Model name"
Model name: Intel(R) Core(TM) i7-9700K

$ journalctl -u nginx -f
Jul 03 10:20:11 server nginx[1200]: worker process started
```

---

## 9️⃣ Scripting & Shell

| Command / Concept | Use Case |
|---|---|
| `#!/bin/bash` | Shebang — defines script interpreter |
| `chmod +x script.sh` | Make script executable |
| `./script.sh` | Run a script |
| `bash script.sh` | Run script explicitly with bash |
| `var="value"` | Define a variable |
| `$var` / `${var}` | Access variable value |
| `read var` | Take user input |
| `if [ cond ]; then ... fi` | Conditional statement |
| `if [[ cond ]]; then ... fi` | Extended test (bash-specific) |
| `for i in list; do ... done` | For loop |
| `while [ cond ]; do ... done` | While loop |
| `case $var in ... esac` | Multi-branch conditional |
| `function name() { ... }` | Define a function |
| `$1, $2, $@, $#` | Positional args, all args, arg count |
| `$?` | Exit status of last command |
| `$$` | PID of current shell |
| `exit 0` | Exit script with status code |
| `trap 'cmd' SIGNAL` | Run command on receiving a signal |
| `source script.sh` / `. script.sh` | Execute script in current shell |
| `set -e` | Exit script on any error |
| `set -x` | Debug mode — print each command |
| `export VAR=value` | Set environment variable |
| `alias ll='ls -la'` | Create command shortcut |
| `history` | Show command history |
| `!!` | Repeat last command |
| `!n` | Run command number n from history |

```bash
#!/bin/bash
set -e
for file in *.log; do
  if [[ -f "$file" ]]; then
    echo "Processing $file"
    gzip "$file"
  fi
done
echo "Done! Exit status: $?"
```

---

## 🔟 Version Control (Git)

| Command | Use Case |
|---|---|
| `git init` | Initialize new repo |
| `git clone url` | Clone remote repo |
| `git status` | Show working tree status |
| `git add file` | Stage a file |
| `git add .` | Stage all changes |
| `git commit -m "msg"` | Commit staged changes |
| `git commit -am "msg"` | Stage tracked files + commit |
| `git push` | Push commits to remote |
| `git pull` | Fetch + merge from remote |
| `git fetch` | Download remote changes (no merge) |
| `git branch` | List branches |
| `git branch name` | Create new branch |
| `git checkout branch` | Switch branch |
| `git checkout -b branch` | Create + switch branch |
| `git switch branch` | Modern branch switch |
| `git merge branch` | Merge branch into current |
| `git rebase branch` | Reapply commits on top of another base |
| `git log` | Commit history |
| `git log --oneline --graph` | Compact visual history |
| `git diff` | Show unstaged changes |
| `git diff --staged` | Show staged changes |
| `git stash` | Temporarily save changes |
| `git stash pop` | Reapply stashed changes |
| `git reset --hard HEAD` | Discard all local changes |
| `git revert commit` | Undo a commit safely (new commit) |
| `git cherry-pick commit` | Apply a specific commit elsewhere |
| `git remote -v` | Show remote repo URLs |
| `git tag v1.0` | Create a version tag |
| `git blame file` | See who changed each line |

```bash
$ git checkout -b feature/login
$ git add .
$ git commit -m "Add login flow"
$ git push origin feature/login
```

---

## 1️⃣1️⃣ Users, Groups & Authentication

| Command | Use Case |
|---|---|
| `useradd username` | Create new user |
| `useradd -m -s /bin/bash username` | Create user with home dir & shell |
| `userdel username` | Delete a user |
| `userdel -r username` | Delete user + home directory |
| `usermod -aG group user` | Add user to group (append) |
| `usermod -s /bin/zsh user` | Change user's default shell |
| `passwd username` | Set/change password |
| `passwd -l username` | Lock a user account |
| `chage -l username` | View password expiry info |
| `groupadd groupname` | Create new group |
| `groupdel groupname` | Delete a group |
| `groupmod -n newname oldname` | Rename a group |
| `groups username` | Show groups a user belongs to |
| `id username` | Show UID/GID/group info |
| `cat /etc/passwd` | List all system users |
| `cat /etc/group` | List all system groups |
| `cat /etc/shadow` | View encrypted passwords (root only) |
| `su - username` | Switch to another user (full env) |
| `sudo -l` | List sudo privileges for current user |

```bash
$ sudo useradd -m -s /bin/bash devops
$ sudo passwd devops
$ sudo usermod -aG sudo devops
```

---

## 1️⃣2️⃣ Environment & Shell Configuration

| Command | Use Case |
|---|---|
| `echo $VAR` | Print variable value |
| `export VAR=value` | Set env variable for child processes |
| `unset VAR` | Remove a variable |
| `env` | List all environment variables |
| `printenv` | Print environment variables |
| `printenv VAR` | Print a specific variable |
| `set` | Show all shell variables + functions |
| `echo $PATH` | Show executable search path |
| `export PATH=$PATH:/new/dir` | Add directory to PATH |
| `which command` | Show path of an executable |
| `whereis command` | Locate binary, source, man page |
| `type command` | Show how shell interprets command |
| `.bashrc` / `.bash_profile` | Shell startup config files |
| `/etc/environment` | System-wide environment variables |
| `source ~/.bashrc` | Reload shell config |

```bash
$ export JAVA_HOME=/usr/lib/jvm/java-17
$ echo $JAVA_HOME
/usr/lib/jvm/java-17

$ which python3
/usr/bin/python3
```

---

## 1️⃣3️⃣ Advanced Search & Text Manipulation

| Command | Use Case |
|---|---|
| `find /path -name "*.log"` | Search files by name |
| `find /path -type f` | Find only regular files |
| `find /path -type d` | Find only directories |
| `find /path -mtime -7` | Files modified in last 7 days |
| `find /path -size +100M` | Files larger than 100MB |
| `find /path -name "*.tmp" -delete` | Find and delete matching files |
| `find /path -exec chmod 644 {} \;` | Run command on each result |
| `locate filename` | Fast file search (uses prebuilt DB) |
| `updatedb` | Update the `locate` database |
| `which command` | Locate an executable |
| `awk '{print $2}'` | Column extraction / text processing |
| `awk -F',' '{sum+=$3} END{print sum}'` | Sum a column in CSV |
| `sed -n '/pattern/p' file` | Print only matching lines |
| `sed '/pattern/d' file` | Delete matching lines |
| `grep -A 3 pattern file` | Show 3 lines after match |
| `grep -B 3 pattern file` | Show 3 lines before match |
| `grep -C 3 pattern file` | Show 3 lines context both sides |
| `xargs -I{} cmd {}` | Pass each input line as an argument |
| `parallel` | Run commands in parallel (GNU parallel) |
| `watch command` | Repeat a command every N seconds |
| `tee file` | Write output to file AND stdout |

```bash
$ find /var/log -type f -mtime +30 -name "*.log" -delete

$ awk -F',' '{sum+=$3} END{print "Total:", sum}' sales.csv
Total: 45820

$ ps aux | grep nginx | tee nginx_procs.txt
```

---

## 1️⃣4️⃣ System Administration & Automation

| Command | Use Case |
|---|---|
| `crontab -e` | Edit current user's cron jobs |
| `crontab -l` | List current user's cron jobs |
| `crontab -r` | Remove all cron jobs |
| `systemctl start service` | Start a service |
| `systemctl stop service` | Stop a service |
| `systemctl restart service` | Restart a service |
| `systemctl status service` | Check service status |
| `systemctl enable service` | Enable service on boot |
| `systemctl disable service` | Disable service on boot |
| `systemctl daemon-reload` | Reload systemd unit files |
| `service name status` | Legacy service status check |
| `init 0` / `shutdown now` | Shutdown system |
| `reboot` | Restart system |
| `systemctl list-units` | List active systemd units |
| `at time` | Schedule a one-time task |
| `logrotate` | Manage log file rotation |
| `swapon -s` | Show active swap usage |
| `mount -a` | Mount all filesystems in fstab |
| `blkid` | Show block device UUIDs |
| `parted` | Partition management tool |
| `mkfs.ext4 /dev/sdX` | Format a partition |

```bash
$ crontab -e
0 2 * * * /home/user/backup.sh >> /var/log/backup.log 2>&1

$ sudo systemctl restart nginx
$ sudo systemctl enable nginx
```

---

## 🎯 Rapid Recall Table (Say These Out Loud)

| Need to... | Reach for |
|---|---|
| See file content | `cat`, `less`, `head`, `tail` |
| Search text/files | `grep`, `find`, `locate` |
| Edit text in bulk | `sed`, `awk`, `tr` |
| Fix permissions | `chmod`, `chown`, `setfacl` |
| Manage processes | `ps`, `top`, `kill`, `lsof` |
| Move data around | `cp`, `mv`, `scp`, `rsync` |
| Compress/backup | `tar`, `gzip`, `rsync`, `dd` |
| Check the network | `ping`, `ss`, `curl`, `dig` |
| Automate tasks | `cron`, shell scripts, `systemctl` |
| Diagnose the system | `top`, `df -h`, `free -h`, `journalctl` |
| Version control | `git add/commit/push/pull` |

**You now hold the full arsenal. Go get that offer! 🚀**
