# Oracle Database 21c Installation with Ansible

This project automates the installation and configuration of **Oracle Database 21c** using **Ansible**. It also includes steps to set up **passwordless SSH** for Ansible on **Debian-based systems**.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Setting Up Passwordless SSH for Ansible](#setting-up-passwordless-ssh-for-ansible)
3. [Oracle Database 21c Installation](#oracle-database-21c-installation)
4. [Playbook Tasks Breakdown](#playbook-tasks-breakdown)
5. [Running the Playbooks](#running-the-playbooks)
6. [Post-Installation Steps](#post-installation-steps)

---

## Prerequisites

Before proceeding, ensure the following:

- The **target server** is running **Oracle Linux 8**.
- **Java JDK 17** is required for Oracle Database 21c installation.
- **Oracle Database 21c** installation files.
- **Ansible** installed on the control node.
- **SSH access** to the target server.
- Both **Java JDK 17** and **Oracle Database 21c** installation files must be **downloaded manually** and placed in the **files/** directory before running the playbooks.

---

## Setting Up Passwordless SSH for Ansible

### Step 1: Update the System and Install OpenSSH Server
Run the following commands on the target server to ensure SSH is installed and running:

```bash
sudo apt update -y
sudo apt install openssh-server -y
```

### Step 2: Enable and Start the SSH Service
Enable and start the SSH service on the target server:

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

### Step 3: Generate an SSH Key Pair for Ansible
On the Ansible control node, generate an SSH key pair:

```bash
ssh-keygen -t rsa -b 4096 -C "ansible"
```
Press Enter to accept the default file location (~/.ssh/id_rsa). Optionally, set a passphrase for added security.

### Step 4: Verify the Key Generation

```bash
ls -la ~/.ssh
```
You should see id_rsa (private key) and id_rsa.pub (public key).

### Step 5: Copy the Public Key to the Target Server

Copy the public key to the target server (replace user and 192.168.56.101 with your username and server IP):

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub user@192.168.56.111
```

### Step 6: Test Passwordless SSH Connection
Verify that you can connect to the target server without a password:

```bash
ssh -i ~/.ssh/id_rsa user@192.168.56.111
```
If successful, you will be logged into the target server without being prompted for a password.

## Oracle Database 21c Installation

### Server Preparation
The server.yml playbook prepares the target server for the Oracle Database 21c installation by performing the following tasks:

1. Create necessary directories for Oracle installation.
2. Check if Oracle ZIP files are available on the target server and transfer them if needed.
3. Install any prerequisites for Oracle Database 21c.

### Installing and Starting the Database
The install.yml playbook handles the installation of Oracle Database 21c and the configuration of the database, including:

1. Extracting Oracle DB installation ZIP files.
2. Configuring silent installation using the Oracle response files.
3. Running the Oracle installer in silent mode.
4. Running post-installation root scripts.
5. Setting Oracle environment variables.
6. Creating the Oracle Database using the DBCA utility.
7. Starting the Oracle Database instance.

### Playbook Tasks Breakdown
The playbooks perform the following tasks:

1. Create necessary directories: Prepares the filesystem for Oracle installation.
2. Check if Oracle ZIP file exists: Verifies if the installation ZIP file is present.
3. Copy Oracle ZIP file to server: Transfers the ZIP file if it’s missing.
4. Extract Oracle DB installation zip file: Extracts the Oracle binaries.
5. Copy Oracle response file to server: Configures silent installation parameters.
6. Run Oracle installer in silent mode: Executes the Oracle installer.
7. Run root scripts: Executes mandatory post-installation scripts.
8. Set Oracle environment variables: Configures environment variables for the Oracle user.
9. Create Oracle Database using DBCA: Creates the database instance.
10. Show installation output: Displays the installation logs for verification.

## Running the Playbooks
To run the playbooks, follow these steps:

### Step 1: Clone the Repository
Clone this repository to your Ansible control node:

```bash
git clone https://github.com/MohamedLong/ODB_21_install_with_ansible.git
cd ODB_21_install_with_ansible
```
### Step 2: Update the Inventory File
Edit the inventory file to include the IP address or hostname of your target server:

```bash
[oracle_db]
192.168.56.101
```
### Step 3: Prepare the Server
Run the server.yml playbook to prepare the target server for Oracle Database installation:

```bash
ansible-playbook -i inventory server.yml
```
### Step 4: Install and Start the Database

Run the install.yml playbook to install and start Oracle Database 21c:

```bash
ansible-playbook -i inventory install.yml
```

## Post-Installation Steps

After the playbooks complete:

### Step 1: Verify the Oracle Database Installation
Run the verify.yml playbook to check if the Oracle Database installation was successful and is operating correctly. This playbook performs several checks including:

- Verifying the Oracle Database listener status.
- Ensuring the Oracle Database instance is running and accessible.

Run the following command:

```bash
ansible-playbook -i inventory verify.yml
```
### Step 2: Verify Database Connection and Status
You can also manually verify the installation by connecting to the database:

```bash
sqlplus / as sysdba
```
Once connected, check the database status:


```bash
SELECT status FROM v$instance;
```


### 🔥 Bonus: Wiping the Server for Reinstallations

During my testing, I needed a way to **reset** the server again and again without manually deleting files.  
So, I created **wipe.yml** – a playbook that **erases everything** so you can start fresh!  

Because **we are automation magicians 🪄, and we hate manual work!** 🚀 
