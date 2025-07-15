
-----

## Automate User Creation on Linux Server using Ansible

### **Introduction**

Managing user accounts is a fundamental administrative task for Linux servers. Manually creating and managing user accounts can be time-consuming and error-prone, especially when dealing with multiple servers or a large number of users. Ansible, a powerful automation engine, simplifies this process by allowing you to define user configurations declaratively in playbooks.

This project will guide you through creating an Ansible playbook to automate user creation on one or more Linux servers. You will learn how to define user properties like home directories, group memberships, and SSH key-based access, ensuring consistency and efficiency in your server management.

### **Objectives**

Upon successful completion of this project, you will be able to:

  * Understand how Ansible can automate user management tasks.
  * Set up a basic Ansible environment to manage Linux servers.
  * Create an Ansible playbook to define and automate user creation.
  * Configure additional user settings such as group memberships and SSH public key authentication.
  * Verify the successful creation of users and test their login capabilities.

### **Prerequisites**

To follow along with this project, you will need:

  * **Linux Control Node:** One Linux server where Ansible is installed. (If you haven't installed Ansible yet, please refer to the "Setting up Ansible on a Linux Server" project or a similar Ansible installation guide.)
  * **Target Linux Server(s):** At least one additional Linux server that Ansible will manage.
  * **SSH Access:** SSH access from your control node to the target server(s) using public key authentication (passwordless SSH).
  * **Sudo Privileges:** The `ansible_user` on the control node must have `sudo` privileges on the target server(s) to create users.
  * **Text Editor:** A text editor (like `nano`, `vi`, or VS Code) to create and edit Ansible playbooks and inventory files.

### **Estimated Time**

1-2 hours

### **Tasks Outline**

1.  **Verify Ansible Installation and SSH Setup** on the control machine.
2.  **Set Up the Ansible Inventory File** for the target Linux server.
3.  **Create SSH Public Key Files** for the new users.
4.  **Create an Ansible Playbook** to automate user creation with basic settings.
5.  **Configure Additional User Settings** in the playbook (groups and SSH access).
6.  **Run the Playbook** and **Verify User Creation** and login.

-----

### **Project Tasks: Detailed Steps**

Follow these detailed steps on your **Linux Control Node** unless explicitly stated otherwise.

#### **Task 1 - Verify Ansible Installation and SSH Setup**

Before starting, ensure your Ansible control node is ready.

1.  **Verify Ansible installation:**

    ```bash
    ansible --version
    ```

      * Confirm Ansible is installed and its version is displayed. If not, install it using `sudo apt update && sudo apt install ansible -y` (for Ubuntu/Debian) or equivalent for your distribution.

2.  **Verify SSH key-based authentication to your target server:**
    Ensure you can SSH into your target server without a password.

    ```bash
    ssh user@<target-server-ip>
    ```

      * Replace `user` with your SSH username and `<target-server-ip>` with the target server's IP. If you are prompted for a password, you need to set up SSH key-based authentication first (e.g., using `ssh-keygen -t rsa` and `ssh-copy-id user@<target-server-ip>`).
      * Type `exit` to log out of the target server.

#### **Task 2 - Set Up the Ansible Inventory File**

The inventory file defines the target server(s) that Ansible will manage.

1.  **Navigate to your Ansible configuration directory:**
    If you don't have one, create it:

    ```bash
    mkdir -p ~/ansible
    cd ~/ansible
    ```

2.  **Create or edit your inventory file:**

    ```bash
    nano inventory.ini
    ```

3.  **Add your target server details:**

    ```ini
    [linux_servers]
    target1 ansible_host=<target-server-ip> ansible_user=<your-ssh-user>
    ```

      * Replace `<target-server-ip>` with the actual IP address or hostname of your Linux server.
      * Replace `<your-ssh-user>` with the username you use to SSH into the target server (this user must have `sudo` privileges on the target).
      * Save and close the file.

#### **Task 3 - Create SSH Public Key Files for New Users**

For the new users (`user1`, `user2`) to be able to SSH into the target server, they will need their own SSH key pairs. For this project, you'll create placeholder public key files on your control node. In a real scenario, these would be the actual public keys provided by the users.

1.  **Create a directory for SSH keys (e.g., `~/ansible/keys`):**

    ```bash
    mkdir -p ~/ansible/keys
    ```

2.  **Create placeholder public key files:**

    ```bash
    echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC... user1@example.com" > ~/ansible/keys/user1.pub
    echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC... user2@example.com" > ~/ansible/keys/user2.pub
    ```

      * **IMPORTANT:** Replace the example SSH key material (`AAAAB3NzaC1yc2E...`) with actual valid SSH public key content if you want to test real SSH login later. For a simple demo, any string will allow the playbook to run, but actual login will fail without a matching private key. You can generate real keys with `ssh-keygen -t rsa -f ~/.ssh/user1_rsa` and then copy `~/.ssh/user1_rsa.pub` content.

#### **Task 4 - Create an Ansible Playbook to Automate User Creation**

Now, let's create the Ansible playbook that defines the user creation process.

1.  **Create the playbook file:**

    ```bash
    nano create_users.yml
    ```

2.  **Add the following playbook content:**
    This playbook uses the `user` module to create users and the `authorized_key` module to add SSH public keys.

    ```yaml
    # create_users.yml

    - name: Automate user creation and SSH key setup
      hosts: linux_servers # Target the 'linux_servers' group from inventory.ini
      become: yes          # Use sudo/root privileges on the target machine

      vars:
        # Define the users and their properties in a structured way
        new_users:
          - username: "user1"
            groups: "sudo,www-data" # Example: user1 in sudo and www-data groups
            ssh_key_path: "~/ansible/keys/user1.pub" # Path to user1's public key on the control node

          - username: "user2"
            groups: "docker" # Example: user2 in docker group
            ssh_key_path: "~/ansible/keys/user2.pub" # Path to user2's public key on the control node

      tasks:
        - name: Create new users with specified settings
          ansible.builtin.user:
            name: "{{ item.username }}"       # User's name
            state: present                   # Ensure the user exists
            shell: /bin/bash                 # Set default shell
            create_home: yes                 # Create home directory if it doesn't exist
            groups: "{{ item.groups }}"      # Assign to specified groups
            append: yes                      # Append to existing groups if user already exists
          loop: "{{ new_users }}"            # Loop through the 'new_users' variable

        - name: Add SSH public key for each user
          ansible.posix.authorized_key: # Use fully qualified collection name for clarity
            user: "{{ item.username }}"      # Target user
            state: present                   # Ensure the key is present
            key: "{{ lookup('file', item.ssh_key_path) }}" # Read key content from file
          loop: "{{ new_users }}"            # Loop through the 'new_users' variable
    ```

      * **`hosts: linux_servers`**: Specifies that this playbook will run on all hosts in the `linux_servers` group defined in your `inventory.ini`.

      * **`become: yes`**: Instructs Ansible to use privilege escalation (e.g., `sudo`) to run tasks as root on the target machine, which is necessary for user management.

      * **`vars`**: Defines a list of dictionaries (`new_users`) where each dictionary represents a user and their properties (`username`, `groups`, `ssh_key_path`). This makes the playbook highly flexible and easy to extend.

      * **`loop: "{{ new_users }}"`**: This loop iterates over each item in the `new_users` list, running the `user` and `authorized_key` tasks for each user.

      * **`user` module**: Manages user accounts. `state: present` ensures the user exists. `groups` specifies the groups the user should be a member of.

      * **`authorized_key` module**: Manages SSH authorized keys for users.

      * **`lookup('file', item.ssh_key_path)`**: This is a Jinja2 template filter that reads the content of the specified file (`user1.pub`, `user2.pub`) from the control node and inserts it as the `key` value for the `authorized_key` module.

      * Save and close the file.

#### **Task 5 - Run the Playbook and Verify User Creation**

Now, execute the playbook to automate the user creation process.

1.  **Run the playbook:**
    Make sure you are in the directory containing `inventory.ini` and `create_users.yml`.

    ```bash
    ansible-playbook -i inventory.ini create_users.yml
    ```

      * Ansible will connect to your target server(s), execute the tasks, and report the status. You should see `changed` status for the tasks, indicating users were created/modified.

    **Expected Output (partial):**

    ```
    PLAY [Automate user creation and SSH key setup] *********************************

    TASK [Gathering Facts] *********************************************************
    ok: [target1]

    TASK [Create new users with specified settings] ********************************
    changed: [target1] => (item={'username': 'user1', 'groups': 'sudo,www-data', 'ssh_key_path': '~/ansible/keys/user1.pub'})
    changed: [target1] => (item={'username': 'user2', 'groups': 'docker', 'ssh_key_path': '~/ansible/keys/user2.pub'})

    TASK [Add SSH public key for each user] ****************************************
    changed: [target1] => (item={'username': 'user1', 'groups': 'sudo,www-data', 'ssh_key_path': '~/ansible/keys/user1.pub'})
    changed: [target1] => (item={'username': 'user2', 'groups': 'docker', 'ssh_key_path': '~/ansible/keys/user2.pub'})

    PLAY RECAP *********************************************************************
    target1                    : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ```

2.  **Verify the users were created on the target server:**
    SSH into your target server and check the `/etc/passwd` file and home directories.

    ```bash
    ssh user@<target-server-ip>
    cat /etc/passwd | grep user
    ls -l /home/
    exit
    ```

    You should see entries for `user1` and `user2` in `/etc/passwd` and their respective home directories under `/home/`.

3.  **Test SSH access for the newly created users (if you used real SSH keys):**
    If you generated actual SSH key pairs for `user1` and `user2` and copied their public keys into the `~/ansible/keys` directory on your control node, you can now test logging in as these users using their private keys.

    ```bash
    # Assuming you have user1's private key at ~/.ssh/user1_rsa
    ssh -i ~/.ssh/user1_rsa user1@<target-server-ip>
    # Assuming you have user2's private key at ~/.ssh/user2_rsa
    ssh -i ~/.ssh/user2_rsa user2@<target-server-ip>
    ```

    If you did not use real SSH keys, this step will fail, which is expected.

### **Conclusion**

In this project, you successfully automated the creation and configuration of user accounts on a Linux server using Ansible. You learned how to define users, assign them to groups, and configure SSH key-based access within an Ansible playbook.

This capability is fundamental for efficient system administration, allowing you to manage user accounts consistently and at scale across multiple servers. You can extend this playbook for more advanced configurations, such as setting up password policies, managing user deletion (`state: absent`), or deploying specific user-level configurations.
