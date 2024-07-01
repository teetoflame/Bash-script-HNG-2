---

# User Management Automation Script

## Overview

The `create_users.sh` script is designed to automate the creation and management of user accounts on a Linux system. By reading a specified input file containing usernames and associated groups, the script facilitates streamlined user provisioning with a focus on security and efficiency. This tool is particularly useful for system administrators and DevOps engineers tasked with managing user accounts in dynamic and large-scale environments.

### Features

- **Automated User Creation**: Creates users and their primary groups based on input from a text file.
- **Random Password Generation**: Generates secure, random passwords for each user.
- **Group Management**: Adds users to specified groups, creating groups if they don't exist.
- **Detailed Logging**: Logs all actions to `/var/log/user_management.log` for audit and troubleshooting.
- **Secure Password Storage**: Stores generated passwords securely in `/var/secure/user_passwords.csv` with restricted access.

## Requirements

- Linux environment (tested on Ubuntu)
- Bash shell
- OpenSSL for password generation
- Root or sudo privileges to execute user and group management commands

## Usage

### 1. Clone the Repository

Clone the repository to your local machine or directly onto your Linux server:

```bash
git clone https://github.com/yourusername/user-management-script.git
cd user-management-script
```

### 2. Prepare the Input File

Create a text file with user and group information in the format:

```plaintext
username;group1,group2,group3
```

Example:

```plaintext
light;sudo,dev,www-data
idimma;sudo
mayowa;dev,www-data
```

### 3. Run the Script

Execute the script by providing the input file as an argument. Make sure to run it with sufficient privileges (e.g., as root or with `sudo`):

```bash
sudo ./create_users.sh <input_file>
```

Replace `<input_file>` with the path to your input file.

### 4. Verify the Output

Upon completion, the script logs all actions to `/var/log/user_management.log` and stores generated passwords in `/var/secure/user_passwords.csv`. You can review these files to verify the script's execution:

```bash
cat /var/log/user_management.log
sudo cat /var/secure/user_passwords.csv
```

### 5. Check Created Users and Groups

To ensure users and groups have been created correctly, you can use the following commands:

- List users: `cut -d: -f1 /etc/passwd`
- List groups: `cut -d: -f1 /etc/group`

## Script Details

### `create_users.sh`

```bash
#!/bin/bash

# Check if input file is provided
if [ -z "$1" ]; then
  echo "Usage: $0 <input_file>"
  exit 1
fi

INPUT_FILE="$1"
LOG_FILE="/var/log/user_management.log"
PASSWORD_FILE="/var/secure/user_passwords.csv"

# Create log file and password file directories if they don't exist
mkdir -p "$(dirname "$LOG_FILE")"
mkdir -p "$(dirname "$PASSWORD_FILE")"

# Ensure password file permissions are secure
touch "$PASSWORD_FILE"
chmod 600 "$PASSWORD_FILE"

# Function to generate random password
generate_password() {
  openssl rand -base64 12
}

# Function to log messages
log_message() {
  echo "$(date +'%Y-%m-%d %H:%M:%S') - $1" >> "$LOG_FILE"
}

# Read and process the input file
while IFS=';' read -r username groups; do
  # Trim whitespace
  username=$(echo "$username" | xargs)
  groups=$(echo "$groups" | xargs)

  # Create user and primary group
  if id "$username" &>/dev/null; then
    log_message "User $username already exists"
  else
    useradd -m -G "$username" -s /bin/bash "$username"
    log_message "User $username created"

    # Generate and set password
    password=$(generate_password)
    echo "$username:$password" | chpasswd
    echo "$username,$password" >> "$PASSWORD_FILE"
    log_message "Password for $username set"

    # Create additional groups
    IFS=',' read -r -a group_array <<< "$groups"
    for group in "${group_array[@]}"; do
      group=$(echo "$group" | xargs)
      if ! getent group "$group" &>/dev/null; then
        groupadd "$group"
        log_message "Group $group created"
      fi
      usermod -aG "$group" "$username"
      log_message "User $username added to group $group"
    done
  fi
done < "$INPUT_FILE"

log_message "User creation process completed"

echo "Script execution completed. Check $LOG_FILE for details."
```

## Contributing

Contributions are welcome! Feel free to fork the repository, make improvements, and submit a pull request.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---
