# Day 68 — Introduction to Ansible and Inventory Setup

> Lab record for learning Ansible's agentless configuration management workflow. Replace the bracketed lab fields and sample output with details from your own run before submitting or sharing this file. Never commit private keys, AWS credentials, or unredacted public IPs.

## Goals

- Understand configuration management and Ansible's architecture.
- Use a control node to manage Linux EC2 instances over SSH.
- Organize managed nodes into inventory groups.
- Run ad-hoc commands and verify their results
## 1. What is Ansible?

Configuration management describes and applies the desired state of machines: which packages are installed, which services run, which users and files exist, and how those settings are maintained. It makes server setup repeatable, reduces manual drift and mistakes, and helps rebuild or update fleets consistently.

Ansible is an automation and configuration-management tool. It is commonly used through readable YAML playbooks, and also supports one-off ad-hoc commands. It generally connects to Linux hosts over SSH and uses Python on the managed host for most modules. No continuously running Ansible agent is required on those hosts.

### How it compares

| Tool | Typical model | Practical distinction |
|---|---|---|
| Ansible | Agentless, push-based execution; usually SSH | Tasks can run from a control node without installing a persistent agent on Linux targets. |
| Puppet | Usually agent-based; desired state is commonly pulled from a Puppet server | Agents periodically check in and enforce catalog state. |
| Chef | Usually agent-based; recipes describe configuration | Chef Infra Client runs on nodes and converges them toward policy. |
| Salt | Supports agent-based (Salt minions) and agentless SSH modes | Often uses a master/minion event-driven model, with SSH available for agentless use. |

These are common deployment patterns, not absolute limits; each tool has additional modes and capabilities.

### Agentless and SSH

“Agentless” means no persistent Ansible service needs to be installed and kept running on each managed node. The control node opens an SSH connection, transfers or invokes the required module, collects its result, and closes the session. SSH authentication, network access, a compatible Python runtime for most Linux modules, and suitable privileges are still required. Ansible itself should be installed on the control node only.

### Architecture

```text
                      Inventory (hosts and groups)
                                  |
                                  v
Control node -- SSH --> web node   app node   db node
     |                    ^          ^         ^
     +-- modules ---------+----------+---------+
     +-- playbooks describe repeatable tasks for selected hosts
```

- **Control node:** runs `ansible` and `ansible-playbook`; for this lab, `[laptop / jump host / other]`.
- **Managed nodes:** the Linux servers Ansible configures; no Ansible agent is installed there.
- **Inventory:** names the nodes and organizes them into groups, along with connection variables.
- **Modules:** units of work such as `ping`, `copy`, `command`, `yum`, and `apt`.
- **Playbooks:** YAML files that apply a sequence of tasks to selected hosts. Ad-hoc commands are useful for quick operations; playbooks are better for repeatable changes.

## 2. Lab environment

**Provisioning method:** `[Terraform / AWS Console]`  
**Control node:** `[OS and machine name]`  
**Managed nodes:** 2–3 EC2 instances; record your actual values below.  
**Region:** `[AWS region]`  
**AMI:** `[Amazon Linux 2 / Ubuntu 22.04 / other]`  
**Instance type:** `t2.micro` (or actual type)  
**SSH ingress:** port 22 restricted to my control node's IP.  
**Key pair:** `[key-pair name; never include the private key]`

| Inventory name | Role | Instance ID (optional) | Public IP (redact if public) | OS / SSH user |
|---|---|---|---|---|
| `web-server` | Web | `[i-…]` | `[redacted]` | `[Amazon Linux / ec2-user]` |
| `app-server` | App | `[i-…]` | `[redacted]` | `[Amazon Linux / ec2-user]` |
| `db-server` | Database | `[i-…]` | `[redacted]` | `[Amazon Linux / ec2-user]` |

Before using Ansible, verify SSH access from the control node. For Ubuntu images, use `ubuntu` instead of `ec2-user`.

```bash
ssh -i ~/your-key.pem ec2-user@<public-ip-1>
ssh -i ~/your-key.pem ec2-user@<public-ip-2>
ssh -i ~/your-key.pem ec2-user@<public-ip-3>
```

### Install Ansible on the control node

Choose the command for the control node's operating system:

```bash
# macOS
brew install ansible

# Ubuntu/Debian
sudo apt update
sudo apt install ansible -y

# Amazon Linux / RHEL (availability varies by release)
sudo yum install ansible -y
# Alternative when a supported package is unavailable:
pip3 install --user ansible
```

Record verification details from your own environment:

```bash
ansible --version
```

- Installed on: `[control node and OS]`
- Ansible version: `[version]`
- Config file shown: `[path or “none”]`
- Python version shown: `[version]`

It is only needed on the control node because that is where inventory parsing, orchestration, and SSH connections originate. Managed nodes need SSH access and the runtime prerequisites for the modules being used, but not an Ansible daemon.

## 3. Inventory

Create a practice directory and save the following as `inventory.ini`. Replace the example addresses and key path locally. The key path is local to the control node; do not commit a private key.

```ini
[web]
web-server ansible_host=<PUBLIC_IP_1>

[app]
app-server ansible_host=<PUBLIC_IP_2>

[db]
db-server ansible_host=<PUBLIC_IP_3>

[all:vars]
ansible_user=ec2-user
ansible_ssh_private_key_file=~/your-key.pem
```

For Ubuntu, set `ansible_user=ubuntu`. Quote or use an absolute path for the key if `~` is not expanded as expected by your environment. On Unix-like systems, restrict the private key permissions, for example `chmod 400 ~/your-key.pem`.

### Group hierarchy and patterns

Add these groups to `inventory.ini` to create parent groups:

```ini
[application:children]
web
app

[all_servers:children]
application
db
```

Examples of host patterns:

```bash
ansible application -i inventory.ini -m ping   # web and app
ansible db -i inventory.ini -m ping            # db only
ansible all_servers -i inventory.ini -m ping   # every lab node
ansible 'web:app' -i inventory.ini -m ping      # web OR app
ansible 'all:!db' -i inventory.ini -m ping      # every host except db
```

### Optional project configuration

Save this as `ansible.cfg` in the practice directory to set defaults:

```ini
[defaults]
inventory = ./inventory.ini
host_key_checking = False
remote_user = ec2-user
private_key_file = ~/your-key.pem
```

Ansible looks for configuration in the current project directory, then the user's home configuration, then the system configuration. `host_key_checking = False` is convenient in a disposable lab, but disables SSH host identity verification; for persistent or sensitive environments, keep checking enabled and manage known hosts instead. If the hosts are Ubuntu, change `remote_user` accordingly.

With the config file in effect, `ansible all -m ping` should use the project inventory without `-i`. Confirm the active config path in `ansible --version` if it does not.

## 4. Ad-hoc commands and results

The Ansible `ping` module checks that Ansible can connect and run a module; it is not an ICMP network ping. Run:

```bash
ansible all -i inventory.ini -m ping
```

Expected result: each host reports `SUCCESS` and `"ping": "pong"`. Paste or summarize your actual result here. Do not claim success until the command has run.

### Five command examples to run and record

Capture the actual output from your lab. The snippets below are commands, not claimed run results.

**1. Check uptime on all servers**

```bash
ansible all -i inventory.ini -m command -a "uptime"
```

Result: `[paste or summarize actual output]`

**2. Check available memory on web servers**

```bash
ansible web -i inventory.ini -m command -a "free -h"
```

Result: `[paste or summarize actual output]`

**3. Check disk space on all servers**

```bash
ansible all -i inventory.ini -m command -a "df -h"
```

Result: `[paste or summarize actual output]`

**4. Install Git on the web group**

```bash
# Amazon Linux / RHEL
ansible web -i inventory.ini -m yum -a "name=git state=present" --become

# Ubuntu / Debian alternative
ansible web -i inventory.ini -m apt -a "name=git state=present update_cache=yes" --become
```

Result: `[paste or summarize actual output]`

**5. Copy a local file and verify it**

```bash
printf 'Hello from Ansible\n' > hello.txt
ansible all -i inventory.ini -m copy -a "src=hello.txt dest=/tmp/hello.txt"
ansible all -i inventory.ini -m command -a "cat /tmp/hello.txt"
```

Result: `[paste or summarize actual output]`

`--become` asks Ansible to escalate privileges on the managed host, commonly through `sudo`. Use it for tasks that require root privileges, such as installing packages or managing system services. The SSH user must be allowed to elevate privileges; Ansible may prompt for a password if required (use `-K` to prompt for the become password).

### `command` versus `shell`

The `command` module runs a program directly and does not interpret shell syntax. It is the safer default for simple commands. It does not support pipes, redirection, shell variables, or operators such as `&&`.

The `shell` module runs the command through a shell, so pipes, redirects, expansions, and shell operators work. Use it only when shell features are needed, because shell parsing can make quoting and behavior less predictable. Prefer dedicated Ansible modules for system changes where available.

## 5. Troubleshooting notes

- **Unreachable / timeout:** confirm the instance is running, has a reachable address, and its security group allows SSH from the control node.
- **Permission denied:** confirm the key file, key permissions, and AMI-specific SSH user (`ec2-user` or `ubuntu`).
- **Host key prompt or mismatch:** verify the server identity; do not blindly disable checking for a real environment.
- **Missing Python / module failure:** confirm the managed host has Python available, or use the appropriate Ansible raw/bootstrap approach for initial setup.
- **Privilege escalation failure:** confirm the remote user has sudo rights; use `--become` only where required.
- **Package module mismatch:** use `yum`/`dnf` for compatible RHEL-family hosts and `apt` for Debian-family hosts.

## 6. Evidence to add before submission

- [ ] Actual `ansible --version` details recorded.
- [ ] Actual instance count, OS, region, and provisioning method recorded.
- [ ] Inventory reviewed and all public addresses redacted if sharing publicly.
- [ ] Screenshot added showing successful `ansible all -m ping` results for every host.
- [ ] Five commands above run, with actual outputs summarized or attached.
- [ ] Confirm `ansible all -m ping` works without `-i` when `ansible.cfg` is present.
- [ ] No private key, AWS secret, or unredacted sensitive infrastructure detail is committed.

Suggested screenshot path: `2026/day-68/ansible-ping-success.png`. Add the image to the repository and embed it here:

```markdown
![Successful Ansible ping to all managed nodes](ansible-ping-success.png)
```

## Learn in public

> Started the Ansible journey today — set up a control node, created an inventory with three EC2 instances, and ran ad-hoc commands to manage all servers from one terminal. No agents installed on the managed nodes; Ansible connects over SSH.
>
> #90DaysOfDevOps #DevOpsKaJosh #TrainWithShubham

# Day 68 — Introduction to Ansible and Inventory Setup

> Lab record for learning Ansible's agentless configuration management workflow. Replace the bracketed lab fields and sample output with details from your own run before submitting or sharing this file. Never commit private keys, AWS credentials, or unredacted public IPs.

## Goals

- Understand configuration management and Ansible's architecture.
- Use a control node to manage Linux EC2 instances over SSH.
- Organize managed nodes into inventory groups.
- Run ad-hoc commands and verify their results.

## 1. What is Ansible?

Configuration management describes and applies the desired state of machines: which packages are installed, which services run, which users and files exist, and how those settings are maintained. It makes server setup repeatable, reduces manual drift and mistakes, and helps rebuild or update fleets consistently.

Ansible is an automation and configuration-management tool. It is commonly used through readable YAML playbooks, and also supports one-off ad-hoc commands. It generally connects to Linux hosts over SSH and uses Python on the managed host for most modules. No continuously running Ansible agent is required on those hosts.

### How it compares

| Tool | Typical model | Practical distinction |
|---|---|---|
| Ansible | Agentless, push-based execution; usually SSH | Tasks can run from a control node without installing a persistent agent on Linux targets. |
| Puppet | Usually agent-based; desired state is commonly pulled from a Puppet server | Agents periodically check in and enforce catalog state. |
| Chef | Usually agent-based; recipes describe configuration | Chef Infra Client runs on nodes and converges them toward policy. |
| Salt | Supports agent-based (Salt minions) and agentless SSH modes | Often uses a master/minion event-driven model, with SSH available for agentless use. |

These are common deployment patterns, not absolute limits; each tool has additional modes and capabilities.

### Agentless and SSH

“Agentless” means no persistent Ansible service needs to be installed and kept running on each managed node. The control node opens an SSH connection, transfers or invokes the required module, collects its result, and closes the session. SSH authentication, network access, a compatible Python runtime for most Linux modules, and suitable privileges are still required. Ansible itself should be installed on the control node only.

### Architecture

```text
                      Inventory (hosts and groups)
                                  |
                                  v
Control node -- SSH --> web node   app node   db node
     |                    ^          ^         ^
     +-- modules ---------+----------+---------+
     +-- playbooks describe repeatable tasks for selected hosts
```

- **Control node:** runs `ansible` and `ansible-playbook`; for this lab, `[laptop / jump host / other]`.
- **Managed nodes:** the Linux servers Ansible configures; no Ansible agent is installed there.
- **Inventory:** names the nodes and organizes them into groups, along with connection variables.
- **Modules:** units of work such as `ping`, `copy`, `command`, `yum`, and `apt`.
- **Playbooks:** YAML files that apply a sequence of tasks to selected hosts. Ad-hoc commands are useful for quick operations; playbooks are better for repeatable changes.

## 2. Lab environment

**Provisioning method:** `[Terraform / AWS Console]`  
**Control node:** `[OS and machine name]`  
**Managed nodes:** 2–3 EC2 instances; record your actual values below.  
**Region:** `[AWS region]`  
**AMI:** `[Amazon Linux 2 / Ubuntu 22.04 / other]`  
**Instance type:** `t2.micro` (or actual type)  
**SSH ingress:** port 22 restricted to my control node's IP.  
**Key pair:** `[key-pair name; never include the private key]`

| Inventory name | Role | Instance ID (optional) | Public IP (redact if public) | OS / SSH user |
|---|---|---|---|---|
| `web-server` | Web | `[i-…]` | `[redacted]` | `[Amazon Linux / ec2-user]` |
| `app-server` | App | `[i-…]` | `[redacted]` | `[Amazon Linux / ec2-user]` |
| `db-server` | Database | `[i-…]` | `[redacted]` | `[Amazon Linux / ec2-user]` |

Before using Ansible, verify SSH access from the control node. For Ubuntu images, use `ubuntu` instead of `ec2-user`.

```bash
ssh -i ~/your-key.pem ec2-user@<public-ip-1>
ssh -i ~/your-key.pem ec2-user@<public-ip-2>
ssh -i ~/your-key.pem ec2-user@<public-ip-3>
```

### Install Ansible on the control node

Choose the command for the control node's operating system:

```bash
# macOS
brew install ansible

# Ubuntu/Debian
sudo apt update
sudo apt install ansible -y

# Amazon Linux / RHEL (availability varies by release)
sudo yum install ansible -y
# Alternative when a supported package is unavailable:
pip3 install --user ansible
```

Record verification details from your own environment:

```bash
ansible --version
```

- Installed on: `[control node and OS]`
- Ansible version: `[version]`
- Config file shown: `[path or “none”]`
- Python version shown: `[version]`

It is only needed on the control node because that is where inventory parsing, orchestration, and SSH connections originate. Managed nodes need SSH access and the runtime prerequisites for the modules being used, but not an Ansible daemon.

## 3. Inventory

Create a practice directory and save the following as `inventory.ini`. Replace the example addresses and key path locally. The key path is local to the control node; do not commit a private key.

```ini
[web]
web-server ansible_host=<PUBLIC_IP_1>

[app]
app-server ansible_host=<PUBLIC_IP_2>

[db]
db-server ansible_host=<PUBLIC_IP_3>

[all:vars]
ansible_user=ec2-user
ansible_ssh_private_key_file=~/your-key.pem
```

For Ubuntu, set `ansible_user=ubuntu`. Quote or use an absolute path for the key if `~` is not expanded as expected by your environment. On Unix-like systems, restrict the private key permissions, for example `chmod 400 ~/your-key.pem`.

### Group hierarchy and patterns

Add these groups to `inventory.ini` to create parent groups:

```ini
[application:children]
web
app

[all_servers:children]
application
db
```

Examples of host patterns:

```bash
ansible application -i inventory.ini -m ping   # web and app
ansible db -i inventory.ini -m ping            # db only
ansible all_servers -i inventory.ini -m ping   # every lab node
ansible 'web:app' -i inventory.ini -m ping      # web OR app
ansible 'all:!db' -i inventory.ini -m ping      # every host except db
```

### Optional project configuration

Save this as `ansible.cfg` in the practice directory to set defaults:

```ini
[defaults]
inventory = ./inventory.ini
host_key_checking = False
remote_user = ec2-user
private_key_file = ~/your-key.pem
```

Ansible looks for configuration in the current project directory, then the user's home configuration, then the system configuration. `host_key_checking = False` is convenient in a disposable lab, but disables SSH host identity verification; for persistent or sensitive environments, keep checking enabled and manage known hosts instead. If the hosts are Ubuntu, change `remote_user` accordingly.

With the config file in effect, `ansible all -m ping` should use the project inventory without `-i`. Confirm the active config path in `ansible --version` if it does not.

## 4. Ad-hoc commands and results

The Ansible `ping` module checks that Ansible can connect and run a module; it is not an ICMP network ping. Run:

```bash
ansible all -i inventory.ini -m ping
```

Expected result: each host reports `SUCCESS` and `"ping": "pong"`. Paste or summarize your actual result here. Do not claim success until the command has run.

### Five command examples to run and record

Capture the actual output from your lab. The snippets below are commands, not claimed run results.

**1. Check uptime on all servers**

```bash
ansible all -i inventory.ini -m command -a "uptime"
```

Result: `[paste or summarize actual output]`

**2. Check available memory on web servers**

```bash
ansible web -i inventory.ini -m command -a "free -h"
```

Result: `[paste or summarize actual output]`

**3. Check disk space on all servers**

```bash
ansible all -i inventory.ini -m command -a "df -h"
```

Result: `[paste or summarize actual output]`

**4. Install Git on the web group**

```bash
# Amazon Linux / RHEL
ansible web -i inventory.ini -m yum -a "name=git state=present" --become

# Ubuntu / Debian alternative
ansible web -i inventory.ini -m apt -a "name=git state=present update_cache=yes" --become
```

Result: `[paste or summarize actual output]`

**5. Copy a local file and verify it**

```bash
printf 'Hello from Ansible\n' > hello.txt
ansible all -i inventory.ini -m copy -a "src=hello.txt dest=/tmp/hello.txt"
ansible all -i inventory.ini -m command -a "cat /tmp/hello.txt"
```

Result: `[paste or summarize actual output]`

`--become` asks Ansible to escalate privileges on the managed host, commonly through `sudo`. Use it for tasks that require root privileges, such as installing packages or managing system services. The SSH user must be allowed to elevate privileges; Ansible may prompt for a password if required (use `-K` to prompt for the become password).

### `command` versus `shell`

The `command` module runs a program directly and does not interpret shell syntax. It is the safer default for simple commands. It does not support pipes, redirection, shell variables, or operators such as `&&`.

The `shell` module runs the command through a shell, so pipes, redirects, expansions, and shell operators work. Use it only when shell features are needed, because shell parsing can make quoting and behavior less predictable. Prefer dedicated Ansible modules for system changes where available.

## 5. Troubleshooting notes

- **Unreachable / timeout:** confirm the instance is running, has a reachable address, and its security group allows SSH from the control node.
- **Permission denied:** confirm the key file, key permissions, and AMI-specific SSH user (`ec2-user` or `ubuntu`).
- **Host key prompt or mismatch:** verify the server identity; do not blindly disable checking for a real environment.
- **Missing Python / module failure:** confirm the managed host has Python available, or use the appropriate Ansible raw/bootstrap approach for initial setup.
- **Privilege escalation failure:** confirm the remote user has sudo rights; use `--become` only where required.
- **Package module mismatch:** use `yum`/`dnf` for compatible RHEL-family hosts and `apt` for Debian-family hosts.

## 6. Evidence to add before submission

- [ ] Actual `ansible --version` details recorded.
- [ ] Actual instance count, OS, region, and provisioning method recorded.
- [ ] Inventory reviewed and all public addresses redacted if sharing publicly.
- [ ] Screenshot added showing successful `ansible all -m ping` results for every host.
- [ ] Five commands above run, with actual outputs summarized or attached.
- [ ] Confirm `ansible all -m ping` works without `-i` when `ansible.cfg` is present.
- [ ] No private key, AWS secret, or unredacted sensitive infrastructure detail is committed.

Suggested screenshot path: `2026/day-68/ansible-ping-success.png`. Add the image to the repository and embed it here:

```markdown
![Successful Ansible ping to all managed nodes](ansible-ping-success.png)
```

## Learn in public

> Started the Ansible journey today — set up a control node, created an inventory with three EC2 instances, and ran ad-hoc commands to manage all servers from one terminal. No agents installed on the managed nodes; Ansible connects over SSH.
>
> #90DaysOfDevOps #DevOpsKaJosh #TrainWithShubham

