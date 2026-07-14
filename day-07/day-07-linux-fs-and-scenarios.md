# Day 7 task

/ :  this is the root directory, it contains all the directories (parent directory)
I would use this to find various applications installed in my machine and navigate through them

/home : this directory contains the folder for the users created in linux
I would use this to store thigs in the folders related to that particular user

/root : home directory of the root user, needs to be root to access this folder
I would use this to store files that should be accessed by root only

/etc : this directory contains the configuration files
I would use this file to change any configuration or settings. Ex:- set up or modify cron jobs

/var/log : Stores logs
I would use this to check the logs of different services, debug why things crashed or didn't start

/tmp : It stores temporary files that are cleaned once the system reboots
I would use this for testing purpose so that the data can be removed once the server restarts

/bin : it contains the binaries, various commands that are used to run the system

/usr/bin : this contains most of the user commands that we use 


/opt : this stores add on packages, large applications (third parties)

du -sh /var/log/* 2>/dev/null | sort -h | tail -5 : this command sorts the last largest 5 files occupying the disk space in ascending order in var/log directory and redirects the error output to /dev/null

cat /etc/hostname : shows the name of your vm, can be changed by modifying vi/etc/hostname

ls -la ~ : this command is used to list all files and directories in the current user's home directory, including hidden files and directories

**Scenario 1: Service Not Starting**
A web application service called 'myapp' failed to start after a server reboot.

Step 1: systemctl status myapp
Why: I will run this command to check if the service is running or running or not, if not I will start it by systemctl startnmyapp

Step 2: journalctl -u myapp -n 50`
Why: I will check the last 50 lines of the logs for myapp to see what cuased the problem

Step 3: systemctl is-enabled myapp
Why: I will check whether the myapp service is enabled to start automatically when the system boots or not

**Scenario 2: High CPU Usage** 
Your manager reports that the application server is slow.
You SSH into the server. What commands would you run to identify
which process is using high CPU?

Step 1: top/htop
Why: I will run this command to see which process are consuming too much of cpu

Step 2: ps aux --sort=-%cpu | head -10
Why: I will use this command to display the top 10 process consuming the cpu sorted in descending order

Step 3: kill PID/ kill -9 PID
Why; I will note the process id of the process which is consuming the cpu and kill it so that the cpu becomes stable

**Scenario 3: Finding Service Logs** 
A developer asks: "Where are the logs for the 'docker' service?"
The service is managed by systemd.
What commands would you use?

Step 1: systemctl status docker
Why: To check whether docker is up and running

Step 2: journalctl -u docker -n 50
Why: To display the last 50 lines of logs for docker service that can be shared with the developer

Step 3: journalctl -u docker -f
Why: This displays the logs for docker service in real time

**Scenario 4: File Permissions Issue** 
A script at /home/user/backup.sh is not executing.
When you run it: ./backup.sh
You get: "Permission denied"

Step 1: ls -l /home/user/backup.sh
Why: I will run this command to check the current permissions of the file, if there's no x in the file that means it is not executable

Step 2:  chmod +x /home/user/backup.sh
Why: With this command I will give the file executable permissions

Step 3: ls -l /home/user/backup.sh
Why: I will again verify whether x is added to the file or not which will make it executable

Step 4: ./backup.sh
Why: Will run the script