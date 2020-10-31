## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

(Images/Azure_arch.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the filebeat-playbook.yml file may be used to install only certain pieces of it, such as Filebeat.

---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

  - name: install filebeat deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: service filebeat start

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting inboud access to the network.
What aspect of security do load balancers protect? What is the advantage of a jump box?_

It protects availability by using fult tolerance so if one server goes down the traffic can be reassigned to other server.
It protects integrity by restricting access so the information hosted on the webservers are not changed.
By using a jumbox we can deploy multiple containers from docker and connect remotely to the internal network

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the filesystem of the VMs and system metrics.

What does Filebeat watch for?

It watches for any event ocurrance on the webservers

What does Metricbeat record?

It records resource consumption such as number of servers, CPU, memory, disk usage, inbound and outbound traffic

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.0.0.4   | Linux            |
| Web_A    |Web server| 10.0.0.5   | Linux            |
| Web_B    |Web server| 10.0.0.6   | Linux            |
| ELK      |Monitoring| 10.1.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses: 98.15.50.59

Machines within the network can only be accessed by the Jump box.

Which machine did you allow to access your ELK VM? What was its IP address?_

The jumpbox has ssh access to the ELK-VM with IP: 10.0.0.4

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 10.0.0.1-254         |
|    ELK   | No                  | 10.0.0.1-254         |
| Web_A    | No                  | 10.0.0.1-254         |
| Web_B    | No                  | 10.0.0.1-254         |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
What is the main advantage of automating configuration with Ansible?

It is scalable in a big environment, it is fast and less prone to error since we only have to create a config file and deploy it in each container.

The playbook implements the following tasks:

Download image
Install package
copy files on the container
enable configuration of the service
start service

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

(Images/docker_ps.png)

The playbook elk-install.yml:

---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: elkuser
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present
      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present
      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present
      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes
      # Use docker_container module
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


### Target Machines & Beats
This ELK server is configured to monitor the following machines:

| Web_A | 10.0.0.5 |
| Web_B | 10.0.0.6 |

We have installed the following Beats on these machines:

| Server | Service  | Service    |
|--------|----------|------------|
| Web_A  | filebeat | metricbeat |
| Web_B  | filebeat | metricbeat |

These Beats allow us to collect the following information from each machine:

Filebeat collects syslog for each occurrence on the server
Metricbeat collects performance metrics such as CPU, memory, storage, network traffic

e.g

For instance the syslog will keep track what applications or services have been installed in any period of time and by whom.

playbook metricbeat-playbook.yml

---
- name: installing and launching metricbeat
  hosts: webservers
  become: yes
  tasks:

  - name: download metricbeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

  - name: install metricbeat deb
    command: dpkg -i metricbeat-7.4.0-amd64.deb

  - name: drop in metricbeat.yml
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

  - name: enable and configure system module
    command: metricbeat modules enable system

  - name: setup metricbeat
    command: metricbeat setup

  - name: start metricbeat service
    command: service metricbeat start

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the playbook file to the ansible control node
- Update the hosts file to include the webservers, elk server and IP information
- Run the playbook, and navigate to http://52.252.56.93:5601/app/kibana to check that the installation worked as expected.


- Which file is the playbook? Where do you copy it?

 This is the playbook located in this path   src: /etc/ansible/files/filebeat-config.yml
 It is copied to dest: /etc/filebeat/filebeat.yml in the dwva container
      
- Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?

Update the ansible.cfg file with remote user and the hosts file with the target information.
To specify which machine to install elk server add:

[elk]
10.1.0.4 ansible_python_interpreter=/usr/bin/python3

then run:

$ansible-playbook elk-install.yml

To install filebeat you create a playbook called filebeat-playbook.yml then run:

$ansible-playbook filebeat-playbook.yml

- Which URL do you navigate to in order to check that the ELK server is running?

http://52.252.56.93:5601/app/kibana

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._

git clone https://github.com/yourusername/project-1.git
# Move Playbooks and hosts file Into `/etc/ansible`
$ cp project-1/playbooks/* .
$ cp project-1/files/* ./files
