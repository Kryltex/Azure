## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

<figure><img src=Diagrams/Azure.png><figcaption></figcaption></figure>

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select 
portions of the YAML file may be used to install only certain pieces of it, such as Filebeat.

### Playbook 1: my-playbook.yml
```
---
  - name: Config Web VM with Docker
    hosts: webservers
    become: true
    tasks:

    - name: Install docker.io
      apt:
        force_apt_get: yes
        update_cache: yes
        name: docker.io
        state: present

    - name: Install pip3
      apt:
        name: python3-pip
        state: present

    - name: Install Python Docker Module
      pip:
        name: docker
        state: present

    - name: Download and launch a docker web container
      docker_container:
        name: dvwa
        image: cyberxsecurity/dvwa
        state: started
        restart_policy: always
        published_ports: 80:80

    - name: Enable docker service
      systemd:
        name: docker
        enabled: yes
```

### Playbook 2: install-elk.yml
```
---
  - name: Configure Elk VM with Docker
    hosts: ELK
    remote_user: azureuser
    become: true
    tasks:

    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module (It will default to pip3)
    - name: Install Docker module
      pip:
        name: docker
        state: present

      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: 262144
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        # Please list the ports that ELK runs on
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
```

### Playbook 3: filebeat-playbook.yml
```
---
  - name: installing and launching filebeat
    hosts: webservers
    become: yes
    tasks:

    - name: download filebeat deb
      command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb

    - name: install filebeat deb
      command: dpkg -i filebeat-7.6.1-amd64.deb

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

    - name: enable service filebeat on boot
      systemd:
        name: filebeat
        enabled: yes
```

### Playbook 4: metricbeat-playbook.yml
```
---
  - name: Install metric beat
    hosts: webservers
    become: true
    tasks:
      # Use command module
    - name: Download metricbeat
      command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb

      # Use command module
    - name: install metricbeat
      command: dpkg -i metricbeat-7.6.1-amd64.deb

      # Use copy module
    - name: drop in metricbeat config
      copy:
        src: /etc/ansible/files/metricbeat-config.yml
        dest: /etc/metricbeat/metricbeat.yml

      # Use command module
    - name: enable and configure docker module for metric beat
      command: metricbeat modules enable docker

      # Use command module
    - name: setup metric beat
      command: metricbeat setup

      # Use command module
    - name: start metric beat
      command: service metricbeat start

      # Use systemd module
    - name: enable service metricbeat on boot
      systemd:
        name: metricbeat
        enabled: yes
```

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build

### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.
* As computing moves more to the cloud, Load Balancing plays an important security role. A Load Balacer can defend an organization against distributed denial-of-service (DDoS) attacks with its off-loading function.
* The Jump Box minimises the attack surface by utilizing a remote connections to the virtual network through a single virtual machine.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the configuration and system files.
* Filebeat is a lightweight way to forword and centralize logs
* Metricbeat is a lightweight way to send system and service statistics

The configuration details of each machine may be found below.

| Name      | Function | IP Address | Operating System |
|-----------|----------|------------|------------------|
| Jump Box  | Gateway  | 10.0.0.5   | Linux            |
| Web-1     | DVWA     | 10.0.0.6   | Linux            |
| Web-2     | DVWA     | 10.0.0.7   | Linux            |
| Elk-Stack | ELK      | 10.1.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
* 20.124.1.248

Machines within the network can only be accessed by the Jump Box.
* There is a policy in place that only allows the Jump Box access to the ELK virtual machine. The Jump Box's IP address is 10.0.0.5

A summary of the access policies in place can be found in the table below.

| Name      | Publicly Accessible | Allowed IP Addresses |
|-----------|---------------------|----------------------|
| Jump Box  | No                  | 20.124.1.248         |
| Web-1     | No                  | 20.124.1.248         |
| Web-2     | No                  | 20.124.1.248         |
| Elk-Stack | No                  | 20.124.1.248         |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
* There are many advantages of automating configurations with Ansible, Playbooks are written in YAML and it's easy to learn. Rapid configuration and deployment of virtual machines ensure security measures can be scripted while enabling scaling by deployment to more or fewer machines as required to meet demand.

The playbook implements the following tasks:
* Install Docker
* Install Python
* Install Docker Module
* Increase virtual memory
* Use more memory
* Download and launch a docker elk container
* Enable Docker on boot

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

<figure><img src=Diagrams/Docker-ps.PNG><figcaption></figcaption></figure>

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
* Web-1: 10.0.0.6
* Web-2: 10.0.0.7

We have installed the following Beats on these machines:
* Filebeat
* Metricbeat

These Beats allow us to collect the following information from each machine:
- _TODO: In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the playbook file to the Ansible Docker Container.
- Update the hosts file `/etc/ansible/hosts` to include the following
<figure><img src=Diagrams/Host_file.PNG><figcaption></figcaption></figure>
- Run the playbook, and navigate to ____ to check that the installation worked as expected.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
- _Which URL do you navigate to in order to check that the ELK server is running?

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
