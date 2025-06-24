# 🐍 DevSecOps pipeline for Python project 🔐

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Build Status](https://img.shields.io/jenkins/build?job=PythonSecurityPipeline&server=https%3A%2F%2Fjenkins.example.com)](https://jenkins.example.com/job/PythonSecurityPipeline/)

🚀 A Jenkins end-to-end DevSecOps pipeline for a Python web application, hosted on AWS Ubuntu 18.04 LTS.

![DevSecOps Pipeline Workflow Diagram](https://user-images.githubusercontent.com/11514346/71473164-e57a5500-27cd-11ea-97cb-3c25f0266407.JPG "DevSecOps pipeline workflow diagram")
<img alt="Architecture Diagram of Jenkins, Docker, Selenium, and AWS Resources" src="https://user-images.githubusercontent.com/11514346/71579758-effe5c80-2af5-11ea-97ae-dd6c91b02312.PNG">

*⚠️ Disclaimer: This project is for demonstration purposes with surface level checks only. Do not use it as-is for production.*

## 🔒 Security Considerations

- This demo pipeline is not hardened for production. Review all components thoroughly before deploying.
- Always rotate credentials and avoid embedding secrets in code or images.
- Use a bastion host or VPN for secure SSH access; restrict Security Group rules accordingly.
- Regularly update OS packages and Docker images to address vulnerabilities.
- Keep Jenkins plugins up-to-date. Note: plugins included here date from 2019–2020 and may require upgrades.

## 📋 Table of Contents

- [📋 Prerequisites](#prerequisites)  
- [⚙️ Installation Steps](#installation-steps)  
- [🛠️ Technology Stack](#technology-stack)  
- [🌍 Environment Variables](#environment-variables)  
- [🔍 Security Tools Overview](#security-tools-overview)  
- [🧹 Cleanup and Cost Management](#cleanup-and-cost-management)  
- [🔧 Troubleshooting](#troubleshooting)  
- [🤝 Contributing](#contributing)  
- [👥 Authors](#authors)  
- [📄 License](#license)  

## 📋 Prerequisites

Before you begin, ensure you have the following:

### 💻 System Requirements
- Operating System: Ubuntu 18.04 LTS or later  
- Hardware: Minimum t2.medium EC2 instance (2 vCPU, 4 GB RAM) for Jenkins + Selenium + Docker workload  
- Disk: At least 20 GB free space  

### 📦 Required Software
- Docker Engine version >= 19.03  
- Docker Compose version >= 1.25  
- AWS CLI version 2.x (configured with `aws configure`)  
- OpenSSL (for generating Jenkins password)  
- Java 8 or higher (for Jenkins)  

### ☁️ AWS Account Setup
- Active AWS account with billing enabled  
- Region: eu-west-2 (modify as needed in `jenkins_home/createAwsEc2.yml`)  
- VPC and Subnet created (see AWS Configuration below)  

### 🔑 IAM Permissions
Create or attach an IAM policy with at least the following permissions:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:RunInstances",
        "ec2:TerminateInstances",
        "ec2:DescribeInstances",
        "ec2:CreateTags",
        "ec2:DeleteKeyPair",
        "ec2:CreateKeyPair",
        "ec2:AuthorizeSecurityGroupIngress",
        "ec2:RevokeSecurityGroupIngress",
        "ec2:DescribeKeyPairs",
        "ec2:DescribeSecurityGroups",
        "ec2:DescribeVpcs",
        "ec2:DescribeSubnets"
      ],
      "Resource": "*"
    }
  ]
}
```
Attach this policy to the IAM role assigned to your Ubuntu EC2 host.

---

## ⚙️ Installation Steps

1. **📥 Clone the repository** to your Ubuntu server (t2.medium recommended):
   ```bash
   git clone https://github.com/bshongwe/python-security-pipeline.git
   cd python-security-pipeline
   ```

2. **☁️ AWS Configuration**  
   Edit `jenkins_home/createAwsEc2.yml` and `setup-ubuntu.sh` to match your AWS environment:

   - **🌐 VPC and Subnet**  
     - Create a VPC (e.g., `10.0.0.0/16`) or use an existing one.  
     - Create at least one public subnet (e.g., `10.0.1.0/24`).  
     - Note the `vpc_subnet_id` (e.g., `subnet-02a17e56e6827124a`) and set it in `createAwsEc2.yml`.  

   - **🛡️ Security Group Rules**  
     - SSH (TCP/22) from your IP  
     - HTTP (TCP/80) for WAF testing  
     - Jenkins (TCP/8080) from your IP range  
     - Application (TCP/10007) *optional* from your IP
     
     Example ruleset:
     | Protocol | Port Range | Source           | Purpose                 |
     |----------|------------|------------------|-------------------------|
     | TCP      | 22         | Your IP/32       | SSH access              |
     | TCP      | 80         | 0.0.0.0/0 (opt)  | HTTP for modsecurity    |
     | TCP      | 8080       | Your IP/32       | Jenkins UI              |
     | TCP      | 10007      | Your IP/32 (opt) | App traffic for DAST    |

   - **👤 IAM Role**  
     - Attach the above IAM policy to the EC2 instance profile for automated provisioning.

   - **💰 Cost Warning**  
     - t2.medium runs approximately $0.0464 per hour in eu-west-2.  
     - Terminate resources promptly to avoid unexpected charges (see Cleanup section).

3. **🚀 Run the setup script** to bootstrap Jenkins, Docker, and Selenium:
   ```bash
   sudo sh setup-ubuntu.sh
   ```

4. **🔥 Firewall and Access**  
   Ensure your security group allows inbound traffic on port **8080**. Then visit:
   ```
   http://YOUR_EC2_PUBLIC_HOSTNAME:8080/
   ```

5. **🔐 Login to Jenkins**  
   - Username: `myjenkins`  
   - Password: see the `Jenkins_PW` printed at the end of the setup script  
   - **Change this password immediately** via `Manage Jenkins` → `Configure Global Security`.

6. **▶️ Trigger the Pipeline**  
   From the Jenkins dashboard, select the "PythonSecurityPipeline" project and click **Build Now**.

---

## 🛠️ Technology Stack

- **Jenkins LTS** 🏗️ for CI/CD orchestration (with plugins from 2019–2020; consider upgrading plugins)  
- **Docker** 🐳 for containerization of Jenkins and Selenium  
- **Docker Compose** 📦 to define multi-container setup  
- **Ansible** 🎭 for automated EC2 provisioning and configuration  
- **Security tools** 🔒:  
  - **TruffleHog** 🐷: Scans git history for secrets and credentials  
  - **Safety** ⚡: Dependency vulnerability scanner (`safety scan` recommended; `safety check` is deprecated)  
  - **Bandit** 🔍: Static analysis for Python security issues  
  - **Nikto** 💻: Web application vulnerability scanner  
  - **Lynis** 🛡️: Host system security auditing  
  - **Selenium** 🤖: Browser automation for authenticated DAST  
- **AWS Services** ☁️:  
  - **EC2** 🖥️ for Jenkins and test instance  
  - **VPC & Subnets** 🌐 for network isolation  
  - **Security Groups** 🛡️ for firewall rules

---

## 🌍 Environment Variables

The following environment variables are set in `setup-ubuntu.sh` and passed into the Jenkins container:

- `Jenkins_PW` 🔑: Randomly generated Jenkins administrator password  
- `JAVA_OPTS` ☕: `-Djenkins.install.runSetupWizard=false` to skip the initial Jenkins wizard  
- `JenkinsPublicHostname` 🌐: Public hostname of the EC2 instance (from metadata)  
- `SeleniumPrivateIp` 🔗: Private IP of the EC2 instance for Selenium container discovery

---

## 🔍 Security Tools Overview

| Tool        | Purpose                                                            |
|-------------|--------------------------------------------------------------------|
| TruffleHog 🐷 | Detects high-entropy strings (potential secrets) in git history    |
| Safety ⚡    | Checks Python dependencies against known vulnerability database    |
| Bandit 🔍    | Performs static code analysis to find security issues in Python    |
| Nikto 💻     | Scans web servers for outdated components and known advisories     |
| Lynis 🛡️    | Audits the host operating system for security vulnerabilities      |
| Ansible 🎭   | Automates infrastructure provisioning and configuration management |

---

## 🧹 Cleanup and Cost Management

After testing, destroy all AWS resources to avoid charges:

1. **❌ Terminate test EC2 instances** using the Ansible playbook:
   ```bash
   ansible-playbook -i ~/ansible_hosts ~/killec2.yml
   ```
2. **🗑️ Remove Jenkins container and volumes**:
   ```bash
   docker-compose down --volumes
   ```
3. **📊 Monitor AWS Cost Explorer** to ensure no residual costs.

**📝 Resource Cleanup Checklist**  
- ✅ EC2 instances: terminated  
- ✅ Key pairs: removed (optional)  
- ✅ Security groups: reviewed/deleted  
- ✅ VPC/subnets: deleted if not reused  
- ✅ S3 buckets or EBS volumes: cleaned up

---

## 🔧 Troubleshooting

- **🐳 Docker installation issues**:  
  - Ensure `docker.io` and `docker-compose` are installed with compatible versions.  
  - Run `sudo usermod -aG docker $USER` and re-login.

- **☁️ AWS connectivity problems**:  
  - Verify IAM role and policy attachments.  
  - Check VPC, subnet, and security group configurations.

- **🏗️ Jenkins startup errors**:  
  - Inspect logs: `docker logs jenkins-master`  
  - Confirm `JAVA_OPTS` is set correctly.

- **🔥 Port access and firewall**:  
  - Validate inbound rules for ports 22, 80, 8080, 10007 in Security Group.  
  - Use `curl` or `telnet` to test port reachability.

- **🔑 EC2 SSH issues**:  
  - Confirm the correct keypair is in use: `~/.ssh/psp_ansible_key.pem`.  
  - Check permissions: `chmod 400 ~/.ssh/psp_ansible_key.pem`.

---

## 🤝 Contributing

We welcome contributions! Please follow these steps:

1. **🐛 Report Issues**  
   - Use GitHub Issues to report bugs or request features.

2. **🍴 Fork & Clone**  
   ```bash
   git clone https://github.com/yourusername/python-security-pipeline.git
   cd python-security-pipeline
   ```

3. **🌿 Branch & Commit**  
   - Create a feature branch: `git checkout -b feature/my-change`  
   - Adhere to PEP8 for Python code.  
   - Write clear commit messages and include issue references.

4. **📤 Pull Request**  
   - Push your branch and open a PR against `main`.  
   - Include a detailed description of changes and any testing performed.

---

## 👥 Authors

* **Ernest Bhekizwe Shongwe** - [bshongwe](https://github.com/bshongwe)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.