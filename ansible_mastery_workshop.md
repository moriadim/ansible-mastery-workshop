# 🚀 Ansible Mastery Workshop: ESTIN Cloud Hub

> 💡 **Welcome to Cloud Automation**
> Our goal today is simple: Stop typing commands manually, and start defining your infrastructure as code. We are moving from *Mutable Infrastructure* (servers changing over time and breaking unexpectedly) to *Immutable Infrastructure* (servers built exactly as defined in code, every single time).

## 🧠 Part 1: The Core Philosophy & Architecture

### What is Ansible?
At its core, Ansible is an orchestration and configuration management tool. It reads simple YAML files (Playbooks) and ensures your remote servers match that exact configuration.

### The Architecture: "Agentless" Magic
Unlike other tools that require you to install software (agents) on every server, Ansible uses what you already have:

*   **Control Node**: Your laptop, or a CI/CD server where Ansible is installed.
*   **Managed Nodes**: The servers you are configuring.
*   **The Protocol**: Ansible uses standard **SSH**.
*   **The Requirement**: The managed nodes only need **Python** installed. 

> 🔒 **Security Win**: No agents means no extra backdoors, no extra RAM usage, and no complex firewall rules. If you can SSH into it, Ansible can automate it!

### The Golden Rule: Idempotency
<details>
<summary>Click to reveal the most important concept in DevOps</summary>

**Idempotency** means you can run the exact same Ansible playbook 1 time or 1,000 times, and the end result will be exactly the same, without causing errors or making unnecessary changes. 

If you tell Ansible "Ensure Nginx is installed," it will:
1. Check if Nginx is installed.
2. If yes, do nothing (Output: `OK`).
3. If no, install it (Output: `CHANGED`).
</details>

---

## 🏗️ Part 2: The "Production-Ready" Directory Structure

Don't put all your code in one giant `playbook.yml`. Professionals use **Roles** to keep things modular and reusable. Here is the perfect directory structure:

```text
ansible-project/
├── ansible.cfg              # Ansible settings (inventory path, remote user)
├── inventory/
│   ├── production/
│   │   ├── hosts.aws_ec2.yml # AWS Dynamic Inventory
│   │   └── group_vars/
│   │       ├── all.yml      # Variables for all servers
│   │       └── web.yml      # Variables just for web servers
├── playbooks/
│   ├── site.yml             # The master playbook calling all roles
│   └── webservers.yml       # Playbook specifically for web servers
└── roles/
    ├── baseline/            # Our "Unified Configuration" role
    │   ├── tasks/
    │   │   └── main.yml     # The actual commands
    │   ├── handlers/
    │   │   └── main.yml     # Service restarts (e.g., reload SSHD)
    │   ├── templates/
    │   │   └── sshd_config.j2 # Jinja2 templates for config files
    │   └── defaults/
    │       └── main.yml     # Default variables for the role
    └── nginx/               # A role dedicated to Nginx
```

---

## ⚡ Part 3: The 10 Essential Modules Cheatsheet

Ansible has thousands of modules, but you will use these 10 for 90% of your work.

| Module | What it does | One-Line Example |
| :--- | :--- | :--- |
| `apt` (or `yum`) | Manages packages | `apt: name=htop state=present update_cache=yes` |
| `copy` | Copies static files to servers | `copy: src=files/index.html dest=/var/www/html/` |
| `template` | Copies files, but allows dynamic variables | `template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf` |
| `service` | Manages background services | `service: name=nginx state=started enabled=yes` |
| `user` | Manages Linux users and groups | `user: name=deploy group=admin shell=/bin/bash` |
| `file` | Manages file properties (permissions, symlinks) | `file: path=/opt/app state=directory mode='0755'` |
| `lineinfile` | Ensures a specific line exists in a file | `lineinfile: path=/etc/hosts line='10.0.0.5 db-server'` |
| `git` | Clones or updates Git repositories | `git: repo=https://github.com/org/repo.git dest=/opt/src` |
| `ufw` | Manages the Uncomplicated Firewall | `ufw: rule=allow port=80 proto=tcp` |
| `command` | Runs a raw command (Use sparingly!) | `command: /opt/scripts/init.sh` |

---

## ☁️ Part 4: The Cloud Superpower - AWS Dynamic Inventory

When working in AWS, IP addresses change constantly. Hardcoding IPs in a `hosts` file is an anti-pattern. Instead, we use the `aws_ec2` plugin to ask the AWS API for our servers based on their **Tags**.

> 🎯 **Tags are the Source of Truth**
> Tag your EC2 instances in AWS with `Role: WebServer` and `Environment: Production`. Ansible will automatically find them!

### How to set it up:
1. Name your inventory file ending in `aws_ec2.yml` (e.g., `inventory.aws_ec2.yml`).
2. Tell Ansible how to group your servers dynamically based on tags.

```yaml
# inventory.aws_ec2.yml
plugin: aws_ec2
regions:
  - eu-west-1
keyed_groups:
  # This creates groups like: role_webserver
  - key: tags.Role
    prefix: role
  # This creates groups like: env_production
  - key: tags.Environment
    prefix: env
```
Run `ansible-inventory -i inventory.aws_ec2.yml --graph` to see Ansible magically discover all your servers!

---

## 🛡️ Part 5: The Professional Toolchain & Safety Features

Real-world environments require safety rails and standard tooling. Before running any playbook targeting production, a Senior DevOps Engineer uses these tools:

### The Defensive Arsenal
*   `ansible-playbook --syntax-check site.yml`: Catches YAML formatting errors instantly.
*   `ansible-playbook site.yml --check`: **Dry Run mode**. Ansible pretends to run the playbook and shows what *would* change in yellow, but makes no actual modifications.
*   `ansible-playbook site.yml --diff`: Shows exact line-by-line differences before changing files (crucial when using `template` or `copy`).
*   `ansible-playbook site.yml --limit web_servers`: Restricts the playbook execution only to a subset of your inventory.

### Ansible Lint & Galaxy
*   **`ansible-lint`**: The ultimate code reviewer. Running `ansible-lint <file>` enforces community best practices, checks for deprecated modules, and ensures standard naming conventions.
*   **Ansible Galaxy**: Don't reinvent the wheel! Need Jenkins? Need PostgreSQL? Someone already wrote a perfect role for it.
    *   *Search*: `ansible-galaxy search docker`
    *   *Install*: `ansible-galaxy install geerlingguy.docker`
    *   *Use*: Just add `- role: geerlingguy.docker` to your playbook.

---

## 💻 Part 6: Hands-On Exercises (Student Version)

It's time to build! Work through these exercises.

### 🛠️ Exercise 1: Build the "Baseline" Security Role
Create a `baseline` role that applies the "Unified Configuration" to every server in our infrastructure.
**Requirements:**
- Install the following packages: `vim`, `htop`, `fail2ban`, `curl`.
- Ensure UFW (firewall) is configured to allow SSH (port 22) and is enabled.
- Create a new user called `cloud_admin` with passwordless sudo privileges.

### 🌐 Exercise 2: Load-Balanced Nginx with Jinja2
Deploy an Nginx web server using a custom configuration.
**Requirements:**
- Install Nginx and ensure it starts on boot.
- Use a `template` module to copy an Nginx config file that dynamically includes the hostname of the server in the body of the `index.html`.
- **Crucial**: If the Nginx config file changes, you must restart the Nginx service. (Hint: Look up Ansible `handlers` and `notify`).

### 🤐 Exercise 3: Secrets Management with Ansible Vault
Never commit plaintext passwords to GitHub!
**Requirements:**
- Create an encrypted file called `db_secrets.yml` using `ansible-vault create`.
- Add a variable `db_password: "SuperSecretPassword123!"` inside it.
- Write a simple playbook that uses the `debug` module to print `"The database password is {{ db_password }}"` (run it with `--ask-vault-pass`).

### 🐳 Exercise 4: Standing on the Shoulders of Giants (Ansible Galaxy)
Don't write a Docker role from scratch. Use the community's best role!
**Requirements:**
- Download the `geerlingguy.docker` role using the `ansible-galaxy` CLI.
- Write a short playbook (`install_docker.yml`) that applies this role.
- Add a task *after* the role finishes to guarantee the `docker` service is explicitly started and enabled on boot.

---

## 👨‍🏫 Part 7: Solution Key (Instructor Version)

<details>
<summary><b>Click to show solutions for Instructors</b></summary>

### Exercise 1: Baseline Role (`roles/baseline/tasks/main.yml`)
```yaml
---
# 1. Install common packages
- name: Install common unified baseline packages
  apt:
    name: 
      - vim
      - htop
      - fail2ban
      - curl
    state: present
    update_cache: yes

# 2. Configure UFW Firewall
- name: Allow SSH through UFW and enable firewall
  ufw:
    rule: allow
    port: '22'
    proto: tcp
    state: enabled

# 3. Create Admin User
- name: Create cloud_admin user
  user:
    name: cloud_admin
    groups: sudo
    shell: /bin/bash
    append: yes

- name: Setup passwordless sudo for cloud_admin
  lineinfile:
    path: /etc/sudoers.d/cloud_admin
    line: 'cloud_admin ALL=(ALL) NOPASSWD: ALL'
    create: yes
    mode: '0440'
    validate: 'visudo -cf %s'
```

### Exercise 2: Nginx with Handlers

**`roles/nginx/tasks/main.yml`:**
```yaml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present

- name: Deploy dynamic index.html template
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
  # NOTIFY triggers the handler ONLY when this file actually changes
  notify: restart nginx

- name: Ensure Nginx is enabled and running
  service:
    name: nginx
    state: started
    enabled: yes
```

**`roles/nginx/handlers/main.yml`:**
```yaml
---
- name: restart nginx
  service:
    name: nginx
    state: restarted
```

**`roles/nginx/templates/index.html.j2`:**
```html
<h1>Welcome to ESTIN Cloud Hub!</h1>
<p>You have reached the server: {{ ansible_hostname }}</p>
```

### Exercise 3: Ansible Vault

**Step 1:** Run `ansible-vault create db_secrets.yml`. Enter a password.
**Step 2:** Add code:
```yaml
db_password: "SuperSecretPassword123!"
```
**Step 3:** The Playbook (`test_secrets.yml`):
```yaml
---
- name: Test Vault Extraction
  hosts: localhost
  vars_files:
    - db_secrets.yml
  tasks:
    - name: Print the secret softly
      debug:
        msg: "The database password is {{ db_password }}"
```
**Step 4:** Run with `ansible-playbook test_secrets.yml --ask-vault-pass`

### Exercise 4: Ansible Galaxy

**Step 1:** Run `ansible-galaxy install geerlingguy.docker` on the Control Node.

**Step 2:** Create the Playbook (`install_docker.yml`):
```yaml
---
- name: Deploy Docker using a Community Role
  hosts: all
  become: yes
  roles:
    - geerlingguy.docker
  tasks:
    - name: Ensure Docker service is running and enabled
      service:
        name: docker
        state: started
        enabled: yes
```

</details>

---
> 🎓 **Instructor Note**: Remind students to heavily rely on `ansible-lint` output before submitting their exercises. It forces them into great habits instantly!
