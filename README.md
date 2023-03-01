# Ansible
Ansible - AWS EC2 Instances
Demo Ansible Playbook for creating, stopping, terminating the EC2 instance

Create a custom Security Group in AWS to allow ports 22 (SSH), 80 (HTTP) and ICMP.
Create/Stop/Terminate the EC2 instance - Using Ansible EC2 module
Wait for the SSH is up on Instance.
Installation
Firstly, we need to install and configure.

1. Created key_pair with name "AWS-Ansible"

2. Create Security Group in AWS to allow ports 22/SSH, 80/HTTP and ICMP.
---
- hosts: server
  tasks:
    - name: Setting up the Security Group for new instance
      ec2_group:
          name: Ansible_Security_Group_AWS
          description: Allowing Traffic on port 22 and 80
          region: ap-southeast-1
          rules:
           - proto: tcp
             from_port: 80
             to_port: 80
             cidr_ip: 0.0.0.0/0

           - proto: tcp
             from_port: 22
             to_port: 22
             cidr_ip: 0.0.0.0/0

           - proto: icmp
             from_port: -1
             to_port: -1
             cidr_ip: 0.0.0.0/0
          rules_egress:
           - proto: all
             cidr_ip: 0.0.0.0/0
          vpc_id: vpc-d8826ebd

    - name: Provision EC2 instance
      ec2:
         key_name: AWS-Ansible
         region: ap-southeast-1
         instance_type: t2.micro
         image: ami-67a6e60456e2
         wait: yes
         wait_timeout: 500
         count: 1
         instance_tags:
            Name: AWS-Ansible
         volumes:
            - device_name: /dev/xvda
              volume_type: gp2
              volume_size: 8
         monitoring: yes
         vpc_subnet_id: subnet-18e7fdae5
         assign_public_ip: yes
         group: Ansible_Security_Group_AWS
      register: ec2

    - name: Wait for SSH to come up
      wait_for:
          host: "{{ item.public_dns_name }}"
          port: 22
          delay: 60
          timeout: 320
          state: started
      with_items: "{{ ec2.instances }}"
      Execution Ansible playbooks to create EC2 instance
