## Automate File Backup and Restore on a Linux Server using Ansible

### **Introduction**

Data integrity and availability are paramount in server management. Regular backups are crucial for disaster recovery, ensuring that your critical files and configurations can be restored in case of data loss, corruption, or system failures. Manually performing backups and restorations across multiple Linux servers can be a tedious, error-prone, and time-consuming task.

Ansible, as an automation engine, provides a powerful and repeatable solution for managing these essential administrative processes. This project will guide you through creating Ansible playbooks to automate the backup and restore procedures for files on a Linux server, enhancing your operational efficiency and data safety.

### **Objectives**

Upon successful completion of this project, you will be able to:

  * Understand how Ansible simplifies routine administrative tasks like backup and restoration.
  * Set up a basic Ansible environment for managing Linux servers.
  * Create an Ansible playbook to securely back up specified files or directories to a designated location on the target server.
  * Develop a separate Ansible playbook to restore files from a backup to their original location.
  * Test and verify the integrity and functionality of both the backup and restore processes.

### **Prerequisites**

To follow along with this project, you will need:

  * **Linux Control Node:** One Linux server where Ansible is installed. (If you haven't installed Ansible yet, please refer to the "Setting up Ansible on a Linux Server" project or a similar Ansible installation guide.)
  * **Target Linux Server(s):** At least one additional Linux server that Ansible will manage. This server will be the source of files to be backed up and restored.
  * **SSH Access:** SSH access from your control node to the target server(s) using public key authentication (passwordless SSH).
  * **Sudo Privileges:** The `ansible_user` on the control node must have `sudo` privileges on the target server(s) to create directories and copy files in system-level paths.
  * **Tools:** A text editor (like `nano`, `vi`, or VS Code) to create and edit Ansible playbooks and inventory files.

### **Estimated Time**

2-3 hours

### **Tasks Outline**

1.  **Verify Ansible Installation and SSH Setup** on the control machine.
2.  **Set Up the Ansible Inventory File** for the target Linux server.
3.  **Create Sample Files/Directory** on the target server for testing.
4.  **Create an Ansible Playbook to Back Up Files.**
5.  **Create an Ansible Playbook to Restore Files from a Backup.**
6.  **Test the Backup and Restore Functionality.**

-----

### **Project Tasks: Detailed Steps**

Follow these detailed steps on your **Linux Control Node** unless explicitly stated otherwise.

#### **Task 1 - Verify Ansible Installation and SSH Setup**

Ensure your Ansible control node is correctly set up and can communicate with your target server.

1.  **Verify Ansible installation:**

    ```bash
    ansible --version
    ```

      * Confirm Ansible is installed and its version is displayed.

2.  **Verify SSH key-based authentication to your target server:**
    Ensure you can SSH into your target server without a password.

    ```bash
    ssh user@<target-server-ip>
    ```

      * Replace `user` with your SSH username and `<target-server-ip>` with the target server's IP. If you are prompted for a password, set up SSH key-based authentication first.
      * Type `exit` to log out of the target server.

#### **Task 2 - Set Up the Ansible Inventory File**

The inventory file defines the target server(s) that Ansible will manage for backup and restoration.

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

#### **Task 3 - Create Sample Files/Directory on the Target Server**

To effectively test the backup and restore process, you need some sample files on your target server.

1.  **SSH into your target server:**

    ```bash
    ssh <your-ssh-user>@<target-server-ip>
    ```

2.  **Create a sample directory and files:**
    We'll use `/opt/my_app_data` as the source directory for backup.

    ```bash
    sudo mkdir -p /opt/my_app_data
    echo "This is a test file for backup." | sudo tee /opt/my_app_data/file1.txt
    echo "Another important configuration." | sudo tee /opt/my_app_data/config.conf
    sudo mkdir -p /opt/my_app_data/sub_dir
    echo "Content in subdirectory." | sudo tee /opt/my_app_data/sub_dir/file2.txt
    sudo chown -R <your-ssh-user>:<your-ssh-user> /opt/my_app_data # Set ownership for easier testing
    sudo chmod -R 755 /opt/my_app_data
    ls -lR /opt/my_app_data
    ```

3.  **Exit the target server:**

    ```bash
    exit
    ```

#### **Task 4 - Create an Ansible Playbook to Back Up Files**

This playbook will create a timestamped archive of your target files and store it on the target server.

1.  **Create the playbook file:**

    ```bash
    nano backup.yml
    ```

2.  **Add the following playbook content:**

    ```yaml
    # backup.yml

    - name: Backup files on the server
      hosts: linux_servers
      become: yes # Use sudo for creating /backup directory and archiving system files

      vars:
        # Define the source directory to backup
        source_directory: "/opt/my_app_data"
        # Define the destination directory for backups on the target server
        backup_base_dir: "/var/backups/ansible"
        # Generate a timestamp for the backup filename
        backup_timestamp: "{{ ansible_date_time.iso8601_basic_short }}"
        # Define the full path for the backup archive
        backup_archive_name: "{{ backup_base_dir }}/{{ source_directory | basename }}-{{ backup_timestamp }}.tar.gz"

      tasks:
        - name: Ensure backup base directory exists
          ansible.builtin.file:
            path: "{{ backup_base_dir }}"
            state: directory
            mode: '0700' # Secure permissions for backup directory
            owner: root
            group: root

        - name: Create a compressed archive of the source directory
          ansible.builtin.archive:
            path: "{{ source_directory }}" # Directory or file to archive
            dest: "{{ backup_archive_name }}" # Destination path for the archive
            format: gz # Use gzip compression
            remove: false # Do not remove source files after archiving
          register: backup_result

        - name: Display backup status
          ansible.builtin.debug:
            msg: "Backup of {{ source_directory }} created at {{ backup_archive_name }}. Changed: {{ backup_result.changed }}"
    ```

      * **`become: yes`**: Necessary for creating directories like `/var/backups` and archiving files that might require root permissions.
      * **`source_directory`**: The path on the target server that you want to back up.
      * **`backup_base_dir`**: The directory on the target server where backups will be stored.
      * **`ansible_date_time.iso8601_basic_short`**: A built-in Ansible fact that provides a timestamp, useful for unique backup filenames.
      * **`ansible.builtin.archive` module**: This module is excellent for creating compressed archives of files or directories directly on the remote host.
      * **`register: backup_result`**: Captures the output of the `archive` task into a variable for debugging.

3.  **Save and close the file.**

#### **Task 5 - Create an Ansible Playbook to Restore Files**

This playbook will extract the latest backup archive and restore its contents to the original location.

1.  **Create the playbook file:**

    ```bash
    nano restore.yml
    ```

2.  **Add the following playbook content:**

    ```yaml
    # restore.yml

    - name: Restore files from backup
      hosts: linux_servers
      become: yes # Use sudo for restoring to original location

      vars:
        # Define the original destination directory for restoration
        original_destination_dir: "/opt/my_app_data"
        # Define the base directory where backups are stored
        backup_base_dir: "/var/backups/ansible"

      tasks:
        - name: Find the latest backup archive
          ansible.builtin.find:
            paths: "{{ backup_base_dir }}"
            patterns: "{{ original_destination_dir | basename }}-*.tar.gz" # Find archives matching pattern
            sort: "name" # Sort by name to get the latest (due to timestamp)
            # You might need to adjust sort: "ctime" or "mtime" if timestamps are not strictly increasing
          register: latest_backup_files

        - name: Fail if no backup archive is found
          ansible.builtin.fail:
            msg: "No backup archive found for {{ original_destination_dir | basename }} in {{ backup_base_dir }}."
          when: latest_backup_files.files | length == 0

        - name: Set fact for the latest backup archive path
          ansible.builtin.set_fact:
            latest_backup_archive: "{{ latest_backup_files.files | last }}" # Get the last (latest) file from sorted list

        - name: Display the backup archive to be restored
          ansible.builtin.debug:
            msg: "Restoring from archive: {{ latest_backup_archive.path }}"

        - name: Ensure original destination directory exists before extraction
          ansible.builtin.file:
            path: "{{ original_destination_dir }}"
            state: directory
            mode: '0755'
            owner: "{{ ansible_user }}" # Set owner to the ansible user or specific app user
            group: "{{ ansible_user }}" # Set group
            recurse: yes

        - name: Extract backup archive to a temporary location
          ansible.builtin.unarchive:
            src: "{{ latest_backup_archive.path }}" # Source archive on remote host
            dest: "/tmp/restore_temp" # Temporary extraction directory on remote host
            remote_src: yes # Source is on the remote host
            creates: "/tmp/restore_temp/{{ original_destination_dir | basename }}" # Only extract if not already extracted
          register: unarchive_result

        - name: Copy restored files to original destination (overwriting existing)
          ansible.builtin.copy:
            src: "/tmp/restore_temp/{{ original_destination_dir | basename }}/" # Source from temp extraction
            dest: "{{ original_destination_dir }}/" # Original destination
            remote_src: yes # Source is on the remote host
            owner: "{{ ansible_user }}" # Ensure correct ownership
            group: "{{ ansible_user }}"
            mode: '0755' # Ensure correct permissions for directories
            recurse: yes # Copy contents recursively
          register: copy_result

        - name: Clean up temporary extraction directory
          ansible.builtin.file:
            path: "/tmp/restore_temp"
            state: absent
          when: unarchive_result.changed # Only clean up if extraction actually happened

        - name: Display restore status
          ansible.builtin.debug:
            msg: "Files restored to {{ original_destination_dir }}. Changed: {{ copy_result.changed }}"
    ```

      * **`original_destination_dir`**: The path on the target server where files should be restored.
      * **`ansible.builtin.find`**: Used to locate the latest backup archive based on the naming convention.
      * **`ansible.builtin.fail`**: Stops the playbook if no backup is found.
      * **`ansible.builtin.set_fact`**: Dynamically sets a variable `latest_backup_archive` to the path of the latest backup.
      * **`ansible.builtin.unarchive`**: Extracts the compressed archive. `remote_src: yes` is crucial as the archive is on the target server.
      * **`ansible.builtin.copy` with `remote_src: yes`**: Copies the extracted files from the temporary location to their original destination.
      * **`ansible.builtin.file state: absent`**: Cleans up the temporary directory after restoration.

3.  **Save and close the file.**

#### **Task 6 - Test the Backup and Restore Functionality**

Now, let's run the playbooks and verify the process.

1.  **Run the backup playbook:**

    ```bash
    ansible-playbook -i inventory.ini backup.yml
    ```

      * **Expected Output:** You should see `changed` status for the archive creation task.

2.  **Verify the backup directory and files on the target server:**
    SSH into your target server to confirm the backup archive was created.

    ```bash
    ssh <your-ssh-user>@<target-server-ip>
    ls -l /var/backups/ansible/
    # Example output: my_app_data-20240715T163000.tar.gz
    exit
    ```

3.  **Simulate data loss (Optional but Recommended for testing):**
    To truly test the restore, delete or modify some files in your source directory on the target server.

    ```bash
    ssh <your-ssh-user>@<target-server-ip>
    sudo rm /opt/my_app_data/file1.txt
    echo "Modified content." | sudo tee /opt/my_app_data/config.conf
    ls -lR /opt/my_app_data
    exit
    ```

4.  **Run the restore playbook:**

    ```bash
    ansible-playbook -i inventory.ini restore.yml
    ```

      * **Expected Output:** You should see `changed` status for the unarchive and copy tasks.

5.  **Verify the restored files in the original location on the target server:**
    SSH back into your target server to confirm the files are restored.

    ```bash
    ssh <your-ssh-user>@<target-server-ip>
    ls -lR /opt/my_app_data
    cat /opt/my_app_data/file1.txt # Should show original content
    cat /opt/my_app_data/config.conf # Should show original content
    exit
    ```

    The `file1.txt` should reappear, and `config.conf` should revert to its original content.

### **Conclusion**

In this project, you successfully automated file backup and restoration on a Linux server using Ansible. You created a robust backup playbook that generates timestamped, compressed archives and a corresponding restore playbook that intelligently finds and extracts the latest backup.

These skills are fundamental for maintaining data safety and ensuring business continuity. You can extend these playbooks to:

  * **Backup multiple directories:** Define a list of `source_directory` paths.
  * **Offsite backups:** Use the `ansible.builtin.s3` or `ansible.builtin.gcp_storage_object` modules (or `ansible.builtin.copy` with `delegate_to: localhost` after `fetch`) to transfer archives to cloud storage.
  * **Scheduled backups:** Integrate these playbooks with cron jobs on your control node.
  * **Advanced options:** Implement encryption for archives, checksum verification, or retention policies.
  * **`synchronize` module:** For large directories or frequent incremental backups, consider using the `ansible.posix.synchronize` module, which leverages `rsync` for more efficient file transfers and synchronization.

### **Important Notes and Troubleshooting**

  * **Permissions:** Ensure the `ansible_user` has `sudo` privileges and the necessary read/write permissions on the `source_directory` and `backup_base_dir` on the target server.
  * **Disk Space:** Ensure sufficient disk space on the target server for both the original files and the backup archives.
  * **`remote_src: yes`**: This parameter is crucial when the source file/directory for a `copy` or `unarchive` operation is located on the *remote* (target) machine, not the control node.
  * **Finding Latest Backup:** The `ansible.builtin.find` module with `sort: "name"` works well when timestamps are part of the filename and are in a format that sorts chronologically (like `YYYYMMDDTHHMMSS`). For other naming conventions, you might need to adjust the `sort` parameter or use a more complex `regex` pattern.
  * **Idempotency:** While the backup creates a new archive each time, the restore process is designed to be more idempotent, ensuring the target state matches the backup.
  * **Real-world Backups:** For critical production systems, consider a multi-layered backup strategy including database backups (e.g., `mysqldump`), offsite storage, and regular testing of restoration procedures.
