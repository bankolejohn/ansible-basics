
-----

## Deploy and Configure Nginx Web Server using Ansible

### **Introduction**

Nginx (pronounced "engine-x") is a high-performance HTTP and reverse proxy server, as well as a mail proxy server and a generic TCP/UDP proxy server. It is renowned for its stability, rich feature set, simple configuration, and low resource consumption. Manually deploying and configuring Nginx on multiple servers can be a repetitive and time-consuming process, prone to human error.

This project will guide you through leveraging Ansible, an automation engine, to streamline the installation and configuration of an Nginx web server on a Linux machine. By creating reusable playbooks, you'll gain the skills to efficiently manage web server deployments across your infrastructure.

### **Objectives**

Upon successful completion of this project, you will be able to:

  * Understand how Ansible simplifies the deployment and configuration of applications like Nginx.
  * Set up a basic Ansible environment for managing Linux servers.
  * Create and execute an Ansible playbook to install the Nginx web server.
  * Configure a custom Nginx website by deploying HTML content and defining server blocks.
  * Verify the Nginx deployment and access the custom website from a web browser.

### **Prerequisites**

To follow along with this project, you will need:

  * **Linux Control Node:** One Linux server where Ansible is installed. (If you haven't installed Ansible yet, please refer to the "Setting up Ansible on a Linux Server" project or a similar Ansible installation guide.)
  * **Target Linux Server(s):** At least one additional Linux server that Ansible will manage. This server should be clean for Nginx installation.
  * **SSH Access:** SSH access from your control node to the target server(s) using public key authentication (passwordless SSH).
  * **Sudo Privileges:** The `ansible_user` on the control node must have `sudo` privileges on the target server(s) to install packages and modify system configurations.
  * **Tools:** A text editor (like `nano`, `vi`, or VS Code) to create and edit Ansible playbooks and inventory files.

### **Estimated Time**

2-3 hours

### **Tasks Outline**

1.  **Verify Ansible Installation and SSH Setup** on the control machine.
2.  **Set Up the Ansible Inventory File** for the target Linux server.
3.  **Create an Ansible Playbook to Install Nginx.**
4.  **Create an Ansible Playbook to Configure a Custom Nginx Website.**
5.  **Run the Playbooks** and **Verify the Nginx Deployment** and website access.

-----

### **Project Tasks: Detailed Steps**

Follow these detailed steps on your **Linux Control Node** unless explicitly stated otherwise.

#### **Task 1 - Verify Ansible Installation and SSH Setup**

Ensure your Ansible control node is correctly set up and can communicate with your target server.

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

The inventory file tells Ansible which server(s) it needs to manage for Nginx deployment.

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
    [web_servers]
    target1 ansible_host=<target-server-ip> ansible_user=<your-ssh-user>
    ```

      * **`[web_servers]`**: This defines a group named `web_servers`.
      * **`target1`**: This is an arbitrary name (alias) you give to your target machine within Ansible.
      * **`ansible_host=<target-server-ip>`**: Replace `<target-server-ip>` with the actual IP address or hostname of your Linux server.
      * **`ansible_user=<your-ssh-user>`**: Replace `<your-ssh-user>` with the username you use to SSH into the target server (this user must have `sudo` privileges on the target).
      * Save and close the file.

#### **Task 3 - Create an Ansible Playbook to Install Nginx**

This playbook will handle the installation of the Nginx package and ensure the service is running and enabled.

1.  **Create the playbook file:**

    ```bash
    nano install_nginx.yml
    ```

2.  **Add the following playbook content:**

    ```yaml
    # install_nginx.yml

    - name: Install Nginx on the server
      hosts: web_servers # Target the 'web_servers' group from inventory.ini
      become: yes          # Use sudo/root privileges on the target machine

      tasks:
        - name: Install Nginx package
          ansible.builtin.apt: # Use fully qualified collection name for clarity
            name: nginx
            state: present    # Ensure Nginx is installed
            update_cache: yes # Update apt cache before installing
          when: ansible_os_family == "Debian" # Use apt for Debian-based systems

        - name: Install Nginx package (RedHat)
          ansible.builtin.yum: # Use yum for RedHat-based systems
            name: nginx
            state: present
            update_cache: yes
          when: ansible_os_family == "RedHat"

        - name: Ensure Nginx service is running and enabled at boot
          ansible.builtin.service:
            name: nginx
            state: started # Ensure the service is started
            enabled: yes   # Ensure the service starts automatically on boot
    ```

      * **`ansible.builtin.apt` / `ansible.builtin.yum`**: These modules are used for package management on Debian-based (Ubuntu) and Red Hat-based (CentOS, RHEL, Fedora) systems respectively. The `when` condition ensures the correct module is used based on the target OS family detected by Ansible facts.
      * **`ansible.builtin.service`**: This module manages services on the target machine.

3.  **Save and close the file.**

#### **Task 4 - Configure a Custom Nginx Website Using Ansible**

This playbook will create a custom web directory, deploy an `index.html` file, set up an Nginx server block (virtual host), enable it, and remove the default Nginx configuration.

1.  **Create the playbook file:**

    ```bash
    nano configure_nginx.yml
    ```

2.  **Add the following playbook content:**

    ```yaml
    # configure_nginx.yml

    - name: Configure custom Nginx website
      hosts: web_servers
      become: yes
      tasks:
        - name: Create website root directory
          ansible.builtin.file:
            path: /var/www/mywebsite
            state: directory # Ensure it's a directory
            mode: '0755'     # Set permissions
            owner: www-data  # Set owner to Nginx user (common for Debian/Ubuntu)
            group: www-data  # Set group (common for Debian/Ubuntu)
          # Adjust owner/group for RedHat-based systems if needed (e.g., owner: nginx, group: nginx)

        - name: Deploy custom HTML content
          ansible.builtin.copy:
            content: |
              <!DOCTYPE html>
              <html>
              <head>
              <title>Welcome to My Custom Website</title>
              <style>
                body {
                  font-family: Arial, sans-serif;
                  background-color: #f0f0f0;
                  text-align: center;
                  padding-top: 50px;
                }
                h1 {
                  color: #333;
                }
                p {
                  color: #666;
                }
              </style>
              </head>
              <body>
              <h1>Hello from Ansible-configured Nginx!</h1>
              <p>This page was deployed automatically.</p>
              </body>
              </html>
            dest: /var/www/mywebsite/index.html # Destination path for the HTML file
            mode: '0644' # Set file permissions

        - name: Configure Nginx server block (Debian/Ubuntu style)
          ansible.builtin.copy:
            content: |
              server {
                  listen 80;
                  listen [::]:80;
                  server_name _; # Catch-all server name for basic setup
                  root /var/www/mywebsite;
                  index index.html index.htm;

                  location / {
                      try_files $uri $uri/ =404;
                  }
              }
            dest: /etc/nginx/sites-available/mywebsite # Nginx site configuration file
          when: ansible_os_family == "Debian"

        - name: Configure Nginx server block (RedHat/CentOS style)
          ansible.builtin.copy:
            content: |
              server {
                  listen 80;
                  listen [::]:80;
                  server_name _;
                  root /var/www/mywebsite;
                  index index.html index.htm;

                  location / {
                      try_files $uri $uri/ =404;
                  }
              }
            dest: /etc/nginx/conf.d/mywebsite.conf # Nginx site configuration file for RedHat
          when: ansible_os_family == "RedHat"

        - name: Enable the Nginx server block (Debian/Ubuntu)
          ansible.builtin.file:
            src: /etc/nginx/sites-available/mywebsite
            dest: /etc/nginx/sites-enabled/mywebsite
            state: link # Create a symbolic link to enable the site
          when: ansible_os_family == "Debian"

        - name: Remove default Nginx server block (Debian/Ubuntu)
          ansible.builtin.file:
            path: /etc/nginx/sites-enabled/default
            state: absent # Remove the symbolic link to disable the default site
          when: ansible_os_family == "Debian"

        - name: Test Nginx configuration for syntax errors
          ansible.builtin.command: nginx -t
          register: nginx_test_result
          changed_when: nginx_test_result.rc != 0 # Consider changed if test fails
          failed_when: "'emerg' in nginx_test_result.stderr" # Fail if critical error

        - name: Display Nginx test result
          ansible.builtin.debug:
            var: nginx_test_result.stderr

        - name: Reload Nginx service to apply new configuration
          ansible.builtin.service:
            name: nginx
            state: reloaded # Reload the service to apply config changes
    ```

      * **`ansible.builtin.file`**: Creates the `/var/www/mywebsite` directory.
      * **`ansible.builtin.copy`**: Deploys the `index.html` content and the Nginx server block configuration.
      * **Nginx Configuration Paths**: Note the `dest` paths for Nginx configuration:
          * Debian/Ubuntu: `/etc/nginx/sites-available/mywebsite` for config, symlinked to `/etc/nginx/sites-enabled/mywebsite`.
          * Red Hat/CentOS: `/etc/nginx/conf.d/mywebsite.conf` (often preferred for custom configs).
          * The `when` conditions ensure the correct paths are used based on the OS.
      * **`ansible.builtin.command: nginx -t`**: Runs Nginx's configuration test. This is a critical step before reloading to catch syntax errors.
      * **`ansible.builtin.service: name: nginx state: reloaded`**: Reloads Nginx to apply the new configuration without dropping connections.

3.  **Save and close the file.**

#### **Task 5 - Verify the Nginx Deployment**

Now, run the playbooks and confirm Nginx is working as expected.

1.  **Run the installation playbook:**

    ```bash
    ansible-playbook -i inventory.ini install_nginx.yml
    ```

      * Observe the output to ensure Nginx is installed and started. You should see `changed` messages for installation and service tasks.

2.  **Run the configuration playbook:**

    ```bash
    ansible-playbook -i inventory.ini configure_nginx.yml
    ```

      * Observe the output. You should see `changed` messages for directory creation, file deployment, symbolic link creation/removal, and the Nginx service reload. Pay attention to the `nginx -t` output for any errors.

3.  **Verify Nginx is running and responsive on the target server:**
    From your control node, use `curl` to access the web server.

    ```bash
    curl http://<target-server-ip>
    ```

      * Replace `<target-server-ip>` with the actual IP address of your target server.
      * **Expected Output:** You should see the HTML content you defined in `index.html`.
        ```html
        <!DOCTYPE html>
        <html>
        <head>
        <title>Welcome to My Custom Website</title>
        <style>
          body {
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
            text-align: center;
            padding-top: 50px;
          }
          h1 {
            color: #333;
          }
          p {
            color: #666;
          }
        </style>
        </head>
        <body>
        <h1>Hello from Ansible-configured Nginx!</h1>
        <p>This page was deployed automatically.</p>
        </body>
        </html>
        ```

4.  **Open the target server's IP address in a web browser:**
    Paste `http://<target-server-ip>` into your web browser. You should see the "Hello from Ansible-configured Nginx\!" page.

### **Conclusion**

In this project, you successfully automated the deployment and configuration of the Nginx web server on a Linux machine using Ansible. You created two distinct playbooks: one for installing Nginx and ensuring its service state, and another for deploying a custom website with its HTML content and Nginx server block configuration.

By leveraging Ansible's declarative nature, you've gained practical experience in streamlining web server setup, eliminating manual steps, and ensuring consistent configurations across multiple servers. These skills are invaluable for efficient system administration and DevOps practices. You can now further customize your Nginx configurations, deploy more complex applications, and scale your web infrastructure with ease.

### **Important Notes and Troubleshooting**

  * **Firewall:** Ensure that port 80 (HTTP) and potentially 443 (HTTPS) are open on your target machine's firewall (e.g., `ufw`, `firewalld`) and any cloud provider's security groups/network ACLs to allow inbound connections.
      * **Example for Ubuntu (UFW):**
        ```bash
        # On target server, if UFW is active:
        sudo ufw allow 'Nginx HTTP'
        sudo ufw enable # if not already enabled
        ```
      * **Example for CentOS/RHEL (firewalld):**
        ```bash
        # On target server, if firewalld is active:
        sudo firewall-cmd --permanent --add-service=http
        sudo firewall-cmd --reload
        ```
  * **Nginx User/Group:** The default Nginx user and group can vary by distribution (`www-data` on Debian/Ubuntu, `nginx` on Red Hat/CentOS). Ensure the `owner` and `group` in the `file` module (for `/var/www/mywebsite`) match your target system's Nginx user for proper permissions.
  * **Syntax Errors:** Pay close attention to YAML indentation in your playbooks, as it is crucial. Use online YAML validators if you encounter parsing errors.
  * **`server_name _`:** In the Nginx config, `server_name _;` acts as a catch-all for any domain. For a production site, you would replace `_` with your actual domain name (e.g., `server_name example.com www.example.com;`).
  * **HTTPS/SSL:** For a production website, you would add an HTTPS listener (port 443) and configure SSL certificates within your Nginx server block. Ansible has modules for managing SSL certificates (e.g., `acme`, `openssl_csr`, `copy` for existing certs).
