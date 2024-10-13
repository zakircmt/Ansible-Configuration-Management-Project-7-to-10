
# Project Troubleshooting Study Guide

This study guide covers troubleshooting steps and solutions for various issues related to Jenkins, Git, Ansible, and SSH. It serves as a comprehensive reference to solve problems encountered during ANSIBLE CONFIGURATION MANAGEMENT (AUTOMATE PROJECT 7 TO 10).

---

## Table of Contents

1. [Jenkins Troubleshooting](#jenkins-troubleshooting)
   - [Jenkins WAR File Missing](#jenkins-war-file-missing)
   - [Jenkins Jobs Not Creating Directories](#jenkins-jobs-not-creating-directories)
   - [Completely Removing Jenkins](#completely-removing-jenkins)
2. [Git Branch Management](#git-branch-management)
   - [Merging Branches with Unrelated Histories](#merging-branches-with-unrelated-histories)
   - [Pulling Specific Files from Remote Repository](#pulling-specific-files-from-remote-repository)
3. [Ansible Configuration Management](#ansible-configuration-management)
   - [Inventory File Syntax Errors](#inventory-file-syntax-errors)
   - [SSH Connection Issues in Ansible](#ssh-connection-issues-in-ansible)
4. [SSH Authentication and Key Management](#ssh-authentication-and-key-management)
   - [SSH Agent and Key Issues](#ssh-agent-and-key-issues)
   - [Connecting via SSH Without a `.pem` File](#connecting-via-ssh-without-a-pem-file)
5. [Additional Commands and Tips](#additional-commands-and-tips)

---

## Jenkins Troubleshooting

### Jenkins WAR File Missing

**Problem:**

- Error: Unable to access jarfile `/usr/share/jenkins/jenkins.war`
- The `jenkins.war` file is missing or located in a different directory.

**Solution:**

1. **Locate the `jenkins.war` File:**

    ```bash
    sudo find / -name jenkins.war
    ```

2. **Update the Service File with the Correct Path:**

   - If the WAR file is found at `/usr/share/java/jenkins.war`, update the service file:

     ```bash
     sudo nano /etc/systemd/system/jenkins.service
     ```

   - Update the `ExecStart` line:

     ```ini
     ExecStart=/usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war
     ```

3. **Reload and Restart Jenkins:**

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl restart jenkins
    ```

4. **Ensure Correct Permissions:**

    ```bash
    sudo chown jenkins:jenkins /usr/share/java/jenkins.war
    sudo chmod 644 /usr/share/java/jenkins.war
    ```

---

### Jenkins Jobs Not Creating Directories

**Problem:**

- Jenkins jobs are not creating the necessary directories or archiving artifacts.

**Solution:**

1. **Verify Job Configuration:**

   Ensure that the job is configured to archive artifacts under *Post-build Actions*.

2. **Check Permissions:**

    ```bash
    sudo chown -R jenkins:jenkins /var/lib/jenkins/jobs/
    sudo chmod -R 755 /var/lib/jenkins/jobs/
    ```

3. **Create Missing Directories Manually:**

    ```bash
    sudo mkdir -p /var/lib/jenkins/jobs/<job_name>/builds/<build_number>/archive/
    sudo chown -R jenkins:jenkins /var/lib/jenkins/jobs/<job_name>/
    ```

4. **Check Jenkins Logs for Errors:**

    ```bash
    sudo tail -f /var/log/jenkins/jenkins.log
    ```

---


## Git Branch Management

### Merging Branches with Unrelated Histories

**Problem:**

- Error: There isn't anything to compare. `main` and `feature/branch` are entirely different commit histories.

**Solution:**

1. **Use `--allow-unrelated-histories` Flag:**

    ```bash
    git checkout main
    git merge feature/branch --allow-unrelated-histories
    ```

2. **Rebase the Feature Branch onto `main`:**

    ```bash
    git checkout feature/branch
    git rebase main
    ```

3. **Force Push (Use with Caution):**

    ```bash
    git push -f origin main
    ```

---

### Pulling Specific Files from Remote Repository

**Problem:**

- Need to pull a specific file from the remote repository without pulling all changes.

**Solution:**

1. **Fetch Latest Changes:**

    ```bash
    git fetch origin
    ```

2. **Checkout the Specific File:**

    ```bash
    git checkout origin/main -- path/to/file
    ```

3. **Commit and Push Changes (if necessary):**

    ```bash
    git add path/to/file
    git commit -m "Pulled specific file from origin/main"
    git push origin your-branch
    ```

---

## Ansible Configuration Management

### Inventory File Syntax Errors

**Problem:**

- Errors parsing the `inventory/dev.yml` file due to YAML syntax issues.

**Solution:**

1. **Correct YAML Syntax:**

   Ensure proper indentation and structure in the inventory file.

   Example of a correctly formatted inventory file:

    ```yaml
    all:
      children:
        webserver:
          hosts:
            172.31.33.239:
              ansible_ssh_user: ec2-user
            172.31.37.97:
              ansible_ssh_user: ec2-user
        nfs:
          hosts:
            172.31.44.71:
              ansible_ssh_user: ec2-user
    ```

2. **Validate YAML File:**

   Use a YAML validator to check for syntax errors.

---

### SSH Connection Issues in Ansible

**Problem:**

- Ansible fails to connect to hosts with errors like *UNREACHABLE* due to SSH authentication issues.

**Solution:**

1. **Specify SSH Private Key in Inventory File:**

    ```yaml
    ansible_ssh_private_key_file: /path/to/private-key.pem
    ```

2. **Disable Strict Host Key Checking (if necessary):**

    ```yaml
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
    ```

3. **Ensure Correct Permissions on SSH Key:**

    ```bash
    chmod 400 /path/to/private-key.pem
    ```

4. **Test SSH Connection Manually:**

    ```bash
    ssh -i /path/to/private-key.pem user@host_ip
    ```

---

## SSH Authentication and Key Management

### SSH Agent and Key Issues

**Problem:**

- Errors when adding SSH keys to the agent or during authentication.

**Solution:**

1. **Start the SSH Agent:**

    ```bash
    eval $(ssh-agent)
    ```

2. **Add Your SSH Private Key:**

    ```bash
    ssh-add /path/to/private-key.pem
    ```

3. **Verify Keys Are Added:**

    ```bash
    ssh-add -l
    ```

---

### Connecting via SSH Without a `.pem` File

**Problem:**

- Need to connect to an EC2 instance without using a `.pem` file.

**Solution:**

1. **Set Up Password-Based Authentication:**

   Update the EC2 instance's `/etc/ssh/sshd_config` file to allow password-based login:

    ```bash
    PasswordAuthentication yes
    ```

2. **Create a New User and Set a Password:**

    ```bash
    sudo adduser newuser
    sudo passwd newuser
    ```

3. **Restart the SSH Service:**

    ```bash
    sudo systemctl restart sshd
    ```

---

## Additional Commands and Tips

### Resetting Jenkins Admin Password:

- If you lose the Jenkins admin password, you can reset it by deleting the `config.xml` file and restarting Jenkins:

    ```bash
    sudo rm /var/lib/jenkins/config.xml
    sudo systemctl restart jenkins
    ```

### List Active SSH Connections:

    ```bash
    sudo netstat -tnpa | grep 'ESTABLISHED.*sshd'
    ```

### Common Git Commands:

- `git status` – Check the status of your current branch.
- `git log` – View commit history.
- `git reset --hard` – Reset working directory to the last commit.

---

**Prepared by:** [Md. Zakir Hossen] https://linkedin.com/in/zakircmt/

**Date:** October 2024
