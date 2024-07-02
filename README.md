Introduction

User management in a Linux environment involves creating user accounts, setting passwords, and assigning groups. Automating these tasks can save system administrators time and reduce the likelihood of human error. This article explains my Bash script that automates user creation, password assignment, and group management based on an input file.

I'm excited to learn more about DevOps and showcase my engineering skills. The HNG Internship (https://hng.tech/internship) seems like a perfect fit for this. Hands-on experience with real-world challenges allows you to develop practical solutions and build strong skills in system administration and automation. This script is a demonstration of how automation can streamline processes in system administration.

Script Overview

The provided Bash script automates the following tasks:

Checks if an input file is provided.

Creates necessary directories for logging and password storage.

Generates random passwords.

Logs messages for auditing.

Reads and processes the input file to create users and groups.

Here's a detailed breakdown of the script:

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

Detailed Explanation

Input File Check

The script starts by checking if an input file is provided as an argument. If not, it displays usage instructions and exits.

if [ -z "$1" ]; then
  echo "Usage: $0 <input_file>"
  exit 1
fi

Variable Definitions

The script defines variables for the input file, log file, and password file paths.

INPUT_FILE="$1"
LOG_FILE="/var/log/user_management.log"
PASSWORD_FILE="/var/secure/user_passwords.csv"

Directory and File Setup

It ensures the directories for the log file and password file exist and sets secure permissions for the password file.

mkdir -p "$(dirname "$LOG_FILE")"
mkdir -p "$(dirname "$PASSWORD_FILE")"
touch "$PASSWORD_FILE"
chmod 600 "$PASSWORD_FILE"

Password Generation Function

A function to generate random passwords using openssl is defined.

generate_password() {
  openssl rand -base64 12
}

Logging Function

A function to log messages with timestamps is defined.

log_message() {
  echo "$(date +'%Y-%m-%d %H:%M:%S') - $1" >> "$LOG_FILE"
}

User and Group Management

The script reads the input file line by line, processes each user, and performs the following steps:

Trims whitespace from the username and groups.

Checks if the user already exists.

Creates the user and primary group if the user does not exist.

Generates and sets a password for the user.

Creates additional groups and adds the user to these groups.

while IFS=';' read -r username groups; do
  username=$(echo "$username" | xargs)
  groups=$(echo "$groups" | xargs)

  if id "$username" &>/dev/null; then
    log_message "User $username already exists"
  else
    useradd -m -G "$username" -s /bin/bash "$username"
    log_message "User $username created"

    password=$(generate_password)
    echo "$username:$password" | chpasswd
    echo "$username,$password" >> "$PASSWORD_FILE"
    log_message "Password for $username set"

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

Conclusion

The script automates user and group management, making it easier to manage users in a Linux environment. By using this script, administrators can ensure consistent and secure user creation, password management, and group assignments while maintaining detailed logs for auditing purposes.

Contributing

Calling all DevOps wizards! ‍♀️ Fork it, fix it, then pull request it! Let’s make this script even more magical. My GitHub ( https://github.com/teetoflame/Bash-script-HNG-2 )..

visit https://hng.tech/premium to learn more about HNG.
