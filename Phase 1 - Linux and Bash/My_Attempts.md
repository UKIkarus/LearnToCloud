> [!IMPORTANT] 
> By continuing beyond this point, you acknowledge and understand that this document contains spoilers which may otherwise ruin your experience.

<details>
  <summary>Click to reveal spoilers</summary>

# My Attempts at Phase 1: Linux Command Line CTF Challenge

This document details my journey through the Phase 1 Linux Command Line CTF Challenge. Each challenge has been a learning experience, and I hope my notes and solutions can help others on their path.

For these tasks, I will be using the Windows Subsystem for Linux via Visual Studio Code, remoting into the virtual machine created via the Terraform template provided, I will be hosting this on Azure vs AWS as this is the cloud provider I primarily use currently.

## Challenge 1: The Hidden File

**Description**: Find and read a hidden file in the `ctf_challenges` directory.

**Date of Attempt**: 11/02/2025

**Approach**: I used the `ls -a` command to list all files, including hidden ones, in the `ctf_challenges` directory.

**Solution**:
```sh
cd ctf_challenges
ls -a
cat .hidden_flag
```

**Notes**: Hidden files in Linux start with a dot (.) which is why they are not visible with a regular `ls` command.

## Challenge 2: The Secret File

**Description**: Locate a file containing "secret" in its name within /home/ctf_user.

**Date of Attempt**: 11/02/2025

**Approach**: I utilized the `find` command to search for files with "secret" in their names.

**Solution**:
```sh
find /home/ctf_user -name "*secret*"
cat ./documents/projects/backup/secret_notes.txt
```

**Notes**: The `find` command is powerful for searching files based on various criteria, you can use it to locate files with specific permissions, specific owners, specific filenames, it even allows users to execute commands against each returned result, mastering the `find` command will make navigating Linux environments much easier.

## Challenge 3: The Largest Log

**Description**: Identify and read the largest file in /var/log.

**Date of Attempt**: 11/02/2025

**Approach**: I used the `ls -S` command to sort files by size and then identified the largest file.

**Solution**:
```sh
cd /var/log
ls -S
tail -n 5 large_log_file.log # Displays the last 5 lines of the file
```

**Notes**: Sorting files by size helps quickly identify the largest files, using tail displays the end of the file.

## Challenge 4: The User Detective

**Description**: Find a flag in the .profile of the user with UID 1002.

**Date of Attempt**: 11/02/2025

**Approach**: I checked the `/etc/passwd` file to find the user with UID 1002 and then read their .profile file.

**Solution**:
```sh
grep 1002 /etc/passwd
sudo cat /home/flag_user/.profile # ctf_user is a sudo user!
```

**Notes**: The `/etc/passwd` file contains user information, including UIDs, this allowed me to determine the user tied to the UID and navigate to their home directory.

## Challenge 5: The Permissive File

**Description**: Find a root-owned file with 777 permissions. The flag is the contents of this file.

**Date of Attempt**: 11/02/2025

**Approach**: I used the `find` command to search for files with 777 permissions owned by root.

**Solution**:
```sh
find / -perm 777 -user root -type f -print 2>/dev/null
cat /opt/systems/config/system.conf
```

**Notes**: Understanding file permissions is crucial for this challenge as is redirecting console output, initially I ran the find command without the "2>/dev/null", this worked but showed many permission denied errors. I only wanted to see the results and exclude errors, by setting the type to "f" for file and redirecting errors to "/dev/null" I was presented with a single result.

## Challenge 6: The Hidden Service

**Description**: Identify a process on port 8080 and retrieve its flag.

**Date of Attempt**: 11/02/2025

**Approach**: I used `netstat`, `lsof` and `curl` commands to identify the process running on port 8080.

**Solution**:
```sh
netstat -tuln | grep 8080
sudo lsof -i :8080
curl 127.0.0.1:8080
```

**Notes**: When using netstat I found a process listening on port 8080, by using lsof -i, I can list the process/command and PID to being "NC" when using "man nc" I noted that "nc" is short for "Netcat", essentially sending output over the network, since we know the listening port is 8080 and the type is http, I simply made a web-request to this port on the loopback address!

## Challenge 7: The Encoded Secret

**Description**: Decode a base64-encoded flag.

**Date of Attempt**: 11/02/2025

**Approach**: I used the `base64` command to decode the flag.

**Solution**:
```sh
cat /home/ctf_user/ctf_challenges/encoded_flag.txt | base64 --d | base64 -d
```

**Notes**: Base64 encoding is commonly used for data encoding, in this example I decoded the initial contents of the file only to find that the contents were not as expected and seemed to still be BASE64? at first I was puzzled, thinking I had entered the command incorrectly so I created a file and encoded some text and decoded, this worked, noticing a pattern on the output I piped the results into decode again and discovered that the original contents were encoded twice!

## Challenge 8: SSH Key Authentication

**Description**: Configure SSH key authentication and find a hidden flag.

**Date of Attempt**: 11/02/2025

**Approach**: Use ssh-keygen to create an RSA key, copy this to the server, login with the thumbprint, explore the profile.

**Solution**:
```sh
ssh-keygen -t rsa
ssh-copy-id user@hostname
ssh user@hostname
cat /home/ctf_user/.ssh/secrets/backup/.authorized_keys
```

**Notes**: It may not have been necessary to create the RSA key and login via this method, however, I wanted to show I knew how to create and upload the key, after logging in, I checked the users ".ssh" folder and found that there was a "Secrets" and "Backup" folder which seemed unusual since "authorized_keys" and "known_hosts" usually reside here, browsing through these folders and displaying the contents of ".authorized_keys" showed the final flag.

---

**Key Takeaway**:

These challenges provided an engaging and interactive way to enhance my Linux command line skills. The difficulty level is suitable for entry-level Linux users and covers a range of tools and features that a cloud engineer may encounter. A few twists and turns such as nested Base64 encoding kept things interesting and challenge a users ability to think outside the box. I look forward to continuing my learning journey and tackling more advanced challenges in the future.

> ✓ Correct flag for Challenge 8!  
Flags Found: 8/8 Congratulations!  
You've completed all challenges!

**Tools and commands used**

- `cd` - Change directory is a command used to switch the current working directory to another within the terminal.  
  **Benefits/Use Cases**: Essential for navigating the file system, allowing users to move between directories to access files and execute commands in different locations.

- `ls` - Short for "List" is used to list the contents of a directory, including files and sub-directories.  
  **Benefits/Use Cases**: Useful for viewing the contents of a directory, checking for the presence of files, and understanding the structure of the file system. Parameters like `-a` (show all files, including hidden ones) enhance its functionality further.

- `find` - A command allowing users to search for files or folders on the device.
  **Benefits/Use Cases**: Extremely versatile for locating files and folders based on various criteria like name, size, modification date, owner, and permissions. It can also be used to automatically perform further actions on the found files.  

- `grep` - Global Regular Expression Print is used to search for specific patterns within files and allows for utilising regex.  
  **Benefits/Use Cases**: Powerful for searching text within files, filtering output, and finding specific information. It supports regular expressions for advanced search patterns. In our example we used "grep" to search for the term "1002" to identify which user had this UID on the system.

- `tail` - The tail command is used to read and return a specific number of lines from the end of a file.  
  **Benefits/Use Cases**: Since files come in varying sizes, the tail command is absolutely essential for quickly viewing the last few lines of a file, an example in this case would be a large log file which could otherwise flood the screen with information, fill the output buffer or cause the machine to otherwise hang if trying to print the entirety to the terminal.

- `cat` - The concatenate command is used to read, display and combine the content of files.  
  **Benefits/Use Cases**: Useful for quickly viewing the contents of a file without opening it in an editor. It can also concatenate multiple files and display their combined contents, often used alongside pipes to pass the contents of a file into another command/utility for further processing/manipulation.

- `base64` - A command built into the linux commandline to encode and decode data using Base64 encoding.  
  **Benefits/Use Cases**: Useful for encoding binary data into text format for safe transmission or storage. It can also decode Base64-encoded data back to its original form.

- `netstat` - Network statistics command is used to display network connections, routing tables, and interface statistics.  
  **Benefits/Use Cases**: Useful for monitoring network connections, diagnosing network issues, and understanding network activity on a system. A network administrator might use `netstat` to check for open ports and active connections on a server to ensure there are no unauthorized access points or services listening for connections.

- `curl` - Command-line tool for transferring data with URLs.  
  **Benefits/Use Cases**: Versatile for making HTTP requests, downloading files, and interacting with web APIs. It supports various protocols and options for customizing requests, in this example we used it to make a request to the service running on port 8080.

- `lsof` - List open files command is used to display information about files opened by processes.  
  **Benefits/Use Cases**: Useful for identifying which processes are using specific files or network ports, helping diagnose issues related to file usage or network connections, in this example we used it to determine what process was listening on port 8080.

- `ssh-keygen` - Generate an SSH key pair.  
  **Benefits/Use Cases**: Essential for creating SSH keys for secure authentication to remote servers also allowing for passwordless authentication.

- `ssh-copy-id` - Install your public key in a remote machine's authorized_keys.  
  **Benefits/Use Cases**: Simplifies the process of copying your public key to a remote server for SSH key-based authentication, this will require authentication initially, but will allow for password-less access to the device using RSA keys for future connections.

- `ssh` - Secure Shell is used to securely connect to a remote machine over a network.  
  **Benefits/Use Cases**: Essential for remote administration, allowing users to execute commands on remote servers securely. It supports key-based authentication for enhanced security, this was the primary means of connecting to our remote CTF virtual machine for this challenge, all commands were run from the VM itself via a remote session using SSH.

- `sudo` - Superuser do is a command that allows a permitted user to execute a command as the superuser or another user, as specified by the security policy.  
  **Benefits/Use Cases**: Essential for performing administrative tasks that require elevated privileges. It ensures that only authorized users can execute commands that could affect the system's configuration or security. In this challenge we used "sudo" in order to access another users home-directory and read the ".profile" contents.
  
</details>