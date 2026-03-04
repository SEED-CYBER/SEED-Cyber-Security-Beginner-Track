# Intern Linux Practice Tasks

These tasks are for interns who have user accounts on the Kamatera VPS. None of the interns have root or sudo access. All tasks should be performed from your own user account after logging in via SSH.

---

## 1. Remote Login
- Log in to the server using your assigned username and SSH key:
  ```bash
  ssh your_username@YOUR_SERVER_IP
  ```
- Confirm you are in your home directory by running:
  ```bash
  pwd
  ```

---

## 2. Filesystem Navigation
- List the contents of your home directory:
  ```bash
  ls
  ```
- Create a new directory called `practice`:
  ```bash
  mkdir practice
  ```
- Change into the `practice` directory:
  ```bash
  cd practice
  ```
- Create a nested directory structure:
  ```bash
  mkdir -p projects/scripts
  ```
- Move back to your home directory:
  ```bash
  cd ~
  ```

---

## 3. File Management
- Inside the `practice` directory, create a file called `notes.txt`:
  ```bash
  cd ~/practice
  touch notes.txt
  ```
- Edit `notes.txt` using `nano` or `vim` (choose one):
  ```bash
  nano notes.txt
  # or
  vim notes.txt
  ```
- Add at least three lines describing what you learned today.
- Save and exit the editor.
- Copy `notes.txt` to the `projects` directory:
  ```bash
  cp notes.txt projects/
  ```
- Rename `notes.txt` to `my_notes.txt`:
  ```bash
  mv notes.txt my_notes.txt
  ```
- Delete `my_notes.txt`:
  ```bash
  rm my_notes.txt
  ```

---

## 4. Basic Linux Commands
- Display the current date and time:
  ```bash
  date
  ```
- Show your current username:
  ```bash
  whoami
  ```
- Display disk usage for your home directory:
  ```bash
  du -sh ~
  ```
- Show the last 5 commands you ran:
  ```bash
  history | tail -5
  ```
- Find all `.txt` files in your home directory:
  ```bash
  find ~ -name "*.txt"
  ```
- Display the contents of `projects/notes.txt`:
  ```bash
  cat projects/notes.txt
  ```

---

## 5. Extra Practice
- Create a file called `linux_journey.txt` and write a short paragraph about your experience so far.
- List all files and directories in your home directory, including hidden ones:
  ```bash
  ls -la ~
  ```
- Try using the `man` command to read the manual for `ls`:
  ```bash
  man ls
  ```
- Log out of the server:
  ```bash
  exit
  ```

---

> Complete these tasks and be ready to discuss what you learned in the next session.
