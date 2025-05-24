![jenkins2](https://github.com/user-attachments/assets/a786f682-1879-48af-ac89-d0241214d479)

# **Jenkins Setup Using Ansible Role**
| Created        | Last updated      | Version         | author|  Internal Reviewer | L0 | L1 | L2|
|----------------|----------------|-----------------|-----------------|-----|------|----|----|
| 2025-05-23  | 2025-05-24   |     Version 1         |  Mohamed Tharik |Priyanshu|Khushi|Mukul Joshi |Piyush Upadhyay|

## Table of Contents 
1. [Introduction](#introduction)  
2. [Prerequisites](#prerequisites)  
3. [Jenkins Setup Using Ansible Roles Steps](#jenkins-setup-using-ansible-roles-steps)
   - [1. Create Ansible Role Structure](#1-create-ansible-role-structure)
   - [2. Define Role Files](#2-define-role-files)
   - [3. Create Ansible Playbook in Project Directory](#3-create-ansible-playbook-in-project-directory)
   - [4. Run the Playbook](#4-run-the-playbook)
   - [5. Check Whether the Jenkins is Downloaded](#5-check-whether-the-jenkins-is-downloaded)
4. [Best Practices for Jenkins Setup with Ansible](#best-practices-for-jenkins-setup-with-ansible)  
5. [Conclusion](#conclusion)  
6. [Contact Information](#contact-information)  
7. [References](#references)  

## Introduction 
Jenkins is an open-source automation server widely used for building, testing, and deploying applications. Automating the setup of Jenkins using Ansible ensures consistency, repeatability, and efficiency—especially in infrastructure as code (IaC) environments. This guide provides a structured approach to install and configure Jenkins via an Ansible role.

## Prerequisites

| Requirement                 | Description                                                                 |
|----------------------------|-----------------------------------------------------------------------------|
| Linux Server               | Target machine should be Ubuntu 20.04+ or a compatible Debian-based system.|
| Ansible                    | Installed on the control node (your local system or Ansible server).        |
| SSH Access                 | Ensure passwordless SSH access to the managed node(s).                      |
| Sudo Privileges            | Required for installing packages and managing services on the target host. |
| Java (OpenJDK 17+)         | Needed to run Jenkins. It will be installed via Ansible.                    |
| Internet Connectivity      | Required on the target machine to fetch Jenkins and Java packages.         |
| Inventory Configuration    | Target host must be defined in Ansible inventory under `jenkins_servers`.  |

## Jenkins Setup Using Ansible Roles Steps
### 1. Create Ansible Role Structure
`jenkins_setup_projects/roles` in this directory, 
```bash
ansible-galaxy init jenkins_setup
```
![jenkinsrole4](https://github.com/user-attachments/assets/479ebb79-844a-4043-9456-20f2ab5084a0)
![jenkinsrole](https://github.com/user-attachments/assets/8b6902ff-eb08-4f7d-af8b-e4682062496d)

### 2. Define Role Files
`roles/jenkins_setup/tasks/main.yml`
```bash
---
- name: Update apt cache
  apt:
    update_cache: yes

- name: Install Java
  apt:
    name: openjdk-17-jdk
    state: present
  become: yes

- name: Download Jenkins GPG key
  get_url:
    url: https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    dest: /usr/share/keyrings/jenkins-keyring.asc
    mode: '0644'
  become: yes

- name: Add Jenkins repository using Jinja2 template
  template:
    src: jenkins.repo.j2
    dest: /etc/apt/sources.list.d/jenkins.list
  become: yes

- name: Update apt cache
  apt:
    update_cache: yes
  become: yes

- name: Install Jenkins
  apt:
    name: jenkins
    state: present
  become: yes

- name: Start and enable Jenkins service
  systemd:
    name: jenkins
    state: started
    enabled: yes
  become: yes

- name: Ensure Jenkins system user exists
  user:
    name: jenkins
    shell: /bin/bash
    state: present
  become: yes
```
`roles/jenkins_setup/templates/jenkins.repo.j2`
```bash
deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/
```

### 3. Create Ansible Playbook in Project Directory
`jenkins_setup_projects/jenkins-setup.yml`
```bash
- hosts: webservers
  become: yes
  roles:
    - jenkins_setup
```

### 4. Run the Playbook
```bash
ansible-playbook -i inventory.ini jenkins-setup.yml
```
![jenkinsrole2](https://github.com/user-attachments/assets/725e4772-b3d5-493d-bb4a-bbc2610d974f)

### 5. Check Whether the Jenkins is Downloaded 
```bash
sudo systemctl status jenkins
```
![jenkinsrole3](https://github.com/user-attachments/assets/aa04035d-dc46-424d-aeb3-7a17c7a3c3ad)

## Best Practices for Jenkins Setup with Ansible

| Best Practice                           | Why It Matters                                                                 |
|----------------------------------------|---------------------------------------------------------------------------------|
| **Use Role-Based Structure**           | Promotes reusability, modularity, and clean separation of logic.               |
| **Keep Variables in `defaults/` or `vars/`** | Centralizes configuration, making the role easy to customize and reuse.         |
| **Use Templates for Config Files**     | Enables dynamic configuration based on inventory or variable values.           |
| **Avoid Hardcoded Values**             | Enhances flexibility across environments (dev, stage, prod).                   |
| **Enable `become: yes` Carefully**     | Ensure sudo access is only granted where absolutely necessary.                 |
| **Run Idempotent Tasks**               | Ansible tasks should be repeatable without changing state unnecessarily.       |
| **Tag Your Tasks**                     | Allows running specific parts of the playbook using `--tags` for flexibility.  |

## Conclusion
Using Ansible roles to install Jenkins simplifies the process and brings automation, consistency, and version control to your CI/CD infrastructure. This method is scalable and ideal for enterprise-level deployments where repeatable and automated provisioning is essential.

## Contact Information
| Name | Email address         |
|------|------------------------|
| Mohamed Tharik  | md.tharik.sanaatak@mygurukulam.co    |

## References

| Link | Description |
|------|-------------|
| [Jenkins Official Site](https://www.jenkins.io/) | Official homepage of Jenkins with downloads and documentation. |
| [Jenkins Debian Installation Guide](https://www.jenkins.io/doc/book/installing/linux/#debianubuntu) | Steps to install Jenkins on Debian/Ubuntu-based systems. |
| [Ansible Galaxy Roles Documentation](https://galaxy.ansible.com/docs/contributing/creating_role.html) | Guide on creating and structuring Ansible roles. |
| [OpenJDK Installation](https://openjdk.org/install/) | Official guide for downloading and installing OpenJDK. |
| [Ansible Official Documentation](https://docs.ansible.com/) | Comprehensive documentation for Ansible usage and modules. |

