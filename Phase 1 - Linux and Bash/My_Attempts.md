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

## Challenge 9: DNS troubleshooting

**Description**: Someone modified a critical DNS configuration file. Fix it to reveal the flag.

**Date of Attempt**: 02/03/2025

**Approach**: DNS resolution in linux is usually managed by resolved and references a file called "resolv.conf" which resides in /etc/, this would be the first thing to check for DNS issues to determine the nameservers being used and see if they are valid/reachable.

**Solution**:
```sh
cat /etc/resolv.conf # Notice the nameserver line with the CTF flag breaking the nameserver definition.
sudo nano /etc/resolv.conf #Open nano editor to repair the config (though you could use vi or other options) save the changes.
sudo systemctl restart systemd-resolved # Restart the resolved daemon to have it use the repaired configuration
```

**Notes**: Resolv.conf is an important configuration file when it comes to defining nameservers for DNS lookups, this should be the first thing to check when experiencing issues with DNS and resolving hostnames.

## Challenge 10: Remote upload

**Description**: Transfer any file to the ctf_challenges directory to trigger the flag.

**Date of Attempt**: 02/03/2025

**Approach**: Use `scp` from my own local device to securely copy a file from the local machine to the remote server, since we already have SSH access to the remote system, scp can be used to securly copy the contents of a file to the remote system from our own.

**Solution**:
```sh
scp ./HelloWorld.txt ctf_user@4.234.106.243:/home/ctf_user/ctf_challenges/ # From my own local device.
```

**Notes**: The scp command uses SSH to transfer files securely between the local machine and the remote server. Ensure you have the correct permissions to write to the target directory on the remote server and use the syntax as per the solution "scp Local_Path User@Host:RemotePath" in this challenge a script is configured to broadcast a message when a file was uploaded which presented itself to my SSH session with the flag using the "wall" command. 

## Challenge 11: Web Configuration

**Description**: The web server is running on a non-standard port. Find and fix it.

**Date of Attempt**: 02/03/2025

**Approach**: Since the readme indicates that the webserver is running via Nginx and makes reference to its configuration files, I will check the configuration file to determine which port is being used for the service, adjust it and restart the service to use the correct port.

**Solution**:
```sh
ls /etc/nginx/sites-available/ # View available sites
sudo nano /etc/nginx/sites-available/default # Edit the config file and save from port 8083 to 80
sudo systemctl restart nginx # Restart the nginx webserver so that it now runs on the correct port.
curl http://127.0.0.1 # Make a simple http request on port 80 to the localhost IP which returned the contents of the index file containing the flag!
```

**Notes**: The port being used was not the standard port used for HTTP traffic, though other ports can be used for webservers, users would have to specify the port in their browser in order to navigate to it correctly, upon finding the incorrect port in the config file we changed it back to the default for HTTP of "80" and restarted the service, we can then make a request to the system on port 80 and view the results.

## Challenge 12: Network Traffic Analysis

**Description**: Someone is sending secret messages via ping packets.

**Date of Attempt**: 02/03/2025

**Approach**: We know from the description that someone is sending ping packets to our server and that they are using ping or ICMP for this, so we need to analyze the traffic being sent by using a tool suitable for this, "TCPDump" allows us to capture packets being sent to our server and analyze them further.

**Solution**:
```sh
sudo tcpdump -i any icmp -A -c 10 # tcpdump, any interface, filter for ICMP packets, convert output to ASCII and only capture 10 packets.
```

**Notes**: At first I was struggling to find packets which made sense and no ICMP packets were being found, then I realized that I had not specified the "All" interfaces flag and tried again, this gave me some results, seeing as the flag is going to be text based, I used the "-A" flag to convert the results to display as ASCII and then found the flag being sent.

---

**Key Takeaway**:

These challenges provided an engaging and interactive way to enhance my Linux command line skills. The difficulty level is suitable for entry-level Linux users and covers a range of tools and features that a cloud engineer may encounter. A few twists and turns such as nested Base64 encoding kept things interesting and challenge a users ability to think outside the box. I look forward to continuing my learning journey and tackling more advanced challenges in the future.

> ✓ Correct flag for Challenge 12!  
Flags Found: 12/12 Congratulations!  
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

- `systemctl` - A command to examine and control the systemd system and service manager.
  **Benefits/Use Cases**: Essential for managing system services, including starting, stopping, restarting, and checking the status of services. In Challenge 9, it was used to restart the `systemd-resolved` service, it was also used in challenge 11 to restart the nginx service once the port was updated.

- `scp` - Secure copy command is used to copy files between hosts on a network.
  **Benefits/Use Cases**: Useful for securely transferring files between local and remote systems over SSH. In Challenge 10, it was used to upload a file to the remote server.

- `nano` - A simple text editor for Unix-like systems.
  **Benefits/Use Cases**: Useful for editing configuration files directly from the terminal. In Challenge 9 and 11, it was used to edit configuration files, though vi or any other editors could have been used in its place.

- `tcpdump` - A command-line packet analyzer.
  **Benefits/Use Cases**: Useful for capturing and analyzing network traffic. In Challenge 12, it was used to capture and analyze ICMP packets to find the flag, very useful when wanting a low level view of what is being sent and received across your network.

- `wall` - A command to send a message to all logged-in users.
  **Benefits/Use Cases**: Useful for broadcasting messages to all users currently logged in on the system. In Challenge 10, I believe it was used to notify users when a new file was added to the `ctf_challenges` directory based on the response that appeared after I sent a file using SCP, thus I added it here for reference.
</details>