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

### 1. 🔄 Package Management & Notifications
The 1_update_packages_notify/ folder contains several playbooks to manage package versions on RHEL VMs.
📦 Available Playbooks
- **1_display_version.yml**
    - Displays the currently installed version(s) of the defined packages.
- **1_update_and_notify.yml**
    - Updates packages to the versions specified in 1_var_packages_new.yml and sends an email notification of any changes made.
- **1_package_version_revert.yml**
    - Reverts selected packages to older versions defined in 1_var_packages_old.yml.
- **1_update_to_specific_version.yml**
    - Updates packages to specific versions defined in 1_var_packages_new.yml (useful for version pinning or rolling forward selectively).

## ⚙️ Notes
- If package is already at the specified version → no changes applied.
- If updated or reverted → notification email is sent automatically.
- Version details are managed through variable files:
    - **1_var_packages_new.yml** → defines package versions to upgrade to
    - **1_var_packages_old.yml** → defines package versions to downgrade/revert to

Manually downgrade/revert versions (example):
```console
sudo dnf list --showduplicates httpd
sudo dnf remove httpd -y
sudo dnf install httpd-2.4.3-7.el9
```
## 📧 Email Notifications
Email is sent after package updates.  

Sample configuration:
<img width="652.5" height="274.5" alt="Screenshot 2025-08-19 at 2 39 25 PM" src="https://github.com/user-attachments/assets/d26b1908-fe8d-4675-99ba-0f609ed380a6" />

---

### 2. 🔥 Configure Firewall
The 2_configure_firewall_ports/ folder contains playbooks to enable/disable ports, manage firewall rules, and send configuration reports via email.

📦 Available Playbooks
- **2_define_ports.yml**
    - Defines which ports are to be enabled or disabled.
- **2_open_firewall_ports.yml**
    - Enables ports from 2_define_ports.yml (demo_ports_enable) and sends a configuration report to the admin email.

    # Actions performed:
        - Installs firewalld and required Python libraries
        - Enables defined ports in the default zone
        - Gathers firewall active zones and open ports
        - Builds a firewall configuration report
        - Sends an email notification (SMTP configuration required in playbook)

- **2_disable_firewall.yml**
    - Disables ports listed in demo_ports_disable inside 2_define_ports.yml (e.g. 8081/tcp).

    # Actions performed:
        - Ensures firewalld is installed and running
        - Disables the targeted ports from the firewall
        - Gathers firewall status and open ports by zone
        - Builds a firewall configuration report (timestamped, host-specific)
        - Sends results via email only if changes were made

---

### 3. 💾 Configure LVM/RAID10 for Data Disk
Checks for available disks, partitions them, and configures RAID10 via LVM.  

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
