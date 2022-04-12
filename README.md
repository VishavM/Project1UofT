# Project1UofT

## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![Homework 12 Cloud Security](https://user-images.githubusercontent.com/96074199/162986989-d8183451-9167-4673-9b1d-7d7067f96eed.PNG)




This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.
- Load balancers make sure network enviornment is available through distribution of all incoming     data to web-servers whereas Jump box help in administration of multiple systems with adding security layer between internal and external assets.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the system metrics and event logs.
- What does Filebeat watch for? Log directories.
- What does Metricbeat record? Information from system and services running on server while monitoring them constantly. 

The configuration details of each machine may be found below.


| Name             | Function   | IP Address | Operating System |
|------------------|------------|------------|------------------|
|JumpBoxProvisioner| Gateway    | 10.0.0.4   | Ubuntu-Linux     |
|RedTeam-Web1      | Webapp DVWA| 10.0.0.5   | Ubuntu-Linux     |
|RedTeam-Web2      | Webapp DVWA| 10.0.0.6   | Ubuntu-Linux     |
|Elk-ServerVM      | ELK-Server | 10.1.0.4   | Ubuntu-Linux     |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump-Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 104.43.170.40

Machines within the network can only be accessed by JumpBoxProvisioner.
- Which machine did you allow to access your ELK VM? JumpBoxProvisioner
- What was its IP address? 10.0.0.4(private IP)

A summary of the access policies in place can be found in the table below.

| Name              | Publicly Accessible | Allowed IP Addresses |
|-------------------|---------------------|----------------------|
|JumpBoxProvisioner |YES                  |104.43.170.40         |
|Elk-ServerVM       |YES                  |104.43.170.40         |
|RedTeam-Web1       |NO                   |10.0.0.5              |
|RedTeam-Web2       |NO                   |10.0.0.6              |
|LoadBalancer       |YES                  |OPEN                  |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because all service's running would be limited with streamlined updates, installation making all processess more consistent.

The playbook implements the following tasks:

- Installs docker.io, pip3, and the docker module.
   Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

   Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

   Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present

- Increases the virtual memory 
   Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144
    
- Sysctl module
   Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes

- Docker container for elk server
 Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

- 
The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.



### Target Machines & Beats
This ELK server is configured to monitor the following machines:
|RedTeam-Web1       10.0.0.5              |
|RedTeam-Web2       10.0.0.6              |

We have installed the following Beats on these machines:
- FileBeat
- MetricBeat

These Beats allow us to collect the following information from each machine:
- FileBeat helps to log data for local files, it works as an agent on servers while monitoring the log directories, tailing the file and forwarding them for indexing.
- MetricBeat collects the metric and statistics on our system such as CPU usuage, system health and several core health logs.

 ### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the configuration file to our Web VM's.
- Update the /etc/ansilbe/hosts file to include Webservers and Elkservers VMs.
- Run the playbook, and navigate to http://[104.43.170.40]:5601/app/kibana to check that the installation worked as expected.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it? Filebeat-configuration
- _Which file do you update to make Ansible run the playbook on a specific machine? Filebeat-config.yml
- How do I specify which machine to install the ELK server on versus which to install Filebeat on? Updating the hist files with Elkserver IP addresses and selecting the group to run on ansible.
- Which URL do you navigate to in order to check that the ELK server is running? http://[104.43.170.40]:5601/app/kibana#/home

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
- name: Installing and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:
     Use command module
  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

     Use command module
  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

     Use copy module
  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

     Use command module
  - name: Enable and Configure System Module
    command: filebeat modules enable system

     Use command module
  - name: Setup filebeat
    command: filebeat setup

     Use command module
  - name: Start filebeat service
    command: service filebeat start
- name: Install metric beat
  hosts: webservers
  become: true
  tasks:
     Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

     Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb

     Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

     Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

     Use command module
  - name: setup metric beat
    command: metricbeat setup

     Use command module
  - name: start metric beat
    command: service metricbeat start
