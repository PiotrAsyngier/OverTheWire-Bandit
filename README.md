# 🐧 OverTheWire: Bandit — Complete Walkthrough & Linux CLI Methodology (Levels 0–33)

> ⚠️ **Disclaimer (OverTheWire Rules & OPSEC):** In accordance with OverTheWire policy, plain-text passwords in the text have been omitted or redacted. This document focuses on the **methodology, Linux command-line utilities, privilege escalation vectors, and bash scripting techniques** used to solve each challenge.

---

### Bandit Level 0
The main goal of this level is to establish an SSH connection to the lab environment on port `2220`. Once connected, we use the `ls` and `cat` commands to inspect the home directory and read the `readme` file, which stores the password for the next level:

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
ls
cat readme
```

<img width="1858" height="1007" alt="obraz" src="https://github.com/user-attachments/assets/e9e4f1f7-d4f3-46ca-a465-11adb0f7b7ab" />

<img width="1858" height="282" alt="obraz" src="https://github.com/user-attachments/assets/75deb29e-fa54-4fb3-8aee-91c3300f07d1" />


---

### Bandit Level 1
To solve this level, we need to retrieve the password stored in a file named `-` located in the home directory. In Linux CLI, a single hyphen (`-`) is typically interpreted as `stdin`/`stdout` or a command flag argument. To read the file without triggering standard input behavior, we explicitly specify its path using `./-` or `~/-`:

```bash
ls
cat ~/-
```

<img width="349" height="131" alt="Pasted image 20261103221326" src="https://github.com/user-attachments/assets/8e0d5e44-35a8-40de-ba7f-81b3a2a6f73e" />


---

### Bandit Level 2
This level requires reading a file named `--spaces in this filename--`. Because spaces act as argument separators in Bash and the leading `--` resembles a flag, we enclose the filename in single quotes (or escape the spaces with backslashes) while referencing the home directory path:

```bash
ls
cat ~/'--spaces in this filename--'
```

<img width="570" height="134" alt="obraz" src="https://github.com/user-attachments/assets/40954cd0-85bd-4987-b2c9-a980848f5ee2" />


---

### Bandit Level 3
Here, the password is stored inside a hidden file within the `inhere` directory. After navigating into `inhere`, we run `ls -la`:
* `-a` displays all files, including hidden dotfiles (starting with `.`).
* `-l` outputs the long listing format (permissions, owner, group, file size, and modification timestamp).

```bash
cd inhere
ls -la
cat ~/inhere/...Hiding-From-You
```

<img width="666" height="254" alt="obraz" src="https://github.com/user-attachments/assets/d7ad5286-1f4d-41b1-ad8b-4d541b4deece" />


---

### Bandit Level 4
This challenge requires identifying and reading the only human-readable (ASCII) file inside the `inhere` directory. Because all filenames start with a `-` character (`-file00` through `-file09`), we prepend `./` to the wildcard so the `file` and `cat` utilities do not misinterpret the filenames as command-line flags:

```bash
cd inhere
file ./*
cat ./-file07
```

<img width="900" height="431" alt="obraz" src="https://github.com/user-attachments/assets/57546921-8a2f-4544-be1e-20a29a2d4fa6" />


---

### Bandit Level 5
We need to locate a file inside the `inhere` directory hierarchy that meets three criteria: it is human-readable, not executable, and exactly `1033` bytes in size. Using `find` with the `-size 1033c` filter (where `c` denotes bytes) immediately pinpoints the target file:

```bash
cd inhere
find . -size 1033c ! -executable
cat ./maybehere07/.file2
```

<img width="1297" height="323" alt="obraz" src="https://github.com/user-attachments/assets/871f4612-5456-41c3-881d-955d0f1c634d" />


---

### Bandit Level 6
In this level, the target file is stored somewhere on the entire server filesystem and has the following properties:
1. Owned by user `bandit7` (`-user bandit7`)
2. Owned by group `bandit6` (`-group bandit6`)
3. Size of `33` bytes (`-size 33c`)

We run `find` starting from the root directory (`/`) and append `2>/dev/null` to redirect standard error (`stderr`) to `/dev/null`, filtering out all `Permission denied` messages:

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
cat /var/lib/dpkg/info/bandit7.password
```

<img width="761" height="179" alt="obraz" src="https://github.com/user-attachments/assets/95ff11bb-e828-456c-a1af-6922a8a2facb" />


---

### Bandit Level 7
The password is stored in `data.txt` right next to the word `millionth`. We read the file and pipe the output to `grep -i` (case-insensitive search) to isolate the matching line:

```bash
cat data.txt | grep -i millionth
```

<img width="761" height="138" alt="obraz" src="https://github.com/user-attachments/assets/e5f9cf1f-8591-4428-8539-de607214f8bb" />


---

### Bandit Level 8
We need to find the single line of text in `data.txt` that occurs only once. Running `uniq -u` alone does not work because `uniq` only compares adjacent lines. Therefore, we first pipe `data.txt` through `sort` to group identical lines together, and then pass the sorted output to `uniq -u` (adding `-c` also displays the occurrence count next to the result):

```bash
sort data.txt | uniq -uc
```

<img width="454" height="79" alt="obraz" src="https://github.com/user-attachments/assets/ce9d5323-763b-4b43-874a-d2456ff99c14" />


---

### Bandit Level 9
The password is stored in `data.txt` as human-readable text preceded by several `=` characters amidst binary data. We use the `strings` utility (which extracts printable character sequences of at least 4 characters from binary files) and pipe the output to `grep` to filter for lines containing `=`:

```bash
strings data.txt | grep "="
```

<img width="466" height="148" alt="obraz" src="https://github.com/user-attachments/assets/c2515777-4809-4a6a-8908-29805e421df9" />


---

### Bandit Level 10
The file `data.txt` contains Base64-encoded data. We decode it directly in the terminal using the `base64` utility with the `-d` (decode) flag:

```bash
base64 -d data.txt
```

<img width="509" height="98" alt="obraz" src="https://github.com/user-attachments/assets/4bd79dd9-d2bc-48c8-94a2-0f2a6c1e77bb" />


---

### Bandit Level 11
The contents of `data.txt` have been obfuscated using the **ROT13** cipher (all uppercase and lowercase letters rotated by 13 positions). We use the `tr` (translate) command to map `A-Za-z` to `N-ZA-Mn-za-m`:

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

<img width="607" height="98" alt="obraz" src="https://github.com/user-attachments/assets/4910c71d-5098-474d-8740-6ec8919dd1ad" />


---

### Bandit Level 12
The file `data.txt` is a hexdump of a file that has been repeatedly compressed using multiple algorithms.
1. We create a working directory in `/tmp` and copy `data.txt` into it.
2. We reverse the hexdump back into a raw binary file using `xxd -r`.
3. Before each decompression step, we inspect the file header using the `file` command, rename the file with the appropriate extension (`.gz`, `.bz2`, `.tar`), and unpack it using `gzip -d`, `bzip2 -d`, or `tar -xf` until we reach plain ASCII text:

```bash
mkdir /tmp/mojplik11 && cd /tmp/mojplik11
cp ~/data.txt .
xxd -r data.txt > mojplik12
file mojplik12
# Repeatedly inspect with 'file' and decompress using gzip -d, bzip2 -d, and tar -xf
```

<img width="1513" height="999" alt="obraz" src="https://github.com/user-attachments/assets/7c48d437-7244-4395-8efc-0d2b2a2fed04" />

<img width="1488" height="412" alt="obraz" src="https://github.com/user-attachments/assets/93fadb23-36d8-4c21-81d9-e09e83563960" />


---

### Bandit Level 13
The password for the next level is stored in `/etc/bandit_pass/bandit14`, which is only readable by `bandit14`. However, the current home directory contains a private SSH key (`sshkey.private`). We either connect directly from the server to `localhost` or copy the key to our local machine via `scp`, restrict its permissions with `chmod 600` (required by OpenSSH so the key is not rejected as overly permissive), and authenticate using the `-i` flag:

```bash
scp -P 2220 bandit13@bandit.labs.overthewire.org:sshkey.private .
chmod 600 sshkey.private
ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
cat /etc/bandit_pass/bandit14
```

<img width="1252" height="315" alt="obraz" src="https://github.com/user-attachments/assets/38832404-06db-4cf8-9c22-7079785139d5" />

<img width="1255" height="998" alt="obraz" src="https://github.com/user-attachments/assets/339f737d-3fa0-4794-9702-5e3cc3107c6b" />

<img width="518" height="85" alt="obraz" src="https://github.com/user-attachments/assets/73dd83e2-43bf-4612-8bf1-641f18a41eae" />


---

### Bandit Level 14
To retrieve the next password, we must submit the current level's password to a service listening locally on port `30000`. We pipe the contents of `/etc/bandit_pass/bandit14` directly into `nc` (Netcat):

```bash
cat /etc/bandit_pass/bandit14 | nc localhost 30000
```

<img width="698" height="126" alt="obraz" src="https://github.com/user-attachments/assets/f57ba7e9-94a1-472d-97a9-8f5be6e5543c" />


---

### Bandit Level 15
This level requires submitting the current password to port `30001` on `localhost` over an encrypted **SSL/TLS** connection. Instead of plain Netcat, we use `openssl s_client` to handle the TLS handshake and transmit the password from `/etc/bandit_pass/bandit15`:

```bash
cat /etc/bandit_pass/bandit15
openssl s_client -connect localhost:30001
```

<img width="770" height="988" alt="obraz" src="https://github.com/user-attachments/assets/e8382f48-17ed-4d8c-970c-76a63ef04d4d" />

<img width="1169" height="989" alt="obraz" src="https://github.com/user-attachments/assets/dab4de8c-10db-4103-ada8-5e5725c6a33e" />


---

### Bandit Level 16
We need to find which port on `localhost` in the range `31000–32000` is running an SSL/TLS service that accepts our current password (as opposed to simple `echo` servers).
1. First, we run an `nmap` service and version scan (`-sV`) across the specified port range:

```bash
nmap -sV -p 31000-32000 localhost
```

<img width="731" height="273" alt="obraz" src="https://github.com/user-attachments/assets/5f7cf14f-379d-4eda-b1dd-bdfbfc89656a" />


2. Port `31790` stands out as running an SSL service (`ssl/unknown`). We connect to it using `openssl s_client -quiet` and inject the current password via input redirection (`<`):

```bash
openssl s_client -quiet -connect localhost:31790 < /etc/bandit_pass/bandit16
```

<img width="969" height="984" alt="obraz" src="https://github.com/user-attachments/assets/44ff90ec-625f-4a82-aed7-bf433bd4775d" />


3. The server returns an OpenSSH private key for `bandit17`. We save it inside a temporary directory, apply `chmod 600`, copy it via `scp` (or use it locally), and authenticate into `bandit17`:

```bash
mkdir /tmp/klucz11 && cd /tmp/klucz11
nano klucz12
chmod 600 klucz12
```

<img width="897" height="198" alt="obraz" src="https://github.com/user-attachments/assets/ad2ccd98-3832-4711-923b-f9c9274880f8" />

<img width="1804" height="357" alt="obraz" src="https://github.com/user-attachments/assets/83b90837-5d5a-49fe-8b7e-8e59147f44b4" />

<img width="1184" height="1004" alt="obraz" src="https://github.com/user-attachments/assets/5154b686-96d1-4c40-873b-cb0b1d8747f1" />


---

### Bandit Level 17
There are two files in the home directory: `passwords.old` and `passwords.new`. The password for the next level is the only line that was changed in `passwords.new`. We compare the two files using `diff` (where `<` indicates the old file and `>` indicates the new file):

```bash
diff passwords.old passwords.new
```

<img width="525" height="151" alt="obraz" src="https://github.com/user-attachments/assets/75d0a49a-6ad3-4157-af9f-a97ad6c470cf" />


---

### Bandit Level 18
The password is stored in `readme` in the home directory, but someone modified `.bashrc` to immediately print `Byebye!` and terminate the session upon interactive login.
* **Method 1:** Pass the command directly at the end of the `ssh` invocation so it executes without spawning an interactive login shell (bypassing `.bashrc`):

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
```

<img width="1103" height="343" alt="obraz" src="https://github.com/user-attachments/assets/9246eed3-2286-45e7-904f-ac0dddad22a8" />


* **Method 2:** Force pseudo-terminal allocation (`-t`) and explicitly request `/bin/sh` instead of `/bin/bash`, which ignores `.bashrc`:

```bash
ssh -t bandit18@bandit.labs.overthewire.org -p 2220 "/bin/sh"
```

<img width="1101" height="424" alt="obraz" src="https://github.com/user-attachments/assets/d7eb5de0-b677-440f-bdbb-d0ee32cbccfd" />


---

### Bandit Level 19
In the home directory, we find a binary named `bandit20-do`. Inspecting its permissions with `ls -la` reveals `-rwsr-x---` owned by `bandit20:bandit19`.

<img width="650" height="185" alt="obraz" src="https://github.com/user-attachments/assets/a8077d2d-a4b0-4f0d-a091-1920813103bc" />


The `s` in the user execution bit indicates that the **SUID (Set Owner User ID)** bit is set. Whenever this binary is executed, it runs with the effective privileges of its owner (`bandit20`). We leverage it to read `/etc/bandit_pass/bandit20`:

```bash
./bandit20-do whoami
./bandit20-do cat /etc/bandit_pass/bandit20
```

<img width="634" height="279" alt="obraz" src="https://github.com/user-attachments/assets/1f8c606f-48e0-4190-9d80-cf31e80f58cf" />


---

### Bandit Level 20
This level provides a SUID binary named `suconnect` that connects to `localhost` on a user-specified port, reads a line of text, compares it to the current password (`bandit20`), and—if it matches—transmits the password for `bandit21`.
1. In the first terminal session, we set up a Netcat listener on an arbitrary high port (`55555`) primed to send the current password upon connection:

```bash
echo "<bandit20_password>" | nc -l -p 55555
```

2. In a second terminal session, we trigger `./suconnect 55555`:

```bash
./suconnect 55555
```

<img width="448" height="91" alt="obraz" src="https://github.com/user-attachments/assets/82e0415d-faf6-48ad-8c83-08eb5b366219" />


3. Returning to the first terminal window, we observe the new password received from `suconnect`:

<img width="750" height="90" alt="obraz" src="https://github.com/user-attachments/assets/0bfbb78d-851f-4344-9783-6eb0aa7b956a" />


---

### Bandit Level 21
A program is running automatically at regular intervals via `cron`. We inspect `/etc/cron.d/` and read `cronjob_bandit22`:

```bash
cat /etc/cron.d/cronjob_bandit22
```

Output:
```text
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
```

Next, we inspect the script `/usr/bin/cronjob_bandit22.sh`:

```bash
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

The script copies `bandit22`'s password into a world-readable file in `/tmp`. We simply read that file:

```bash
cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

<img width="710" height="79" alt="obraz" src="https://github.com/user-attachments/assets/c776e9e9-fc87-433b-92e2-bd9556cc9a75" />


---

### Bandit Level 22
Again, we check `/etc/cron.d/` and inspect `cronjob_bandit23`, which points to `/usr/bin/cronjob_bandit23.sh`:

```bash
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user$myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

When executed by `cron` under the `bandit23` user account, `$myname` evaluates to `bandit23`. The script hashes the string `I am user bandit23` with MD5 and writes the password to `/tmp/<md5_hash>`. We replicate the hash calculation ourselves and read the resulting file:

```bash
echo I am user bandit23 | md5sum | cut -d ' ' -f 1
cat /tmp/8ca319486bfbbc3663ea0fbe81326349
```

<img width="1511" height="451" alt="obraz" src="https://github.com/user-attachments/assets/93fcc6f8-a40e-4e5a-8cef-8254b4789bc3" />


---

### Bandit Level 23
Inspecting `/etc/cron.d/cronjob_bandit24` leads us to `/usr/bin/cronjob_bandit24.sh`:

```bash
#!/bin/bash

shopt -s nullglob
myname=$(whoami)

cd /var/spool/"$myname"/foo || exit
echo "Executing and deleting all scripts in /var/spool/$myname/foo:"
for i in * .*;
do
    if [ "$i" != "." ] && [ "$i" != ".." ];
    then
        echo "Handling $i"
        owner="$(stat --format "\%U" "./$i")"
        if [ "${owner}" = "bandit23" ] && [ -f "$i" ]; then
            timeout -s 9 60 "./$i"
        fi
        rm -rf "./$i"
    fi
done
```

This cron job runs every minute as `bandit24`, executes any script owned by `bandit23` placed inside `/var/spool/bandit24/foo/`, and then deletes it.
1. We create a temporary directory `/tmp/katalogtemp` and grant full permissions (`chmod 777 /tmp/katalogtemp`) so the `bandit24` process can write into it.
2. Inside, we write our payload script (`skrypt.sh`) and make it executable (`chmod +x skrypt.sh`):

```bash
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/katalogtemp/pswd.txt
```

3. We copy `skrypt.sh` to `/var/spool/bandit24/foo/`, wait up to 60 seconds for the cron job to execute it with `bandit24` privileges, and read `/tmp/katalogtemp/pswd.txt`:

<img width="645" height="123" alt="obraz" src="https://github.com/user-attachments/assets/e4e20007-68bb-4f45-8ede-c2add823b0ae" />


---

### Bandit Level 24
A daemon listening on port `30002` requires the current password (`bandit24`) followed by a secret 4-digit PIN (`0000–9999`), separated by a space.
We use a Bash `for` loop with brace expansion `{0000..9999}` to generate a wordlist of all 10,000 combinations, pipe it into `nc localhost 30002`, and use `grep -v "Wrong"` to filter out failed attempts:

```bash
mkdir /tmp/folder_temp && cd /tmp/folder_temp
for pin in {0000..9999}; do echo "$(cat /etc/bandit_pass/bandit24)$pin"; done > pswd.txt
nc localhost 30002 < pswd.txt | grep -v "Wrong"
```

<img width="1506" height="322" alt="obraz" src="https://github.com/user-attachments/assets/80730d89-5c10-4359-8639-b3fa246cb568" />


---

### Bandit Level 25
In the home directory, we find `bandit26.sshkey`. However, attempting to SSH into `bandit26` immediately closes the connection. Checking `/etc/passwd` reveals that `bandit26`'s default login shell is not `/bin/bash`, but `/usr/bin/showtext`:

```bash
grep bandit26 /etc/passwd
cat /usr/bin/showtext
```

```bash
#!/bin/sh
export TERM=linux
exec more ~/text.txt
exit 0
```

<img width="718" height="234" alt="obraz" src="https://github.com/user-attachments/assets/b9ca7f2a-90ca-4304-bcac-936139162183" />


Because `more` automatically exits when the entire file fits on screen, we **shrink our terminal window** so `~/text.txt` exceeds the viewport height, forcing `more` to pause in interactive paging mode.
1. Once paused inside `more`, we press `v` to launch the `vim` text editor.
2. Inside `vim`, we override the shell variable and spawn an interactive Bash session:

```vim
:set shell=/bin/bash
:shell
```

3. Now inside an interactive `/bin/bash` shell as `bandit26`, we read `/etc/bandit_pass/bandit26`:

<img width="489" height="102" alt="obraz" src="https://github.com/user-attachments/assets/993027d1-3502-4b7e-b2b5-4271a24bb58e" />


---

### Bandit Level 26
Staying in the `bandit26` session, we find a SUID binary named `bandit27-do` (owned by `bandit27`). Just like in Level 19, we execute it to read `/etc/bandit_pass/bandit27` with `bandit27`'s privileges:

```bash
./bandit27-do cat /etc/bandit_pass/bandit27
```

<img width="628" height="143" alt="obraz" src="https://github.com/user-attachments/assets/05f95b74-05c0-489b-add7-5ef6bece84ed" />


---

### Bandit Level 27
From this level onward, challenges focus on **Git repository enumeration**. We clone the remote repository over SSH (port `2220`) and inspect the `README` file inside:

```bash
git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
cd repo
cat README
```

<img width="1427" height="604" alt="obraz" src="https://github.com/user-attachments/assets/584b53f9-f6e9-45e5-8367-9347488e71d7" />


---

### Bandit Level 28
After cloning the repository and reading `README.md`, we see that the password has been redacted with `xxxxxxxxxx`. Since Git tracks full version history, we inspect the commit history with `git log` (spotted commit: *"fix info leak"*) and run `git diff` between the two commits to expose the plaintext password before it was removed:

```bash
git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo
cd repo
git log
git diff <previous_commit_hash> <latest_commit_hash>
```

<img width="1543" height="1005" alt="obraz" src="https://github.com/user-attachments/assets/619dcbc5-d726-4615-8197-968981fe58cd" />


---

### Bandit Level 29
In this repository, `README.md` on the `master` branch states `<no passwords in production!>`. This hints that credentials might exist on a non-production branch. We list all local and remote branches using `git branch -a`, switch to the `dev` branch with `git checkout dev`, and read `README.md`:

```bash
git branch -a
git checkout dev
cat README.md
```

<img width="802" height="754" alt="obraz" src="https://github.com/user-attachments/assets/1a791489-a22c-43d3-afb5-f13aef922f32" />


---

### Bandit Level 30
Here, `README.md` contains no useful information, and checking `git log` and `git branch -a` reveals nothing unusual. We check for **Git tags** using `git tag`, discover a tag named `secret`, and inspect its contents using `git show secret`:

```bash
git tag
git show secret
```

<img width="928" height="388" alt="obraz" src="https://github.com/user-attachments/assets/99a921e8-c263-40f0-b542-53906ad0970f" />


---

### Bandit Level 31
The `README.md` instructs us to push a file named `key.txt` containing `'May I come in?'` to the `master` branch. Because `.gitignore` blocks `*.txt` files by default, we use `git add -f` to force-stage the file, commit it, and push it to `origin master`. The remote Git hook validates the file and outputs the password for `bandit32` directly in the terminal response:

```bash
echo 'May I come in?' > key.txt
git add -f key.txt
git commit -m "Add key.txt"
git push origin master
```

<img width="767" height="178" alt="obraz" src="https://github.com/user-attachments/assets/a5277b25-7416-488d-bfd0-ad3aefda7c3d" />

<img width="906" height="768" alt="obraz" src="https://github.com/user-attachments/assets/bd783993-e5da-450a-8760-d32f0449ccee" />


---

### Bandit Level 32
Upon logging into `bandit32`, we are placed inside a restricted shell called `UPPERCASE SHELL` (`>>`), which converts all lowercase letters typed by the user into uppercase before executing them.
To escape this restricted shell, we input `$0` (which contains no alphabetic characters and expands to the first positional argument—the underlying `/bin/sh` interpreter). This spawns a standard shell running with `bandit33` privileges (verified via `id`), allowing us to read `/etc/bandit_pass/bandit33`:

```bash
>> $0
$id$ cat /etc/bandit_pass/bandit33
```

<img width="767" height="178" alt="obraz" src="https://github.com/user-attachments/assets/55b10608-74da-4571-90cf-8f5d0a09af29" />


---

### Bandit Level 33
Reading `README.txt` confirms completion of the final level of the OverTheWire Bandit wargame:

```bash
ls
cat README.txt
```

<img width="984" height="277" alt="obraz" src="https://github.com/user-attachments/assets/dced1b5c-e414-4436-ad95-4c40e1b1653f" />
