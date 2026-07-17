1. What does `chmod 755 script.sh` do?
   Ans: it gives read, write and execute permissions to the owner of the file, and read and execute permissions to the group and others.

2. What is the difference between a process and a service?
   Ans: A process is an instance of a program that is currently running, while a service is a background process that runs continuously and provides functionality to other programs or users.
   
3. How do you find which process is using port 8080?
   Ans: netstat -tuln | grep 8080 or lsof -i :8080 can be used to find which process is using port 8080.

4. What does `set -euo pipefail` do in a shell script?
   Ans: `set -euo pipefail` is a command that sets the shell options to:
   - `-e`: Exit immediately if a command exits with a non-zero status.
   - `-u`: Treat unset variables as an error and exit immediately.
   - `-o pipefail`: Return the exit status of the last command in the pipeline that failed, rather than the exit status of the last command.

5. What is the difference between `git reset --hard` and `git revert`?
   Ans: `git reset --hard` moves the current branch pointer to a specified commit and discards all changes in the working directory and staging area, effectively erasing history. `git revert`, on the other hand, creates a new commit that undoes the changes made by a previous commit, preserving the history.

6. What branching strategy would you recommend for a team of 5 developers shipping weekly?
   Ans: For a team of 5 developers shipping weekly, I would recommend using GitFlow or GitHub Flow. GitFlow provides a structured branching model with separate branches for features, releases, and hotfixes, which can help manage development and releases effectively. GitHub Flow is simpler and works well for continuous deployment, where developers create feature branches and merge them into the main branch after review and testing. The choice depends on the team's workflow and release frequency.

7. What does `git stash` do and when would you use it?
   Ans: git stash temporarily saves changes in the working directory that are not ready to be committed, allowing you to switch branches or work on something else without losing your changes. You would use it when you need to quickly switch contexts or when you want to save your work-in-progress without committing it.

8. How do you schedule a script to run every day at 3 AM?
   Ans: You can use cron to schedule a script to run every day at 3 AM. Add a line to your crontab (crontab -e) like this: 0 3 * * * /path/to/your/script.sh

9.  What is the difference between `git fetch` and `git pull`?
   Ans: `git fetch` downloads changes from the remote repository but does not merge them into your local branch. `git pull` does the same but then immediately merges the changes into your current branch.

10. What is LVM and why would you use it instead of regular partitions?
    Ans: LVM (Logical Volume Manager) is a device mapper that provides logical volume management for Linux. It allows you to create, resize, and manage disk volumes in a more flexible way than traditional partitioning. You would use LVM instead of regular partitions because it provides better flexibility in managing disk space, allows for dynamic resizing of volumes, and enables features like snapshots and mirroring.

Teach it back:
`What is crontab, and why do sysadmins swear by it?`
Think of crontab as an alarm clock for your server, except instead of waking you up, it runs commands for you — automatically, on a schedule you set once and forget.
Every Linux system has a cron daemon quietly running in the background, checking a schedule file (the crontab) every minute. If it finds a job that matches the current time, it fires it off — no one has to be logged in, no one has to remember to run it manually.
The syntax looks cryptic at first — five fields for minute, hour, day, month, and weekday, followed by the command — but once you decode it, it's just "run this, at this time, this often." 0 3 * * * means "run at 3:00 AM, every day."
I use it for the boring-but-critical stuff: rotating logs, running nightly backups, kicking off health checks. The value isn't the command itself — it's that it happens reliably without a human in the loop. That's basically the whole promise of automation in one tool.