ansible-aws-automation

Complete Ansible automation for AWS infrastructure - Web server deployment with troubleshooting guide

🚀 Ansible AWS Infrastructure Automation

> Complete beginner-friendly guide to automate AWS EC2 instances using Ansible

![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)

📚 Table of Contents

- [🎯 Project Overview](-project-overview)
- [🛠️ Prerequisites](️-prerequisites)
- [🏗️ Architecture](️-architecture)
- [📋 Step-by-Step Implementation](-step-by-step-implementation)
- [📁 Project Structure](-project-structure)
- [🔧 Troubleshooting](-troubleshooting)
- [📖 Learning Outcomes](-learning-outcomes)
- [🎓 What's Next](-whats-next)

🎯 Project Overview

This project demonstrates Infrastructure as Code (IaC) using Ansible to automate AWS EC2 instances. You'll learn how to:

- Set up Ansible control and target machines on AWS
- Configure SSH key-based authentication
- Write Ansible playbooks for web server automation
- Implement DevOps best practices
- Troubleshoot common automation issues

Real-world application: This is exactly how DevOps engineers manage hundreds of servers in production environments.

🛠️ Prerequisites

What You Need to Know

- Basic Linux commands (cd, ls, nano, ssh)
- Basic AWS concepts (EC2, Security Groups, Key Pairs)
- No Ansible experience required - we'll learn together!

AWS Resources Required

- 2 EC2 instances (Amazon Linux 2023)
- 1 Key Pair for SSH access
- Security Groups configured properly
- Basic AWS knowledge (launching instances)

Cost Estimate

- ~$0.50-$2.00 per day using t2.micro instances
- Free tier eligible for new AWS accounts

🏗️ Architecture

```
┌─────────────────┐         ┌─────────────────┐
│  Control Node   │   SSH   │  Target Node    │
│  (Ansible)      │ ─────── │  (Web Server)   │
│                 │  Port   │                 │
│ • Ansible Core  │   22    │ • Apache HTTP   │
│ • Playbooks     │         │ • Automated     │
│ • Inventory     │         │   Setup         │
└─────────────────┘         └─────────────────┘
```

How it works:

1. Control Node contains Ansible and playbooks
2. Target Node receives automation commands
3. SSH connection enables secure automation
4. Playbooks define what to install and configure

📋 Step-by-Step Implementation

Phase 1: AWS Infrastructure Setup

1.1 Create EC2 Instances

Launch 2 instances with these specifications:

| Setting        | Value                      |
| -------------- | -------------------------- |
| AMI            | Amazon Linux 2023          |
| Instance Type  | t2.micro                   |
| Key Pair       | Your existing key pair     |
| Security Group | Allow SSH (22) + HTTP (80) |

Tag your instances:

- Control Node: `Name = ansible-control`
- Target Node: `Name = ansible-target`

  1.2 Configure Security Groups

```bash
 Inbound Rules Required
Port 22 (SSH)  - Source: Your IP
Port 80 (HTTP) - Source: 0.0.0.0/0 (for web access)
```

Phase 2: Control Node Setup

2.1 Connect to Control Node

```bash
 Connect via SSH
ssh -i your-key.pem ec2-user@CONTROL-PUBLIC-IP

 Update system
sudo yum update -y
```

2.2 Install Ansible

```bash
 Install Python and pip
sudo yum install -y python3 python3-pip

 Install Ansible
pip3 install ansible

 Verify installation
ansible --version
```

2.3 Configure SSH Access

```bash
 Copy your private key to control node
nano ~/.ssh/your-key.pem
 Paste your private key content here

 Set proper permissions
chmod 600 ~/.ssh/your-key.pem

 Test connection to target
ssh -i ~/.ssh/your-key.pem ec2-user@TARGET-PRIVATE-IP
```

Phase 3: Ansible Configuration

3.1 Create Project Directory

```bash
 Create organized project structure
mkdir ~/ansible-aws-automation
cd ~/ansible-aws-automation

 Create subdirectories
mkdir playbooks inventory group_vars
```

3.2 Configure Inventory

```bash
 Create inventory file
nano inventory/hosts.ini
```

Add this content:

```ini
[webservers]
target-server ansible_host=TARGET-PRIVATE-IP ansible_user=ec2-user ansible_ssh_private_key_file=~/.ssh/your-key.pem

[all:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

3.3 Test Connectivity

```bash
 Test Ansible connection
ansible -i inventory/hosts.ini webservers -m ping

 Expected output:
 target-server | SUCCESS => {
     "changed": false,
     "ping": "pong"
 }
```

Phase 4: Write Automation Playbook

4.1 Create Web Server Playbook

```bash
nano playbooks/web-server-setup.yml
```

Copy the complete playbook from `playbooks/web-server-setup.yml` in this repository.

4.2 Run the Automation

```bash
 Execute the playbook
ansible-playbook -i inventory/hosts.ini playbooks/web-server-setup.yml

 Watch the automation magic happen!
```

Phase 5: Verification

5.1 Check Web Server

```bash
 Test web server is running
ansible -i inventory/hosts.ini webservers -a "systemctl status httpd"

 Check what's listening on port 80
ansible -i inventory/hosts.ini webservers -a "netstat -tlnp | grep :80"
```

5.2 Access Your Website

1. Get target's public IP from AWS Console
2. Visit: `http://TARGET-PUBLIC-IP`
3. See your automated website! 🎉

📁 Project Structure

```
ansible-aws-automation/
├── README.md
├── inventory/
│   └── hosts.ini
├── playbooks/
│   ├── web-server-setup.yml
│   └── simple-web-setup.yml
├── group_vars/
│   └── all.yml
├── docs/
│   ├── troubleshooting.md
│   ├── aws-setup.md
│   └── ansible-basics.md
└── examples/
    ├── ad-hoc-commands.md
    └── advanced-playbooks/
```

🔧 Troubleshooting

Common Issues and Solutions

| Problem                   | Solution                                  |
| ------------------------- | ----------------------------------------- |
| SSH connection refused    | Check security groups allow port 22       |
| Permission denied         | Verify key file permissions (`chmod 600`) |
| Package conflicts         | Use simplified playbook without curl      |
| Website not loading       | Check security group allows port 80       |
| Ansible command not found | Verify pip3 installation and PATH         |

Detailed troubleshooting: See `docs/troubleshooting.md`

📖 Learning Outcomes

After completing this project, you'll understand:

✅ Core Concepts

- Infrastructure as Code (IaC) principles
- Ansible architecture and components
- SSH key-based authentication
- YAML syntax for configuration
- Inventory management for multiple servers

✅ Practical Skills

- AWS EC2 instance management
- Linux system administration
- Ansible playbook development
- Web server configuration
- DevOps automation workflows

✅ Industry Best Practices

- Version control for infrastructure code
- Idempotent operations (safe to run multiple times)
- Error handling and troubleshooting
- Documentation and code organization
- Security considerations for automation

🎓 What's Next?

Beginner Level

- [ ] Add more services (database, monitoring)
- [ ] Create multiple environments (dev, staging, prod)
- [ ] Implement Ansible Vault for secrets
- [ ] Add error handling and notifications

Intermediate Level

- [ ] Use Ansible Galaxy roles
- [ ] Implement CI/CD with GitHub Actions
- [ ] Add automated testing with Molecule
- [ ] Create dynamic inventories

Advanced Level

- [ ] Integrate with Terraform for infrastructure
- [ ] Implement rolling deployments
- [ ] Add monitoring and logging
- [ ] Create custom Ansible modules

🤝 Contributing

Feel free to:

- Report issues you encounter
- Suggest improvements to the documentation
- Share your variations of the playbooks
- Add new examples for different use cases

📜 License

This project is licensed under the MIT License - see the LICENSE file for details.

🙏 Acknowledgments

- Amazon Web Services for the cloud platform
- Red Hat Ansible for the automation framework
- DevOps community for best practices and inspiration

---

⭐ If this helped you learn Ansible and AWS automation, please star this repository!
