# Day 69 — Ansible Playbooks and Modules

> Reference notes from the playbook and module lab. The examples are written for the Ubuntu EC2 hosts used in this lab. Replace inventory names and paths to match your environment. Never commit private keys, AWS credentials, or unredacted public IPs.

## What I learned

Ad-hoc commands are useful for one-off checks. Playbooks describe a repeatable desired state in YAML: which hosts to configure, what tasks to run, and which modules should make the changes. Ansible modules are generally idempotent: rerunning a playbook should leave already-correct resources unchanged.

## Playbook structure

A play contains a host pattern and a sequence of tasks. A task invokes a module with arguments. A playbook can contain multiple plays for different host groups.

```yaml
---
- name: Configure web servers
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      ansible.builtin.apt:
        name: nginx
        state: present
```

- `hosts` selects an inventory group or host pattern.
- `become: true` runs tasks with elevated privileges, usually sudo. Set it at the play level for all tasks, or on an individual task when only that task needs it.
- `tasks` are run in order. A fatal task failure stops later tasks for that host; other hosts continue unless the play uses a strategy or error-handling setting that changes this behavior.
- A playbook can have multiple plays, such as separate web, app, and database plays.
- `ansible.builtin.*` is the fully qualified name for Ansible''s built-in modules.

Check inventory and YAML before applying:

```bash
ansible-inventory -i inventory.ini --graph
ansible-playbook -i inventory.ini install-nginx.yml --syntax-check
ansible-playbook -i inventory.ini install-nginx.yml --list-hosts
```

## First playbook: install and configure Nginx

For Ubuntu, use `apt`. For RHEL-family systems, use `dnf` or `yum`. The generic `package` module is a useful choice when the package name is the same across target operating systems.

```yaml
---
- name: Install and start Nginx on web servers
  hosts: web
  become: true

  tasks:
    - name: Refresh apt package lists
      ansible.builtin.apt:
        update_cache: true
        cache_valid_time: 3600

    - name: Install Nginx
      ansible.builtin.apt:
        name: nginx
        state: present

    - name: Start and enable Nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

    - name: Create a custom index page
      ansible.builtin.copy:
        content: "<h1>Deployed by Ansible</h1>\n"
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: ''0644''
```

Run with the inventory:

```bash
ansible-playbook -i inventory.ini install-nginx.yml
```

On the first run, tasks may report `changed`. On a second run, tasks should report `ok` if the desired state is already in place. This repeatability is idempotency.

## Essential modules

### Package management: `apt`, `dnf`, `yum`, and `package`

Use the OS-appropriate package manager to install or remove software. `present` ensures a package is installed; `absent` ensures it is removed.

```yaml
- name: Install utilities on Ubuntu
  ansible.builtin.apt:
    name:
      - git
      - curl
      - wget
      - tree
    state: present
    update_cache: true

- name: Remove a package when no longer needed
  ansible.builtin.apt:
    name: tree
    state: absent
```

On Fedora/RHEL-family machines, use `ansible.builtin.dnf` (or `yum` on older systems). Don''t call `dnf` on an Ubuntu host: Ansible detects `apt` as its package manager. A generic `ansible.builtin.package` task can select the system package manager, but it cannot make different package names equivalent.

### Service management: `service`

```yaml
- name: Ensure Nginx is running on web servers
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
  when: "''web'' in group_names"
```

The condition matters when a play targets `all`: app and database hosts may not have Nginx installed. Alternatively, target the service task with a separate play using `hosts: web`.

### Copy files: `copy`

The source file is on the control node, relative to the playbook (or in a role''s `files/` directory). The destination is on the managed host.

```yaml
- name: Copy application config
  ansible.builtin.copy:
    src: files/app.conf
    dest: /etc/app.conf
    owner: root
    group: root
    mode: ''0644''
```

### Manage files and directories: `file`

```yaml
- name: Create application directory
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
    owner: ubuntu
    group: ubuntu
    mode: ''0755''
```

Use the actual remote account for ownership. Ubuntu images commonly use `ubuntu`; Amazon Linux commonly uses `ec2-user`.

### Run a program: `command`

`command` runs a program directly without a shell. It is the preferred module for ordinary commands when no shell syntax is needed.

```yaml
- name: Check disk space
  ansible.builtin.command: df -h
  register: disk_output
  changed_when: false

- name: Print disk space
  ansible.builtin.debug:
    var: disk_output.stdout_lines
```

### Shell features: `shell`

`shell` runs through a shell, which enables pipes, redirects, expansions, and operators. Use it only when you need those features; prefer a dedicated Ansible module for system changes.

```yaml
- name: Count running processes
  ansible.builtin.shell: ps aux | wc -l
  register: process_count
  changed_when: false

- name: Show process count
  ansible.builtin.debug:
    msg: "Total processes: {{ process_count.stdout }}"
```

### Edit one line: `lineinfile`

```yaml
- name: Set timezone in environment
  ansible.builtin.lineinfile:
    path: /etc/environment
    line: ''TZ=Asia/Kolkata''
    create: true
```

## `command` versus `shell`

Both run commands on the managed host. `command` invokes the program directly and does not interpret shell syntax such as `|`, `>`, `&&`, variable expansion, or globbing. Use it for a command such as `df -h`.

`shell` passes the command to a shell, so shell syntax works. Use it for a pipeline such as `ps aux | wc -l`. Shell parsing adds quoting and injection risks, so use it only when shell features are needed. For actions like installing packages, managing services, or editing files, prefer Ansible modules.

## Handlers: act only after a change

A handler runs at the end of a play when a task notifies it. If the notifying task makes no change, the handler does not run. If several tasks notify the same handler, it normally runs once for that host.

Create `files/nginx.conf` containing a server block. Ubuntu''s main config includes `/etc/nginx/conf.d/*.conf`, so deploy a server-block file there instead of replacing the entire main `nginx.conf`.

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    server_name _;
    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Example task and handler:

```yaml
- name: Deploy Nginx server config
  ansible.builtin.copy:
    src: files/nginx.conf
    dest: /etc/nginx/conf.d/ansible-demo.conf
    owner: root
    group: root
    mode: ''0644''
  notify: Reload nginx
  when: "''web'' in group_names"

# Under the play''s handlers: key
handlers:
  - name: Reload nginx
    ansible.builtin.service:
      name: nginx
      state: reloaded
```

Run the playbook twice. On the first run, a newly copied or changed config should show `changed` and the handler should run. On the second run, an unchanged config should show `ok` and the handler should not appear. If the file already existed and matched before the first run, Ansible correctly skips the handler; change the local config to demonstrate a notification.

## Preview, diff, verbosity, and host limits

```bash
# Predict changes without applying them
ansible-playbook -i inventory.ini install-nginx.yml --check

# Preview and show file differences where supported
ansible-playbook -i inventory.ini nginx-config.yml --check --diff

# Increase diagnostic detail
ansible-playbook -i inventory.ini install-nginx.yml -v
ansible-playbook -i inventory.ini install-nginx.yml -vv
ansible-playbook -i inventory.ini install-nginx.yml -vvv

# Limit a run to a named host
ansible-playbook -i inventory.ini install-nginx.yml --limit web-server

# List selected hosts or tasks without running
ansible-playbook -i inventory.ini install-nginx.yml --list-hosts
ansible-playbook -i inventory.ini install-nginx.yml --list-tasks
```

- `--check` is a dry-run mode. Not every module can predict changes perfectly, and tasks that depend on earlier changes may not behave as in a normal run.
- `--diff` shows before/after content for supported file tasks. Review it for secrets before sharing logs.
- `-v`, `-vv`, and `-vvv` increase diagnostic output; higher verbosity can expose connection details, so redact logs before posting them.
- `--check --diff` helps review proposed changes before applying them, but it is a review aid rather than a guarantee. Validate playbooks in a safe environment too.

## Multiple plays for separate server groups

Use inventory groups so each role''s tasks run only on the intended hosts. This example uses generic package management for shared package names and Ubuntu''s package name for the MySQL-compatible client.

```yaml
---
- name: Configure web servers
  hosts: web
  become: true
  tasks:
    - name: Install Nginx
      ansible.builtin.package:
        name: nginx
        state: present
    - name: Start Nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

- name: Configure app servers
  hosts: app
  become: true
  tasks:
    - name: Install build tools
      ansible.builtin.package:
        name:
          - gcc
          - make
        state: present
    - name: Create app directory
      ansible.builtin.file:
        path: /opt/app
        state: directory
        mode: ''0755''

- name: Configure database servers
  hosts: db
  become: true
  tasks:
    - name: Install MySQL-compatible client on Ubuntu
      ansible.builtin.apt:
        name: default-mysql-client
        state: present
        update_cache: true
    - name: Create data directory
      ansible.builtin.file:
        path: /var/lib/appdata
        state: directory
        mode: ''0700''
```

The distro''s package name matters. On Ubuntu, `mysql` may not be a package name; `default-mysql-client` is the distribution''s default MySQL-compatible client metapackage. Check available package names with `apt-cache search mysql-client` on the target when unsure.

## Lab troubleshooting notes

These are the errors encountered during this lab and what they mean:

- **Empty inventory / “Could not match supplied host pattern”:** Ansible did not load a matching host group. Point to the inventory with `-i inventory.ini`, verify the group name in the playbook, and inspect it with `ansible-inventory -i inventory.ini --graph`.
- **A play targeting `web` was skipped:** The `web` group was missing from the inventory Ansible used. Define it and place the intended web host in it.
- **APT package download returned 404:** the package index referenced a package file no longer available at the mirror. Refresh the cache (`apt update` or an Ansible `apt` task with `update_cache: true`); if it continues, investigate mirror configuration.
- **Nginx service missing on app/db hosts:** the service task targeted hosts where Nginx was not installed. Target only `web` or condition the task on the `web` group.
- **“Unsupported parameters … update_cache” at a service task:** APT options were likely indented under the wrong module. Keep `update_cache` and `cache_valid_time` under `ansible.builtin.apt`.
- **DNF backend detection error on Ubuntu:** the play used `dnf` for an Ubuntu host. Use `apt` there, or `package` for OS selection when names are shared.
- **“No package matching mysql”:** the requested name was unavailable in Ubuntu''s package index. Use a valid distro package name such as `default-mysql-client` and verify with `apt-cache search`.

## Evidence and personal run record

Add screenshots from your own terminal if required by the course submission. Keep them free of public IPs, key paths, credentials, and other sensitive details. No screenshots were included with these notes.

Observed during this lab:

- Inventory hosts `web-server`, `app-server`, and `db-server` responded to fact gathering and the package tasks.
- The Nginx service was present on `web-server`; targeting `all` caused failures on app and db hosts until the service task was scoped to the web group.
- A later run showed `web-server` successful and the remaining essential module tasks succeeding there. Capture and attach the two handler demonstration runs separately if completing that verification.

## Learn in public

> Practiced Ansible playbooks and built a better understanding of modules, service targeting, package managers, and handlers. A repeatable playbook should report changes only when it has something new to apply.
>
> #90DaysOfDevOps #DevOpsKaJosh #TrainWithShubham

