
-----

## Setting up Ansible on a Linux Server

### **Introduction**

Ansible is a powerful open-source automation tool that simplifies the management of IT infrastructure. It allows you to automate configuration management, application deployment, task automation, and orchestration. Unlike other configuration management tools, Ansible is agentless, meaning it communicates with managed nodes over standard SSH, making its setup straightforward.

This project will guide you through the process of installing and configuring Ansible on a Linux server, which will act as your "control node." You will also learn how to prepare your "target machines" for Ansible management using SSH key-based authentication.

### **Objectives**

Upon successful completion of this project, you will be able to:

  * Understand the fundamental concepts of Ansible.
  * Install and configure Ansible on a Linux control node.
  * Set up secure SSH key-based authentication between your control node and target nodes.
  * Create and manage an Ansible inventory file.
  * Verify Ansible's connectivity to your target machines using basic commands.

### **Prerequisites**

To follow along with this project, you will need:

  * **Linux Control Node:** One Linux server (e.g., Ubuntu, CentOS, Fedora, Debian) where Ansible will be installed. This can be a physical machine, a virtual machine (VM), or a cloud instance (e.g., AWS EC2, Azure VM).
  * **Target Machine(s):** At least one additional Linux server or virtual machine that Ansible will manage. This should also be accessible via SSH.
  * **SSH Access:** Initial SSH access to both the control node and the target node(s) using a password or existing key pair.
  * **Sudo Privileges:** A user account with `sudo` privileges on all machines.
  * **Tools:** Basic familiarity with the Linux command line and a text editor (like `nano` or `vi`).

### **Estimated Time**

1-2 hours

### **Tasks Outline**

1.  **Install Ansible** on the control node.
2.  **Configure SSH key-based authentication** for passwordless access to target nodes.
3.  **Create an inventory file** for your target machine(s).
4.  **Test Ansible connectivity** to target machine(s).
5.  **Run a simple Ansible ad-hoc command** to verify functionality.

-----

### **Project Tasks: Detailed Steps**

Follow these detailed steps on your **Linux Control Node** unless explicitly stated otherwise.

#### **Task 1 - Install Ansible on the Control Node**

This task involves installing the Ansible package on your designated control node.

1.  **Update the package repository:**
    It's good practice to update your system's package list to ensure you get the latest version of available packages.

    ```bash
    sudo apt update
    ```

      * **Note:** If you are using a Red Hat-based distribution (e.g., CentOS, Fedora, RHEL), use `sudo yum update -y` or `sudo dnf update -y`.

2.  **Install Ansible:**
    Install Ansible using your distribution's package manager.

    ```bash
    sudo apt install ansible -y
    ```

      * **For Red Hat-based systems (CentOS/RHEL/Fedora):**
        ```bash
        sudo yum install epel-release -y # Install EPEL repository first
        sudo yum install ansible -y
        # Or for Fedora/newer RHEL:
        # sudo dnf install ansible -y
        ```

3.  **Verify the installation:**
    After installation, confirm that Ansible is correctly installed and accessible by checking its version.

    ```bash
    ansible --version
    ```

    **Expected Output:**
    The output should display details about the installed Ansible version, its configuration path, Python version, and other relevant information, similar to this:

    ```
    ansible [core 2.15.5]
      config file = /etc/ansible/ansible.cfg
      configured module search path = ['/home/youruser/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
      ansible python module location = /usr/lib/python3/dist-packages/ansible
      ansible collection location = /home/youruser/.ansible/collections:/usr/share/ansible/collections
      executable location = /usr/bin/ansible
      python version = 3.10.12 (main, Nov 20 2023, 15:14:05) [GCC 11.4.0] (/usr/bin/python3)
      jinja version = 3.1.2
      libyaml version = 0.2.5
    ```

#### **Task 2 - Configure SSH Key-Based Authentication**

Ansible uses SSH to connect to and manage target machines. Setting up key-based authentication eliminates the need for passwords, making automation more secure and seamless. Perform these steps on your **control node**.

1.  **Generate an SSH key pair on the control node:**
    If you don't already have an SSH key pair (typically `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`), generate one.

    ```bash
    ssh-keygen -t rsa
    ```

      * When prompted for the file in which to save the key, press `Enter` to accept the default path (`~/.ssh/id_rsa`).
      * When prompted for a passphrase, you can press `Enter` twice to leave it empty for passwordless access (common in automation, but consider security implications for production). If you set a passphrase, you will need to use `ssh-agent` to manage it.

2.  **Copy the public key to the target machine(s):**
    Use `ssh-copy-id` to securely transfer your public key (`~/.ssh/id_rsa.pub`) to the target machine's `~/.ssh/authorized_keys` file. Repeat this command for each target machine you want to manage.

    ```bash
    ssh-copy-id user@<target-server-ip>
    ```

      * Replace `user` with the username you use to log in to the target server (e.g., `ec2-user`, `ubuntu`, `centos`).
      * Replace `<target-server-ip>` with the actual IP address or hostname of your target server.
      * You will be prompted for the password of the `user` on the target server the first time.

3.  **Test SSH access without a password:**
    Attempt to SSH into your target machine. If successful, you should log in without being prompted for a password.

    ```bash
    ssh user@<target-server-ip>
    ```

      * If you are prompted for a password, review the `ssh-copy-id` step and ensure your public key was correctly added to the target machine's `~/.ssh/authorized_keys` file for the specified user.
      * Type `exit` to log out of the target machine.

    You have now successfully configured passwordless SSH access from your control node to your target machine(s)\!

#### **Task 3 - Create an Inventory File**

The inventory file tells Ansible which servers to manage and how to connect to them.

1.  **Create a directory for Ansible configuration:**
    It's good practice to organize your Ansible files.

    ```bash
    mkdir ~/ansible
    cd ~/ansible
    ```

2.  **Create an inventory file:**
    Use your preferred text editor (e.g., `nano`, `vi`) to create a file named `inventory.ini`.

    ```bash
    nano inventory.ini
    ```

3.  **Add target machine details to the inventory:**
    Paste the following content into the `inventory.ini` file.

    ```ini
    [linux_servers]
    target1 ansible_host=<target1-ip> ansible_user=<user>
    target2 ansible_host=<target2-ip> ansible_user=<user>
    ```

      * **`[linux_servers]`**: This defines a group named `linux_servers`. You can define multiple groups (e.g., `[web_servers]`, `[db_servers]`).
      * **`target1`**, **`target2`**: These are arbitrary names (aliases) you give to your target machines within Ansible.
      * **`ansible_host=<target-ip>`**: Replace `<target1-ip>` and `<target2-ip>` with the actual IP addresses or hostnames of your target servers.
      * **`ansible_user=<user>`**: Replace `<user>` with the username you use to SSH into those target machines (e.g., `ec2-user`, `ubuntu`, `centos`). This user must have `sudo` privileges on the target machines.

4.  **Save and close the file:**

      * If using `nano`: Press `Ctrl+X`, then `Y` (for Yes), then `Enter`.

#### **Task 4 - Test Ansible Connectivity**

Now that Ansible is installed and the inventory is configured, let's test if Ansible can connect to your target machines.

1.  **Test Ansible connectivity to the target machines using the `ping` module:**
    The `ping` module is a simple way to check if Ansible can connect to a host, authenticate, and run a module.

    ```bash
    ansible -i inventory.ini linux_servers -m ping
    ```

      * **`-i inventory.ini`**: Specifies the inventory file to use.
      * **`linux_servers`**: Refers to the group defined in your `inventory.ini`.
      * **`-m ping`**: Instructs Ansible to use the `ping` module.

    **Expected Output:**
    You should see a `pong` response from each target machine, indicating successful connectivity and authentication.

    ```
    target1 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python3"
        },
        "changed": false,
        "ping": "pong"
    }
    target2 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python3"
        },
        "changed": false,
        "ping": "pong"
    }
    ```

      * If you see `UNREACHABLE` or `FAILED`, double-check:
          * The IP addresses in `inventory.ini`.
          * The `ansible_user` is correct and has SSH access.
          * Your SSH key-based authentication (Task 2).
          * Firewall rules on your control node or target node(s) (port 22 must be open).

#### **Task 5 - Run a Simple Ansible Ad-Hoc Command**

Ad-hoc commands are quick, one-liner commands used to perform tasks on your target machines without writing a full playbook.

1.  **Run a command to check the uptime of target machines:**
    The `command` module is used to run arbitrary commands on remote hosts.

    ```bash
    ansible -i inventory.ini linux_servers -m command -a "uptime"
    ```

      * **`-m command`**: Specifies the `command` module.
      * **`-a "uptime"`**: Passes the argument (the command to run) to the module.

    **Expected Output:**

    ```
    target1 | SUCCESS | rc=0 >>
     14:30:00 up 1 day, 5 hours, 30 min,  1 user,  load average: 0.00, 0.01, 0.05
    target2 | SUCCESS | rc=0 >>
     14:30:00 up 2 days, 1 hour, 15 min,  0 users,  load average: 0.00, 0.00, 0.00
    ```

2.  **Run a command to check disk usage:**
    The `shell` module is similar to `command` but allows for more complex shell features (like pipes, redirects).

    ```bash
    ansible -i inventory.ini linux_servers -m shell -a "df -h"
    ```

    **Expected Output:**

    ```
    target1 | SUCCESS | rc=0 >>
    Filesystem      Size  Used Avail Use% Mounted on
    devtmpfs        483M     0  483M   0% /dev
    tmpfs           492M     0  492M   0% /dev/shm
    tmpfs           492M  8.6M  483M   2% /run
    /dev/sda1        30G  4.5G   26G  15% /
    tmpfs           492M     0  492M   0% /tmp
    /dev/sdb1        50G  20G   30G  40% /data
    tmpfs            99M     0   99M   0% /run/user/1000
    target2 | SUCCESS | rc=0 >>
    Filesystem      Size  Used Avail Use% Mounted on
    devtmpfs        483M     0  483M   0% /dev
    tmpfs           492M     0  492M   0% /dev/shm
    tmpfs           492M  8.6M  483M   2% /run
    /dev/sda1        30G  4.5G   26G  15% /
    tmpfs           492M     0  492M   0% /tmp
    /dev/sdb1        50G  20G   30G  40% /data
    tmpfs            99M     0   99M   0% /run/user/1000
    ```

    Observe the outputs to confirm successful execution on all target machines.

### **Conclusion**

Congratulations\! You have successfully set up Ansible on your Linux control node and configured it to manage target machines. This project covered the essential steps: installing Ansible, establishing secure SSH key-based authentication, creating an inventory file, and verifying connectivity using ad-hoc commands.

With this foundation, you are now prepared to explore more advanced Ansible functionalities, such as writing idempotent playbooks to automate complex configurations, deploying applications, and orchestrating multi-tier environments. Ansible's simplicity and power make it an invaluable tool for any IT professional.

### **Important Notes and Troubleshooting**

  * **User Privileges:** The `ansible_user` specified in the inventory file must have `sudo` privileges on the target machines to execute commands that require root access (e.g., installing packages). If commands fail with permission errors, you might need to add `ansible_become=true` to your inventory or ad-hoc commands, or configure `sudoers` on the target machines.
      * Example with `become`: `ansible -i inventory.ini linux_servers -m apt -a "name=nginx state=present" --become`
  * **Firewall:** Ensure that port 22 (SSH) is open on your target machines' firewalls and any network security groups (if in a cloud environment) to allow inbound connections from your control node's IP address.
  * **SSH Host Key Checking:** If you encounter `Host key verification failed` errors, it might be due to a new/changed host key. You can temporarily disable strict host key checking by adding `StrictHostKeyChecking=no` and `UserKnownHostsFile=/dev/null` to your `~/.ssh/config` or by adding `-o StrictHostKeyChecking=no` to ad-hoc commands (not recommended for production). The best practice is to add the host's key to `~/.ssh/known_hosts` by connecting via `ssh` once manually.
  * **Python Interpreter:** Ansible relies on Python being installed on target machines. Most modern Linux distributions come with Python pre-installed. If not, you might need to manually install it or specify the Python interpreter path in your inventory (e.g., `ansible_python_interpreter=/usr/bin/python3`).
  * **Inventory File Location:** While `inventory.ini` in the current directory works, for more complex setups, you can place inventories in `/etc/ansible/hosts` (system-wide) or `~/.ansible/hosts`. You can also specify an inventory file using the `-i` flag or configure it in `ansible.cfg`.
