# Day 2 task

------------ Core components of linux ---------------

-- kernel (core)

- Main component of linux os and the core interface between a computer's hardware and its processes (communicate between two of them)
- kernel has 4 major jobs - memory management, process management, device drivers, system calls and security

-- User space

- Userspace is a space controlled by user (Most of the system’s functionality is implemented in Userspace) or a place where user and application resides
- this has limited privileges unlike root Userspace programs are executed in user mode. In Linux, the kernel provides userspace with a sandboxed environment, called user mode, that users can freely act in
- examples:- web browsers (firefox), cli tools (grep, awk, bash), system services (sshd, systemd)

-- Init/systemd

- Systemd is the first process started by linux and has PID (process id =1), it starts the services at boot and also restarts failed services
- Systemd is responsible for booting the system and managing all services, processes, and system resources and provides a unified, consistent framework for managing the entire Linux system
- systemd handles System logging via journald, Resource management using cgroups, On-demand service activation, Mount and automount points, network configuration, user sessions, and timers

----------- Processes in linux ----------------

- Process Creation in Linux begins with the fork() system call, which creates a new process by duplicating the calling (parent) process, child process inhibits most of the process from parent process but receives a unique process id
- After fork(), the child process typically uses the exec() family of system calls (e.g., execve, execvp) to replace its current program image with a new program. This two-step process—fork() followed by exec()—is fundamental for launching new applications.
- Kernel assigns a PID and tracks the process
- R-Running or ready to run process
- S- Sleeping process (waiting for an event or user input, can wake up)
- D- Uninterruptible (waiting for hardware/disk, can't be interrupted)
- T- Stopped/terminated (stopped by the user)
- Z- Zombie (Terminated but still listed in the process table until cleared by its parent process)

-------- Daily linux commands ---------------

- ps: to view running processes
- top/htop: to check the top processes consuming the cpu
- df -h: to check the disk space
- ls -l: to view the contents inside a directory (and file structure)
- systemctl: (systemcontroller) to start stop services
- journalctl: to view the logs
