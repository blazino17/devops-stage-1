Automating User Management with a Bash Script

Introduction
In a growing organization, managing user accounts manually can become cumbersome and error-prone. Automating this process ensures consistency, saves time, and enhances security.
 In this article, we’ll walk through a bash script that automates user creation, group assignments, and password generation. 
 This script is designed for a SysOps engineer tasked with onboarding new developers efficiently.

Script Breakdown
Our script, create_users.sh, 
performs the following tasks:


INPUT_FILE="$1"
while IFS=';' read -r username groups; do
    ...
done < "$INPUT_FILE"
#The script reads a text file specified as an argument. Each line contains a username and associated groups, separated by a semicolon.

User and Group Creation

if ! id "$username" &>/dev/null; then
    useradd -m -g "$username" -G "$(echo "$groups" | tr ',' ' ')" "$username"
    ...
fi
#For each user, the script checks if the user already exists. If not, it creates the user and their personal group.

Home Directory Setup

chmod 700 "/home/$username"
chown "$username:$username" "/home/$username"
#The script sets up the user's home directory with appropriate permissions.

Password Generation and Assignment

password=$(generate_password)
echo "$username,$password" >> $PASSWORD_FILE
echo "$username:$password" | chpasswd
#A random password is generated for each user, stored securely, and set using the chpasswd command.

Logging Actions

echo "User $username created with groups: $groups" | tee -a $LOGFILE
#All actions are logged to /var/log/user_management.log.

Error Handling
#The script handles existing users and groups, ensuring smooth execution without interruptions.

Conclusion
Automating user management tasks with a bash script enhances efficiency and security in a growing organization. This script provides a reliable solution for SysOps engineers to manage user accounts effectively.
