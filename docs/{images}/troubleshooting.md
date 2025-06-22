# 🔧 Ansible AWS Automation - Troubleshooting Guide

> **Complete solutions for common issues you might encounter during setup and execution**

## 📋 Quick Diagnostic Checklist

Before diving into specific issues, run this quick checklist:

```bash
# 1. Check Ansible installation
ansible --version

# 2. Test basic connectivity
ansible -i inventory/hosts.ini all -m ping

# 3. Verify SSH access manually
ssh -i ~/.ssh/your-key.pem ec2-user@TARGET-PRIVATE-IP

# 4. Check target server status from AWS Console
# 5. Verify Security Group settings
```

## 🚨 Common Issues and Solutions

### 1. SSH Connection Problems

#### Issue: "Permission denied (publickey)"

```bash
# Error message:
# target-server | UNREACHABLE! => {
#     "msg": "Failed to connect to the host via ssh: Permission denied (publickey)."
# }
```

**Solutions:**

```bash
# Solution A: Fix key permissions
chmod 600 ~/.ssh/your-key.pem
ls -la ~/.ssh/your-key.pem  # Should show: -rw-------

# Solution B: Verify key content
cat ~/.ssh/your-key.pem  # Should start with -----BEGIN RSA PRIVATE KEY-----

# Solution C: Test SSH manually
ssh -i ~/.ssh/your-key.pem ec2-user@TARGET-PRIVATE-IP -v  # Verbose output

# Solution D: Update inventory with correct path
nano inventory/hosts.ini
# Ensure: ansible_ssh_private_key_file=~/.ssh/your-key.pem
```

#### Issue: "Host key verification failed"

```bash
# Error message:
# Host key verification failed.
```

**Solutions:**

```bash
# Solution A: Add to known hosts
ssh-keyscan -H TARGET-PRIVATE-IP >> ~/.ssh/known_hosts

# Solution B: Use StrictHostKeyChecking=no (already in inventory)
# This is included in the inventory file: ansible_ssh_common_args='-o StrictHostKeyChecking=no'

# Solution C: Clear old host keys
ssh-keygen -R TARGET-PRIVATE-IP
```

### 2. Ansible Installation Issues

#### Issue: "ansible: command not found"

```bash
# Error message:
# bash: ansible: command not found
```

**Solutions:**

```bash
# Solution A: Install via pip3 (recommended)
sudo yum install -y python3 python3-pip
pip3 install ansible
echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# Solution B: Install via yum (alternative)
sudo yum install -y epel-release
sudo yum install -y ansible

# Solution C: Verify installation
which ansible
ansible --version
```

#### Issue: "No module named 'ansible'"

```bash
# Error when using wrong Python version
```

**Solutions:**

```bash
# Solution A: Use correct Python version
python3 -m pip install ansible

# Solution B: Set Python interpreter in inventory
# Add to [all:vars]: ansible_python_interpreter=/usr/bin/python3

# Solution C: Check Python versions
python --version
python3 --version
which python3
```

### 3. Inventory Configuration Problems

#### Issue: "Could not match supplied host pattern"

```bash
# Error message:
# ERROR! Could not match supplied host pattern, ignoring: webservers
```

**Solutions:**

```bash
# Solution A: Check inventory file syntax
ansible-inventory -i inventory/hosts.ini --list

# Solution B: Verify group names match
cat inventory/hosts.ini | grep -E "^\[.*\]"

# Solution C: Test with explicit host
ansible -i inventory/hosts.ini target-server -m ping

# Solution D: Use different inventory format
# Try: ansible-playbook -i "TARGET-IP," playbook.yml
```

#### Issue: "Failed to parse inventory"

```bash
# Syntax errors in inventory file
```

**Solutions:**

```bash
# Solution A: Validate INI syntax
python3 -c "import configparser; c=configparser.ConfigParser(); c.read('inventory/hosts.ini')"

# Solution B: Check for common syntax errors
# - No spaces around = in host definitions
# - Proper section headers [groupname]
# - No duplicate host names

# Solution C: Use minimal inventory for testing
echo "target-server ansible_host=TARGET-IP ansible_user=ec2-user" > test-inventory.ini
```

### 4. Playbook Execution Errors

#### Issue: "Package conflicts" (curl-minimal error)

```bash
# Error message shows curl package conflicts
```

**Solutions:**

```bash
# Solution A: Use simple playbook (without curl)
# Use the simple-web-setup.yml playbook instead

# Solution B: Remove curl from package list
# Edit playbook and remove curl from package installation

# Solution C: Handle curl separately
- name: Install curl properly
  shell: |
    sudo yum remove -y curl-minimal
    sudo yum install -y curl
  ignore_errors: yes
```

#### Issue: "Service failed to start"

```bash
# Error message:
# Job for httpd.service failed because the control process exited
```

**Solutions:**

```bash
# Solution A: Check service status on target
ansible -i inventory/hosts.ini webservers -a "systemctl status httpd"

# Solution B: Check Apache configuration
ansible -i inventory/hosts.ini webservers -a "httpd -t"

# Solution C: Check port availability
ansible -i inventory/hosts.ini webservers -a "netstat -tlnp | grep :80"

# Solution D: Start service manually
ansible -i inventory/hosts.ini webservers -a "systemctl start httpd"
```

### 5. Network and Security Issues

#### Issue: "Website not accessible"

```bash
# Can't access http://PUBLIC-IP from browser
```

**Solutions:**

```bash
# Solution A: Check AWS Security Group
# 1. Go to AWS Console > EC2 > Security Groups
# 2. Find your instance's security group
# 3. Add Inbound Rule: HTTP (80) from 0.0.0.0/0

# Solution B: Verify service is running
ansible -i inventory/hosts.ini webservers -a "systemctl is-active httpd"

# Solution C: Check if port 80 is listening
ansible -i inventory/hosts.ini webservers -a "ss -tlnp | grep :80"

# Solution D: Test from control node
ansible -i inventory/hosts.ini webservers -a "curl -I http://localhost"
```

#### Issue: "Connection timeout"

```bash
# SSH connection times out
```

**Solutions:**

```bash
# Solution A: Check AWS Security Group
# Ensure SSH (22) is open from your IP

# Solution B: Verify instance is running
# Check AWS Console - instance should be "running"

# Solution C: Check VPC and subnet routing
# Ensure route table has internet gateway

# Solution D: Try from different network
# Your ISP might block certain ports
```

### 6. File and Permission Issues

#### Issue: "Permission denied" during file operations

```bash
# Error when creating files in web directory
```

**Solutions:**

```bash
# Solution A: Use 'become: yes' in playbook
# This is already included in the playbook

# Solution B: Check directory permissions
ansible -i inventory/hosts.ini webservers -a "ls -la /var/www/html"

# Solution C: Set proper ownership
- name: Set web directory ownership
  file:
    path: /var/www/html
    owner: apache
    group: apache
    mode: '0755'
    recurse: yes
```

## 🔍 Advanced Debugging Techniques

### Enable Verbose Output

```bash
# Level 1: Basic info
ansible-playbook -i inventory/hosts.ini playbook.yml -v

# Level 2: More details
ansible-playbook -i inventory/hosts.ini playbook.yml -vv

# Level 3: Connection debugging
ansible-playbook -i inventory/hosts.ini playbook.yml -vvv

# Level 4: Everything (very verbose)
ansible-playbook -i inventory/hosts.ini playbook.yml -vvvv
```

### Check Ansible Configuration

```bash
# Display current configuration
ansible-config dump

# Check which config file is used
ansible-config view

# List all config sources
ansible-config list
```

### Test Individual Tasks

```bash
# Run specific tags only
ansible-playbook -i inventory/hosts.ini playbook.yml --tags="packages"

# Skip specific tags
ansible-playbook -i inventory/hosts.ini playbook.yml --skip-tags="updates"

# Start from specific task
ansible-playbook -i inventory/hosts.ini playbook.yml --start-at-task="Install packages"

# Check mode (dry run)
ansible-playbook -i inventory/hosts.ini playbook.yml --check
```

### Gather System Information

```bash
# Collect all facts about target systems
ansible -i inventory/hosts.ini webservers -m setup

# Get specific facts
ansible -i inventory/hosts.ini webservers -m setup -a "filter=ansible_os_family"

# Check disk space
ansible -i inventory/hosts.ini webservers -a "df -h"

# Check memory usage
ansible -i inventory/hosts.ini webservers -a "free -m"

# Check running processes
ansible -i inventory/hosts.ini webservers -a "ps aux | head -20"
```

## 📊 Health Check Commands

### Pre-execution Checks

```bash
# 1. Verify Ansible can reach targets
ansible -i inventory/hosts.ini all -m ping

# 2. Check sudo privileges
ansible -i inventory/hosts.ini all -b -a "whoami"

# 3. Verify Python installation
ansible -i inventory/hosts.ini all -a "python3 --version"

# 4. Check available disk space
ansible -i inventory/hosts.ini all -a "df -h /"

# 5. Verify internet connectivity
ansible -i inventory/hosts.ini all -a "ping -c 1 google.com"
```

### Post-execution Verification

```bash
# 1. Check if Apache is running
ansible -i inventory/hosts.ini webservers -a "systemctl is-active httpd"

# 2. Verify Apache is enabled
ansible -i inventory/hosts.ini webservers -a "systemctl is-enabled httpd"

# 3. Check if port 80 is listening
ansible -i inventory/hosts.ini webservers -a "netstat -tlnp | grep :80"

# 4. Test web server response
ansible -i inventory/hosts.ini webservers -a "curl -s -o /dev/null -w '%{http_code}' http://localhost"

# 5. Check website file exists
ansible -i inventory/hosts.ini webservers -a "ls -la /var/www/html/index.html"
```

## 🛠️ Emergency Recovery Procedures

### If Everything Goes Wrong

```bash
# 1. Stop and restart target instance
# AWS Console > EC2 > Select instance > Actions > Instance State > Reboot

# 2. Reset SSH connection
ssh-keygen -R TARGET-PRIVATE-IP
rm ~/.ssh/known_hosts

# 3. Clean and reinstall Ansible
pip3 uninstall ansible
pip3 install ansible

# 4. Start with minimal test
echo "target ansible_host=TARGET-IP ansible_user=ec2-user" > minimal.ini
ansible -i minimal.ini target -m ping

# 5. Use AWS Systems Manager Session Manager as backup
# If SSH fails completely, use SSM in AWS Console
```

### Clean Slate Recovery

```bash
# If you need to start completely fresh:

# 1. Terminate current instances
# 2. Launch new instances with same configuration
# 3. Download fresh key pair
# 4. Update inventory with new IPs
# 5. Start setup process again
```

## 📞 Getting Help

### When to Seek Additional Help

1. **Hardware issues**: Instance won't start or constantly crashes
2. **Network issues**: Can't SSH from anywhere, not just Ansible
3. **AWS billing issues**: Unexpected charges or service limits
4. **Security concerns**: Potential unauthorized access

### Resources for Further Help

- **AWS Support**: For infrastructure and service issues
- **Ansible Documentation**: https://docs.ansible.com/
- **Community Forums**: Reddit r/ansible, Stack Overflow
- **Official Ansible GitHub**: For bug reports and feature requests

### Creating Effective Support Requests

When asking for help, include:

1. **Error message** (complete, not truncated)
2. **Command executed** that caused the error
3. **Environment details** (OS, Ansible version, AWS region)
4. **What you've already tried** from this troubleshooting guide
5. **Expected vs actual behavior**

## 🎯 Prevention Tips

### Best Practices to Avoid Issues

1. **Always test with minimal examples first**
2. **Use version control** for your playbooks and inventory
3. **Document your changes** and configurations
4. **Regular backups** of working configurations
5. **Monitor AWS costs** and resource usage
6. **Keep Ansible and packages updated**
7. **Test in development** before running in production

### Monitoring and Alerting

```bash
# Set up basic monitoring
# Create a monitoring script for regular health checks

#!/bin/bash
# health-check.sh
ansible -i inventory/hosts.ini all -m ping > /tmp/ansible-health.log 2>&1
if [ $? -ne 0 ]; then
    echo "Ansible connectivity issue detected" | mail -s "Ansible Alert" your-email@example.com
fi
```

Remember: Most issues are configuration-related and can be resolved by carefully checking syntax, permissions, and network settings. Take your time and work through problems systematically! 🚀
