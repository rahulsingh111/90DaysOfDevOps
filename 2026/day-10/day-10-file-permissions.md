## Day 10

### Task 1: Create Files (10 minutes)

1. Create empty file `devops.txt` using `touch`---> touch devops.txt
2. Create `notes.txt` with some content using `cat` or `echo`----> echo "this is notes.tx" > notes.txt
3. Create `script.sh` using `vim` with content: `echo "Hello DevOps"`---> vim script.sh

content of script.sh
#!/bin/bash
echo "Hello DevOps"

ls -l

-rw-r--r-- 1 rahul rahul  0 Feb 23 17:39 devops.txt
-rw-r--r-- 1 rahul rahul 17 Feb 23 17:40 notes.txt
-rwxr-xr-x 1 rahul rahul 33 Feb 23 17:41 script.sh


### Task 2: Read Files (10 minutes)

1. cat notes.txt
this is notes.tx

2. vim -R script.sh

3. head -n 5 /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync

4. tail -n 5 /etc/passwd
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
uuidd:x:106:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:107:113::/nonexistent:/usr/sbin/nologin
landscape:x:108:115::/var/lib/landscape:/usr/sbin/nologin
rahul:x:1000:1000:,,,:/home/rahul:/bin/bash

### Task 3: Understand Permissions (10 minutes)

ls -l devops.txt notes.txt script.sh

-rw-r--r-- 1 rahul rahul  0 Feb 23 17:39 devops.txt ----> owner: read/write, group: read, others: read
-rw-r--r-- 1 rahul rahul 17 Feb 23 17:40 notes.txt ---> owner: read/write, group: read, others: read
-rwxr-xr-x 1 rahul rahul 33 Feb 23 17:41 script.sh --> owner: read/write/execute, group: read/execute, others: read/execute

### Task 4: Modify Permissions (20 minutes)

1. chmod+x script.sh --> make script.sh executable
./script.sh
Hello DevOps

2. chmod -w devops.txt --> set devops.txt to read-only

3. chmod 640 notes.txt --> set notes.txt to rw for owner, r for group, none for others

4. mkdir project --> create directory project

5. chmod 755 project --> set permissions for project directory

### Task 5: Test Permissions (10 minutes)

1. Try writing to a read-only file - what happens?
echo "Trying to write to devops.txt" >> devops.txt
bash: devops.txt: Permission denied

2. Try executing a file without execute permission
chmod -x script.sh --> remove execute permission from script.sh
./script.sh

3. Document the error messages
bash: ./script.sh: Permission denied