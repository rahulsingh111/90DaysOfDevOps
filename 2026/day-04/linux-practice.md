# Day 4 task

- ps aux (Displayed a list of all running processes with user, PID, CPU, and memory usage)
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.7  1.4  22116 13508 ?        Ss   17:34   0:01 /sbin/init
root           2  0.0  0.0      0     0 ?        S    17:34   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        S    17:34   0:00 [pool_workqueue_release]
root           4  0.0  0.0      0     0 ?        I<   17:34   0:00 [kworker/R-rcu_gp]
root           5  0.0  0.0      0     0 ?        I<   17:34   0:00 [kworker/R-sync_wq]

- top (shows live running process)
top - 18:00:10 up 25 min,  1 user,  load average: 0.04, 0.01, 0.00
Tasks: 115 total,   1 running, 114 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :    914.2 total,    192.6 free,    384.0 used,    502.6 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.    530.2 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
    564 root      20   0 1802028  48060  34728 S   0.7   5.1   0:01.30 containerd
   2818 root      20   0       0      0      0 I   0.3   0.0   0:00.05 kworker/u8:0-events_power_efficient
      1 root      20   0   22660  14060   9624 S   0.0   1.5   0:05.03 systemd
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kthreadd
      3 root      20   0       0      0      0 S   0.0   0.0   0:00.00 pool_workqueue_release
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-rcu_gp
      5 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-sync_wq
      6 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-kvfree_rcu_reclaim

- sudo apt install apache2 (used to install packages)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  apache2-bin apache2-data apache2-utils libapr1t64 libaprutil1-dbd-sqlite3 libaprutil1-ldap libaprutil1t64 liblua5.4-0 ssl-cert
Suggested packages:
  apache2-doc apache2-suexec-pristine | apache2-suexec-custom www-browser
The following NEW packages will be installed:
  apache2 apache2-bin apache2-data apache2-utils libapr1t64 libaprutil1-dbd-sqlite3 libaprutil1-ldap libaprutil1t64 liblua5.4-0 ssl-cert
0 upgraded, 10 newly installed, 0 to remove and 2 not upgraded.
Need to get 2090 kB of archives.
After this operation, 8113 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/main amd64 libapr1t64 amd64 1.7.2-3.1ubuntu0.1 [108 kB]
Get:2 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble/main amd64 libaprutil1t64 amd64 1.6.3-1.1ubuntu7 [91.9 kB]
Get:3 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble/main amd64 libaprutil1-dbd-sqlite3 amd64 1.6.3-1.1ubuntu7 [11.2 kB]
Get:4 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble/main amd64 libaprutil1-ldap amd64 1.6.3-1.1ubuntu7 [9116 B]
Get:5 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble/main amd64 liblua5.4-0 amd64 5.4.6-3build2 [166 kB]
Get:6 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/main amd64 apache2-bin amd64 2.4.58-1ubuntu8.10 [1334 kB]
Get:7 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/main amd6

- ps aux | grep apache2 (shows the process related to specific service)
www-data    2638  0.0  0.1   3620  1628 ?        Ss   17:42   0:00 /usr/bin/htcacheclean -d 120 -p /var/cache/apache2/mod_cache_disk -l 300M -n
ubuntu      2810  0.0  0.2   7076  2224 pts/0    S+   17:44   0:00 grep --color=auto apache2

- sudo kill 1624 (killed the process having id 1624)

- systemctl status apache2.service (shows the current status of apache service)
× apache2.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/apache2.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Thu 2026-02-05 17:42:53 UTC; 5min ago
       Docs: https://httpd.apache.org/docs/2.4/
        CPU: 25ms


- journalctl -xeu apache2.service (shows the logs related to apache service)
Feb 05 17:50:20 ip-172-31-24-131 systemd[1]: apache2.service: Control process exited, code=exited, status=1/FAILURE
░░ Subject: Unit process exited
░░ Defined-By: systemd
░░ Support: http://www.ubuntu.com/support
░░
░░ An ExecStart= process belonging to unit apache2.service has exited.
░░
░░ The process' exit code is 'exited' and its exit status is 1.
Feb 05 17:50:20 ip-172-31-24-131 systemd[1]: apache2.service: Failed with result 'exit-code'.
░░ Subject: Unit failed
░░ Defined-By: systemd
░░ Support: http://www.ubuntu.com/support
░░
░░ The unit apache2.service has entered the 'failed' state with result 'exit-code'.
Feb 05 17:50:20 ip-172-31-24-131 systemd[1]: Failed to start apache2.service - The Apache HTTP Server.

- ss -tulnp | grep :80 (shows the process using that specific port)
tcp   LISTEN 0      511               0.0.0.0:80         0.0.0.0:*    users:(("nginx",pid=594,fd=5),("nginx",pid=593,fd=5),("nginx",pid=592,fd=5))
tcp   LISTEN 0      511                  [::]:80            [::]:*    users:(("nginx",pid=594,fd=6),("nginx",pid=593,fd=6),("nginx",pid=592,fd=6))


- systemctl list-units
  UNIT                                                                         LOAD   ACTIVE SUB       DESCRIPTION
  proc-sys-fs-binfmt_misc.automount                                            loaded active running   Arbitrary Executable File Formats File System Automount Point
  sys-devices-pci0000:00-0000:00:04.0-nvme-nvme0-nvme0n1-nvme0n1p1.device      loaded active plugged   Amazon Elastic Block Store cloudimg-rootfs
  sys-devices-pci0000:00-0000:00:04.0-nvme-nvme0-nvme0n1-nvme0n1p14.device     loaded active plugged   Amazon Elastic Block Store 14
  sys-devices-pci0000:00-0000:00:04.0-nvme-nvme0-nvme0n1-nvme0n1p15.device     loaded active plugged   Amazon Elastic Block Store UEFI
  sys-devices-pci0000:00-0000:00:04.0-nvme-nvme0-nvme0n1-nvme0n1p16.device     loaded active plugged   Amazon Elastic Block Store BOOT
  sys-devices-pci0000:00-0000:00:04.0-nvme-nvme0-nvme0n1.device                loaded active plugged   Amazon Elastic Block Store
  sys-devices-pci0000:00-0000:00:05.0-net-ens5.device                          loaded active plugged   Elastic Network Adapter (ENA)
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.1-tty-ttyS1.device loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/serial8250:0.1/tty/ttyS1
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.2-tty-ttyS2.device loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/serial8250:0.2/tty/ttyS2
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.3-tty-ttyS3.device loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/serial8250:0.3/tty/ttyS3
  sys-devices-pnp0-00:04-00:04:0-00:04:0.0-tty-ttyS0.device                    loaded active plugged   /sys/devices/pnp0/00:04/00:04:0/00:04:0.0/tty/ttyS0
  sys-devices-virtual-block-loop0.device                                       loaded active plugged   /sys/devices/virtual/block/loop0
  sys-devices-virtual-block-loop1.device                                       loaded active plugged   /sys/devices/virtual/block/loop1
  sys-devices-virtual-block-loop2.device                                       loaded active plugged   /sys/devices/virtual/block/loop2
  sys-devices-virtual-block-loop3.device                                       loaded active plugged   /sys/devices/virtual/block/loop3
  sys-devices-virtual-block-loop4.device                                       loaded active plugged   /sys/devices/virtual/block/loop4

- systemctl status ssh (shows the status of ssh service)
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/ssh.service.d
             └─ec2-instance-connect.conf
     Active: active (running) since Thu 2026-02-05 17:36:58 UTC; 25min ago
TriggeredBy: ● ssh.socket
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 1179 (sshd)
      Tasks: 1 (limit: 1017)
     Memory: 4.8M (peak: 5.5M)
        CPU: 54ms
     CGroup: /system.slice/ssh.service
             └─1179 "sshd: /usr/sbin/sshd -D -o AuthorizedKeysCommand /usr/share/ec2-instance-connect/eic_run_authorized_keys %u %f -o AuthorizedKeysCommandUser ec2-instance-connect [listener] 0 of 10-100 star>

- tail -n 30 syslog (shows last 30 lines of syslog)
2026-02-05T17:49:46.077767+00:00 ip-172-31-24-131 systemd[1]: apache2.service: Control process exited, code=exited, status=1/FAILURE
2026-02-05T17:49:46.077855+00:00 ip-172-31-24-131 systemd[1]: apache2.service: Failed with result 'exit-code'.
2026-02-05T17:49:46.078169+00:00 ip-172-31-24-131 systemd[1]: Failed to start apache2.service - The Apache HTTP Server.
2026-02-05T17:49:56.794681+00:00 ip-172-31-24-131 systemd[1]: Starting systemd-tmpfiles-clean.service - Cleanup of Temporary Directories...
2026-02-05T17:49:56.895605+00:00 ip-172-31-24-131 systemd[1]: systemd-tmpfiles-clean.service: Deactivated successfully.
2026-02-05T17:49:56.895775+00:00 ip-172-31-24-131 systemd[1]: Finished systemd-tmpfiles-clean.service - Cleanup of Temporary Directories.
2026-02-05T17:50:06.212684+00:00 ip-172-31-24-131 systemd[1]: Starting sysstat-collect.service - system activity accounting tool...
2026-02-05T17:50:06.222433+00:00 ip-172-31-24-131 systemd[1]: sysstat-collect.service: Deactivated successfully.
2026-02-05T17:50:06.222676+00:00 ip-172-31-24-131 systemd[1]: Finished sysstat-collect.service - system activity accounting tool.
2026-02-05T17:50:20.080410+00:00 ip-172-31-24-131 systemd[1]: Starting apache2.service - The Apache HTTP Server...
