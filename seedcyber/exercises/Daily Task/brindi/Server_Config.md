
# Kamatera VPS User Management and SSH Key Setup

## Objective
Guide interns in setting up their own user accounts and SSH keys for secure access to the Kamatera VPS.

---

## Step 1: Generate SSH Keys
1. **Open Terminal** (Linux/Mac) or **Command Prompt** (Windows with Git Bash).
2. Run:
	```bash
	ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
	```
3. Follow the prompts:
	- Press Enter to accept the default location.
	- Optionally, set a passphrase for added security.
4. **Key Locations:**
	- Private Key: `~/.ssh/id_rsa` (**Keep this secret!**)
	- Public Key: `~/.ssh/id_rsa.pub` (Share this with the server)

---

## Step 2: Add SSH Key to Kamatera
### During Server Creation
- In the Kamatera control panel, locate the SSH Key field.
- Paste the contents of your public key (`id_rsa.pub`).

### For Existing Servers
1. Log into the Kamatera Console as **root**.
2. Create the `.ssh` directory and set permissions:
	```bash
	mkdir -p ~/.ssh
	chmod 700 ~/.ssh
	```
3. Add your public key:
	- Open the file:
	  ```bash
	  nano ~/.ssh/authorized_keys
	  ```
	- Paste your public key and save (Ctrl+O, Enter, Ctrl+X).
4. Set permissions:
	```bash
	chmod 600 ~/.ssh/authorized_keys
	```

---

## Step 3: Log Into Your VPS
1. Open your local terminal.
2. Connect via SSH:
	```bash
	ssh root@YOUR_SERVER_IP
	```
	- Replace `YOUR_SERVER_IP` with the actual IP address.
	- Type "yes" if prompted about host authenticity on your first login.

---

## Step 4: Create a New User Account
1. Create a new user:
	```bash
	sudo adduser intern_name
	```
2. (Optional) Add user to the sudo group:
	```bash
	sudo usermod -aG sudo intern_name
	```

---

## Step 5: Configure SSH Access for New User
1. Switch to the new user:
	```bash
	sudo su - intern_name
	```
2. Set up SSH directory and permissions:
	```bash
	mkdir -p ~/.ssh
	chmod 700 ~/.ssh
	```
3. Add public key:
	- Open the file:
	  ```bash
	  nano ~/.ssh/authorized_keys
	  ```
	- Paste the intern's public key and save.
4. Set correct permissions:
	```bash
	chmod 600 ~/.ssh/authorized_keys
	exit
	```
5. **Permissions Summary:**
	- `700` (.ssh folder): Only the user can read/write/execute.
	- `600` (authorized_keys): Only the user can read/write.

---

## Step 6: Basic Security & Firewall (UFW)
1. Enable the firewall:
	```bash
	sudo ufw allow OpenSSH
	sudo ufw enable
	```
2. (Recommended) Disable root login:
	- Edit SSH config:
	  ```bash
	  sudo nano /etc/ssh/sshd_config
	  ```
	- Change `PermitRootLogin yes` to `PermitRootLogin no`.
	- Restart SSH:
	  ```bash
	  sudo systemctl restart ssh
	  ```

---

## Step 7: Practical Daily Tasks for Interns
- **Remote Access:** Log in using `ssh username@IP`.
- **Filesystem Practice:**
  - `ls` — List files
  - `cd` — Change directory
  - `mkdir` — Create directory
- **Security Check:** If you disabled root login, verify that you can no longer log in as root.
