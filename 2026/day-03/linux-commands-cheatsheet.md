# linux command cheatsheet

--------- Process management --------

- ps aux (display running processes)
- pwd (list current directory)
- kill PID (to kill the processes)
- ps -ef | grep <process name> (to list the processes related to a particular process)
- top/htop (real-time process and CPU usage)
- uptime (shows system's uptime)
- df -h (shows file system)
- free -h (shows memory usage)
- systemctl status <process name> (check the status of the process, active or inactive)

---------- File System ------------

- touch (create a file)
- mkdir (create directory), rmdir (remove directory)
- cp (to copy), mv (to move), rm (to remove), rm -f (remove forcefully)
- cat (displays the content of a file)
- tail -f /var/log/<logpath> (view logs in realtime)
- chmod (change ownership of the file)
- ls -l (shows the files details)
- du -sh * (Check disk usage per directory)

------- Network troubleshooting --------

- ping <website> (check the status of the website working or not)
- ip addr show (to see the ip address details)
- dig, traceroute (shows hops to reach the final destination)
- curl <website> (fetches data from a url)
- ss -tuln (lists listening ports and services)