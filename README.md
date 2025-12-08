# Configuration Management with Ansible

This project is a solution for the **[Configuration Management](https://roadmap.sh/projects/configuration-management)** project from [roadmap.sh](https://roadmap.sh). It demonstrates how to automate the configuration of a Linux server using **Ansible**, specifically tailored to a local development environment using **WSL (Windows Subsystem for Linux)**.

## 📖 Project Overview

The goal of this project is to use Ansible to provision a "Worker Node" with the following configurations:
1.  **Base Setup**: System updates, essential utilities (like `fail2ban`), and creating a dedicated user.
2.  **Nginx**: Installing and configuring the Nginx web server.
3.  **App Deployment**: Deploying a static HTML website (via tarball or git).
4.  **SSH Hardening**: Managing SSH keys and security.

## 🏗 Architecture

This project was built and tested using **two distinct WSL 2 distributions** running on the same Windows machine:

* **Control Node (Ansible Host)**: `Ubuntu` (WSL)
    * This node runs the Ansible playbooks.
* **Managed Node (Worker)**: `Debian` (WSL)
    * This is the target server that gets configured.

## 📂 Project Structure

```bash
.
├── inventory.ini       # Defines the target hosts (The Debian WSL instance)
├── setup.yml           # Main playbook to run all roles
├── stop.yml            # Utility playbook to stop services (optional)
└── roles/
    ├── base/           # Updates apt cache, installs git, curl, fail2ban
    ├── nginx/          # Installs nginx, configures firewall/ports
    ├── app/            # Deploys the static website code
    └── ssh/            # Configures SSH keys and permissions
```

## 🚀 Prerequisites

Before running the playbooks, ensure you have the following set up in your WSL environments:

### 1. Configure the Worker Node (Debian)
Ensure the Debian instance has an SSH server running and Python installed (required for Ansible).

```bash
# On Debian (Worker)
sudo apt update
sudo apt install openssh-server python3 -y
sudo service ssh start
```
*Note: You may need to modify `/etc/ssh/sshd_config` to allow password authentication temporarily to copy your keys, or manually add your key.*

### 2. Configure the Control Node (Ubuntu)
Install Ansible and generate an SSH key pair.

```bash
# On Ubuntu (Control Node)
sudo apt update
sudo apt install ansible -y

# Generate SSH Key (if you haven't already)
ssh-keygen -t rsa -b 4096
```

### 3. Establish Connectivity
Copy the SSH key from Ubuntu to Debian to allow passwordless configuration.

```bash
# On Ubuntu (Control Node)
ssh-copy-id user@<DEBIAN_WSL_IP_ADDRESS>
```

## ⚙️ Configuration

### 1. Update Inventory
Edit the `inventory.ini` file to match the IP address of your Debian WSL instance. You can find the IP inside Debian by running `ip addr`.

```ini
[webservers]
172.x.x.x ansible_user=your_debian_user ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### 2. Check Connection
Verify that Ansible can talk to the worker node:

```bash
ansible -i inventory.ini webservers -m ping
```
*Success Output:*
```json
172.x.x.x | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## 🏃‍♂️ Usage

To run the full configuration and deploy the application:

```bash
ansible-playbook -i inventory.ini setup.yml
```

### Running Specific Roles
You can run specific parts of the configuration using tags:

```bash
# Only run the Nginx configuration
ansible-playbook -i inventory.ini setup.yml --tags "nginx"

# Only deploy the App
ansible-playbook -i inventory.ini setup.yml --tags "app"
```

## 🛠 Troubleshooting WSL Issues

* **SSH Service Stops:** WSL doesn't use `systemd` by default in older versions. If you get a connection refused error, ensure SSH is running on Debian:
    `sudo service ssh start`
* **Dynamic IPs:** WSL instances often change IPs on restart. Always check `ip addr` on Debian and update `inventory.ini` before running playbooks.

## 📜 Credits

Based on the [Configuration Management](https://roadmap.sh/projects/configuration-management) project by [roadmap.sh](https://roadmap.sh).
