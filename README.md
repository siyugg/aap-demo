# 📘 AAP Demo – Ansible Playbooks for RHEL VM

This repository provides Ansible playbooks to:  

- Update and manage packages on RHEL VMs  
- Manage firewall rules  
- Configure LVM/RAID10 for data disks  
- Send email notifications after key operations  

---

## ⚙️ Prerequisites

Before running the playbooks:

- **RHEL VM (v9 recommended)**  
  > RHEL10 does not support multiple versions of some packages (e.g. `httpd`).  

- **Ansible installed** (≥ 2.13)  

- **AWS CLI + credentials** if attaching EBS volumes  

- **Python packages for AWS integration**
```console
pip install boto3 botocore
```
- Required collections/roles:
```console
ansible-galaxy collection install amazon.aws
sudo dnf install rhel-system-roles -y
```
---

## 🚀 Usage Guide

Clone the repository:
```console
git clone https://github.com/siyugg/aap-demo.git
cd aap-demo
```
---

### 1. 🔄 Upgrade All System Packages
Upgrades system packages to the latest available version.  
```console
ansible-playbook playbooks/upgrade.yml -i inventory/hosts
```
- If package is already latest → skip  
- If update is applied → notify user via email  

Manually downgrade/revert versions (example):
```console
sudo dnf list --showduplicates httpd
sudo dnf remove httpd -y
sudo dnf install httpd-2.4.3-7.el9
```
## 📧 Send Email Notifications
Email is sent after package updates.  
```console
ansible-playbook playbooks/notify.yml -i inventory/hosts
```
Sample configuration:
```console
name: Send update notification
mail:
host: smtp.example.com
port: 25
from: ansible@demo.org
to: user@example.com
subject: "System Update Completed"
body: "Packages and system configuration have been updated successfully."
```
---

### 2. 🔥 Configure Firewall
Allows required ports such as HTTP/HTTPS.  
```console
ansible-playbook playbooks/firewalld.yml -i inventory/hosts
```
Example of what’s applied:
```console
name: Allow HTTP and HTTPS
firewalld:
service: "{{ item }}"
permanent: yes
state: enabled
with_items:
    - http
    - https
```
---

### 3. 💾 Configure LVM/RAID10 for Data Disk
Checks for available disks, partitions them, and configures RAID10 via LVM.  
```console
ansible-playbook playbooks/storage.yml -i inventory/hosts
```
- If disks already exist → skip configuration  
- If new disks are detected → partition and configure LVM/RAID10  
- Supports AWS EBS volumes via `amazon.aws.ec2_vol`  

---

## 🛑 Known Issues
- **RHEL10**: Package versions (e.g. `httpd`) may not support downgrading.  
  - ✅ Solution: Use **RHEL9** for multi-version package control.  

---

## ✅ Roadmap / To Do
- Enhance rollback support for package management  
- Add centralized logging for playbook runs  
- Expand for cloud-init volume provisioning  