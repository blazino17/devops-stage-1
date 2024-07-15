Automating User Management with a Bash Script

Introduction


In a growing organization, managing user accounts manually can become cumbersome and error-prone. Automating this process ensures consistency, saves time, and enhances security.

 In this article, we’ll walk through a bash script that automates user creation, group assignments, and password generation. 
 
 This script is designed for a SysOps engineer tasked with onboarding new developers efficiently.

Script Breakdown

Our script, create_users.sh, 

performs the following tasks:

User Management Script

This Bash script is designed for managing user accounts on a Linux system. It creates users, assigns them to specified groups, sets up their home directories with appropriate permissions, and generates secure random passwords. The script requires root privileges to execute.

Prerequisites

The script must be run with root privileges.
Ensure that getent, groupadd, usermod, useradd, chpasswd, mkdir, and chmod commands are available on your system.

Script Overview

sudo bash create_users.sh <name-of-text-file>

Input File Format

The input file should contain lines in the following format:


username;group1,group2,group3

Each line represents a user to be created and the groups they should be added to. Comments and empty lines in the input file will be ignored.

Script Breakdown

Root Privileges Check
The script starts by checking if it is executed with root privileges. If not, it exits with an error message.


if (( UID != 0 ))
then
    echo "Script requires root accessibility"
    exit 1
fi

Define Log and Password File Paths
The script defines paths for the log file and a secure password file, creating and setting permissions for them as needed.


LOGFILE="/var/log/user_management.log"
PASSWORD_FILE="/var/secure/user_passwords.csv"
mkdir -p /var/secure
touch "$LOGFILE" "$PASSWORD_FILE"
chmod 600 "$PASSWORD_FILE"

Generate Random Password
A helper function generate_random_password is defined to create secure random passwords.

generate_random_password()
{
    local length=${1:-10}
    tr -dc 'A-Za-z0-9!?%+=' < /dev/urandom | head -c "$length"
}
Log Messages
A helper function log_message is used to log messages with timestamps.

log_message()
{
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> "$LOGFILE"
}

Create User Function
The create_user function handles creating the user, adding them to specified groups, setting up the home directory, and setting a password.


create_user()
{
    local username=$1
    local groups=$2

    if ! getent group "$username" > /dev/null; then
        groupadd "$username"
        log_message "Created personal group $username"
    fi

    if getent passwd "$username" > /dev/null; then
        usermod -g "$username" "$username"
        log_message "User $username already exists, modified primary group to $username"
    else
        useradd -m -g "$username" "$username"
        log_message "Created user $username"
    fi

    IFS=',' read -r -a groups_array <<< "$groups"
    for group in "${groups_array[@]}"; do
        group=$(echo "$group" | xargs)
        if ! getent group "$group" > /dev/null; then
            groupadd "$group"
            log_message "Created group $group"
        fi
        usermod -aG "$group" "$username"
        log_message "Added user $username to group $group"
    done

    chmod 700 "/home/$username"
    chown "$username:$username" "/home/$username"
    log_message "Set up home directory for user $username"

    if [ -z "$3" ]; then
        password=$(generate_random_password 12)
        echo "$username:$password" | chpasswd
        echo "$username,$password" >> "$PASSWORD_FILE"
        log_message "Set password for user $username"
    fi
}

Processing the Input File

The script reads the input file line by line and calls create_user for each user specified.


while IFS=';' read -r username groups; do
    [[ -z "$username" || "$username" =~ ^# ]] && continue
    if getent passwd "$username" > /dev/null; then
        create_user "$username" "$groups" "exists"
    else
        create_user "$username" "$groups"
    fi
done < "$1"

Completion Log

Finally, the script logs the completion of the user creation process.


log_message "User creation process completed."

This script automates user management tasks, making it easier to handle multiple user accounts and their configurations in a consistent and secure manner.

Error Handling
#The script handles existing users and groups, ensuring smooth execution without interruptions.

Conclusion
Automating user management tasks with a bash script enhances efficiency and security in a growing organization. This script provides a reliable solution for SysOps engineers to manage user accounts effectively.






