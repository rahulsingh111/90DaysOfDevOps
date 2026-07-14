### Task 1: Create Users
[root@rhelvm ~]# for user in tokyo berlin professor; do sudo useradd -m "$user"; done
useradd: user 'tokyo' already exists
useradd: user 'berlin' already exists
useradd: user 'professor' already exists
[root@rhelvm ~]# ls /home/
batman  berlin  clark  homer  jerry  lex  nairobi  nexus  professor  rahul  seinfield  tokyo
[root@rhelvm ~]#



tokyo:x:1003:1005::/home/tokyo:/bin/bash
berlin:x:1004:1006::/home/berlin:/bin/bash
professor:x:1005:1007::/home/professor:/bin/bash

### Task 2: Create Groups
for group in developers admins; do sudo groupadd "$group"; done
developers:x:1008:
admins:x:1009:

### Task 3: Assign to Groups
[rahul@rhelvm ~]$ sudo usermod -aG developers tokyo
[rahul@rhelvm ~]$ sudo usermod -aG developers,admins berlin
[rahul@rhelvm ~]$ sudo usermod -aG admins professor
[rahul@rhelvm ~]$ for user in tokyo berlin professor; do id "$user"; done
uid=1003(tokyo) gid=1005(tokyo) groups=1005(tokyo),1008(developers)
uid=1004(berlin) gid=1006(berlin) groups=1006(berlin),1008(developers),1009(admins)
uid=1005(professor) gid=1007(professor) groups=1007(professor),1009(admins)

### Task 4: Shared Directory

[root@rhelvm ~]# mkdir /opt/dev-project
[root@rhelvm ~]# chown :developers /opt/dev-project/
[root@rhelvm ~]# chmod 775 /opt/dev-project/
[root@rhelvm ~]# su -tokyo -c "touch /opt/dev-project/tokyo_file.txt"
su: invalid option -- 't'
Try 'su --help' for more information.
[root@rhelvm ~]# su - tokyo -c "touch /opt/dev-project/tokyo_file.txt"
[root@rhelvm ~]# su - berlin -c "touch /opt/dev-project/berlin_file.txt"
[root@rhelvm ~]# ls -l /opt/dev-project/
total 0
-rw-r--r--. 1 berlin berlin 0 Feb 13 03:47 berlin_file.txt
-rw-r--r--. 1 tokyo  tokyo  0 Feb 13 03:47 tokyo_file.txt

### Task 5: Team Workspace
[root@rhelvm ~]# mkdir /opt/dev-project
[root@rhelvm ~]# chown :developers /opt/dev-project/
[root@rhelvm ~]# chmod 775 /opt/dev-project/
[root@rhelvm ~]# su -tokyo -c "touch /opt/dev-project/tokyo_file.txt"
su: invalid option -- 't'
Try 'su --help' for more information.
[root@rhelvm ~]# su - tokyo -c "touch /opt/dev-project/tokyo_file.txt"
[root@rhelvm ~]# su - berlin -c "touch /opt/dev-project/berlin_file.txt"
[root@rhelvm ~]# ls -l /opt/dev-project/
total 0
-rw-r--r--. 1 berlin berlin 0 Feb 13 03:47 berlin_file.txt
-rw-r--r--. 1 tokyo  tokyo  0 Feb 13 03:47 tokyo_file.txt
[root@rhelvm ~]# useradd -m nairobi
[root@rhelvm ~]# ls /home
batman  berlin  clark  homer  jerry  lex  nairobi  nexus  professor  rahul  seinfield  tokyo
[root@rhelvm ~]# groupadd project-team
[root@rhelvm ~]# for user in nairobi,tokyo; do sudo usermod -aG project-team "$user"; done
usermod: user 'nairobi,tokyo' does not exist
[root@rhelvm ~]# for user in nairobi , tokyo; do sudo usermod -aG project-team "$user"; done
usermod: user ',' does not exist
[root@rhelvm ~]# for user in nairobi tokyo; do sudo usermod -aG project-team "$user"; done
[root@rhelvm ~]# mkdir /opt/team-workpsace
[root@rhelvm ~]# chown :project-team /opt/dev-project
[root@rhelvm ~]# chmod 775 /opt/dev-project/
[root@rhelvm ~]# su - nairobi -c "touch /opt/dev-project/nairobi_data.txt"
[root@rhelvm ~]# ls -l /opt/dev-project/
total 0
-rw-r--r--. 1 berlin  berlin  0 Feb 13 03:47 berlin_file.txt
-rw-r--r--. 1 nairobi nairobi 0 Feb 13 03:56 nairobi_data.txt
-rw-r--r--. 1 tokyo   tokyo   0 Feb 13 03:47 tokyo_file.txt

