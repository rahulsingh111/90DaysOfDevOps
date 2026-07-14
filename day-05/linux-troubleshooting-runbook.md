# RUNBOOK

Target Service (ssh)

- systemctl status ssh (Check service status)
 Observation: Service is active (running) since Fri 2026-02-06 17:29:06 UTC;  Server listening on 0.0.0.0 port 22

- journalctl -u ssh -n 50 --no-pager (check logs)
 Observation:  Server listening on 0.0.0.0 port 22 and ssh service is started also it lists the ip of the vm whose public key is accepted for ssh connection

- tail -n 50 /var/log/auth.log 
 Observation: shows  Loading rules from directory /usr/share/polkit-1/rules.d, shows open and closed sessions for users

- top -p 1390(pid of ssh)
 Observation: Single idle sshd master process (PID 1390) in sleeping state, 0% CPU usage, ~7 MB resident memory, no active sessions or resource contention detected.

- who (check who is logged in currently)
 Observation: shows a single user (ubuntu) connected to the vm

- df -h / (checks disk space)
 Observation: shows 37% of disk space is utilized

- mkdir /tmp/runbook-demo
 Observation: created a folder runbook-demo inside tmp directory

- ss -tupln
 Observation: Network sockets list shows expected services listening on configured ports, no unexpected listeners detected, and no abnormal UDP/TCP bindings or conflicts observed.

- iostat
 Observation: indicates the system is largely idle,steady I/O on the primary NVMe disk and no signs of disk saturation, latency issues, or I/O contention.

**If this worsens (next steps)**

Restart strategy
Open a secondary SSH session, validate config with sshd -t and restart primary ssh services with (systemctl restart ssh)

Increase log verbosity
Temporarily set <LogLevel DEBUG> in /etc/ssh/sshd_config, reload with systemctl reload sshd, and inspect logs

Deep inspection
Attach strace -p <sshd_pid> to the affected child process to observe blocking calls and identify where sessions are not connecting