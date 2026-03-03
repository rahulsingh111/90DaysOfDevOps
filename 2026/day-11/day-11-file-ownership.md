### Task 1: Understanding Ownership (10 minutes)

AFter running ls-l i found the details about the files (owner and group) in the home directory. The owner is the user who created the file, while the group is a collection of users that can have specific permissions on the file. The owner has full control over the file, while the group can have read, write, or execute permissions based on the settings.

### Task 2: Basic chown Operations (20 minutes)

I created the file `devops-file.txt` with touch devops-file.txt and checked its owner using `ls -l`. I then changed the owner to `tokyo` using `sudo chown tokyo devops-file.txt` and verified the change. Next, I changed the owner to `berlin` and verified it again.

### Task 3: Basic chgrp Operations (15 minutes)

I created the file `team-notes.txt` and checked its group using `ls -l`. I created the group `heist-team` using `sudo groupadd heist-team` and changed the file group to `heist-team` using `sudo chgrp heist-team team-notes.txt`.

### Task 4: Combined Owner & Group Change (15 minutes)

I used the `chown professor:heist-team project-config.yaml` command to change both the owner and group of a file simultaneously, the owner to `professor` and the group to `heist-team`. I verified the changes using `ls -l` to ensure both the owner and group were updated correctly.

### Task 5: Recursive Ownership (20 minutes)

I created the directory structure for the heist project and used the `chown professor:planners -R heist-project/` command to change ownership recursively for all files and directories within `heist-project/`. I verified the changes using `ls -lR heist-project/` to ensure that all files and directories had the correct ownership.

### Task 6: Practice Challenge (20 minutes)

I didn't create the users `tokyo`, `berlin`, and `nairobi` as they were already there. I then created the directories anf files with `mkdir` and `touch`.I then assigned ownership of specific files to these users and set appropriate group permissions with the help of the command `chown user:group filename` to practice managing file ownership and access control effectively.