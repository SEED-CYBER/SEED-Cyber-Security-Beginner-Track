# Exercise 1: Linux Basics – Setup, Navigation, Directories, Files, and Cleanup

## Setup and Navigation
- Open your terminal using Ctrl + Alt + T (standard for many Linux distributions).
- Confirm your location: Type `pwd` to print your current working directory.
- List current items: Use `ls` to see what is in your home folder.
- List hidden items: Run `ls -a` to view hidden files (those starting with a dot).

## Working with Directories (Folders)
- Create a folder: Run `mkdir LinuxPractice` to create a new directory.
- Enter the folder: Use `cd LinuxPractice` to move into it.
- Create nested folders: Use `mkdir -p projects/notes` to create multiple levels at once.
- Go back: Type `cd ..` to move up one level to your home directory.
- Rename a folder: Use `mv LinuxPractice MyLab`.

## File Operations
- Create an empty file: Enter your MyLab folder and type `touch draft.txt`.
- Add content: Run `nano draft.txt`, type "Hello Linux", and save with Ctrl + O, then exit with Ctrl + X.
- Rename the file: Use `mv draft.txt final_note.txt`.
- Move the file: Run `mv final_note.txt projects/` to move it into the projects subfolder.
- Copy the file: Use `cp projects/final_note.txt projects/backup_note.txt` to create a copy.

## Deletion & Cleanup
- Delete a file: Use `rm projects/backup_note.txt`.
- Delete an empty folder: Use `rmdir projects/notes`.
- Delete a non-empty folder: Be careful and use `rm -r projects` to remove the folder and everything inside it.
