# Day 12 Task

**Mindset & plan:** I revisited my Day 01 learning plan and realized that my goals are still aligned with what I want to achieve. However, I am not getting enough time time to work on 90daysofdevops due to my work schedule but I am planning to revisit the lectures, understand the concepts and finish multiple days at on go.

**Processes & services:**
commands I ran
`ps aux | grep nginx` : I ran this command to check if nginx is running and to see the processes related to nginx. It showed me the master process and worker processes of nginx, confirming that it is active.

`journalctl -u nginx.service` : I used this command to view the logs of the nginx service. It provided me with detailed information about the service's activity, including any errors or warnings that might have occurred.

**File skills:**
`echo "this is a dummy file">> dummy.txt` : I ran this command to create a new file called dummy.txt and add the text "this is a dummy file" to it. This is a quick way to create and write to a file.

`chmod 756 dummy.txt` : I used this command to change the permissions of dummy.txt. The permissions set allow the owner to read, write, and execute the file, while the group and others can read and execute it.

`chown rahul:tech-team dummy.txt` : I ran this command to change the ownership of dummy.txt to the user "rahul" and the group "tech-team".

`ls -l dummy.txt` : I used this command to list the details of dummy.txt, including its permissions, ownership, and size. This helped me verify the changes I made with chmod and chown.

`cp dummy.txt dummy1.txt` : I ran this command to create a copy of dummy.txt and name it dummy1.txt. This is a simple way to duplicate a file.

`mkdir dummy, mv dummy.txt dummy1.txt dummy`: I ran these commands to create a new directory called dummy and move both dummy.txt and dummy1.txt into that directory. This helps in organizing files into folders.

**Cheat sheet refresh:** 
1. `ps aux` - This command is essential for checking running processes and their resource usage.
2. `systemctl status <service>` - This is crucial for checking the status of a service and troubleshooting issues.
3. `journalctl -u <service>` - This command is important for viewing logs related to a specific service, which is vital for debugging.
4. `chown` - This command is key for managing file ownership, which is important for access control.
5. `ls -l` - This command is fundamental for viewing file permissions and ownership, which is essential for managing files effectively.

## Mini Self-Check
These 3 commands save me the most time right now:
- `ps aux` because it allows me to quickly check the status of processes and identify any issues.
- `systemctl status <service>` because it provides a quick overview of the health of a service and helps in troubleshooting.
- `journalctl -u <service>` because it gives me detailed logs that are crucial for diagnosing problems with services.


To check if a service is healthy, I would run the following commands:
- `systemctl status <service>` to get an overview of the service's status and any potential issues.
- `journalctl -u <service>` to review the logs for any errors or warnings that might indicate problems with the service.

To safely change ownership and permissions without breaking access, I would use the following command:   `chown rahul:tech-team dummy.txt && chmod 756 dummy.txt` This command changes the ownership of dummy.txt to the user "rahul" and the group "tech-team", and then sets the permissions to allow the owner to read, write, and execute the file, while the group and others can read and execute it.

In the next 3 days, I will focus on improving my understanding of system monitoring and logging. I want to become more proficient in using tools like `journalctl` and `systemctl` to effectively monitor services and troubleshoot issues. I would like to gain confidence in interpreting logs and understanding the health of services, which is crucial for maintaining a stable and efficient system.