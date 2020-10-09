## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![ELK-Network-Diagram](Cloud-Project/ELK-Network-Diagram.jpg)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the Ansible file may be used to install only certain pieces of it, such as Filebeat.


This document contains the following details:

- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available , in addition to restricting inbound access to the network.

Load balancing ensures that work to process incoming traffic traffic will be shared by both vulnerable web servers. Access controls will ensure that only authorised users -namely ourselves- will be able to connect in the first place.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file system of the VMs on the network as well as to watch system metrics such as CPU usage and SSH logins amongst other things.

The configuration details of each machine may be found below.

| Name     | Function   | IP Address | Operating System |
| -------- | ---------- | ---------- | ---------------- |
| Jump Box | Gateway    | 10.1.0.4   | Linux            |
| Elk      | Monitoring | 10.0.0.4   | Linux            |
| DVWA 1   | Web Server | 10.1.0.12  | Linux            |
| DVWA 2   | Web Server | 10.1.0.13  | Linux            |
| DVWA 3   | Web Server | 10.1.0.6   | Linux            |

In addition to the above, Azure has provisioned a load balancer in front of all machines except for the jump box. The targeted zones used are in the following availability zones:

- **Availability Zone 1 ** : DVWA 1 + DVWA 2

- **Availability Zone 2** : ELK

  ****

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the jump-box  machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses: 52.187.(masked). Each public address will different according to your setup in Azure.  It is also important to note that Azure automatically assigned new public IP addresses to your Jumpbox VM each time it is restarted.  

Machines within the network can only be accessed by each other. The DVWA VMs send the traffic to the ELK server.

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
| -------- | ------------------- | -------------------- |
| Jump Box | Yes                 | 52.187.  (masked)    |
| Elk-VM   | Yes                 | 10.0.0.1-254         |
| Web- 1   | No                  | 10.0.0.1-254         |
| Web- 2   | No                  | 10.0.0.1-254         |
| Web- 3   | No                  | 10.0.0.1-254         |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because it minimises the possibility of errors in the process and makes it easier to deploy the ELK stack.

The playbook implements the following tasks:

- Installs docker and ansible on the ELK VM 
- Installs the YMAL Playbook on the ELK VM
- Downloads the image from the server.

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

**Note**: The following image link needs to be updated. Replace `docker_ps_output.png` with the name of your screenshot image file.  

![TODO: Update the path with the name of your screenshot of docker ps output](Images/docker_ps_output.png)

The playbook is below.

```yaml
---
- name: Configure Elk VM with Docker
  hosts: elkservers
  remote_user: azadmin
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
```





### Target Machines & Beats

This ELK server is configured to monitor the DWVA 1 and DVWA 2 VMs, at 10.1.0.12 and 10.1.0.13, respectively.

We have installed the following Beats on these machines:

- Filebeat 

These Beats allow us to collect the following information from each machine:

- Filebeat: Filebeat detects changes to the filesystem. Specifically, we use it to collect Apache logs.

The playbook below installs Filebeat on the target hosts. the playbook for installing Metricbeat  is not included, but looks identical - simply replace filebeat with metricbeat.



```yaml
---
- name: Install filebeat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download filebeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

    # Use command module
  - name: install filebeat
    command: dpkg -i filebeat-7.4.0-amd64.deb

    # Use copy module
  - name: drop in filebeat config
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/metricbeat/filebeat.yml

    # Use command module
  - name: enable and configure docker module for filebeat
    command: filebeat modules enable docker

    # Use command module
  - name: setup filebeat
    command: filebeat setup

    # Use command module
  - name: start filebeat
    command: service filebeat start
```

### Using the Playbook

In order to use the playbook, you will need to have an Ansible control node already configured.  We use the Jump-box for this purpose. Assuming you have such a control node provisioned: 

SSH into the Jump-box and follow the steps below:

- Copy the playbooks to the Ansible Control node.

- Run the playbook, and navigate to  appropriate target to check that the installation worked as expected.

The easiest way to copy the playbooks is to use Git: 

```bash
$ cd /etc/ansible
$ mkdir files
# Clone Repository + IaC Files
$ git clone https://github.com/deano10134/cloud-project.git
# Move Playbooks and hosts file Into `/etc/ansible`
$ cp cloud-project/playbooks/* .
$ cp cloud-project/files/* ./files
```

This copies the playbook files to the correct place.

Next, you must create a `hosts` file to specify which VMs to run each playbook on. Run the commands below:

```bash
$ cd /etc/ansible
$ cat > hosts <<EOF
[webservers]
10.1.0.12
10.1.0.13
10.1.0.4

[elk]
10.0.0.4
EOF
```

After this, the commands below run the playbook:

```bash
 $ cd /etc/ansible
 $ ansible-playbook install_elk.yml elk
 $ ansible-playbook install_filebeat.yml webservers
 $ ansible-playbook install_metricbeat.yml webservers
```

To verify success, wait five minutes to give ELK time to start up. 

Then, run: `curl http://10.0.0.4:5601`. This is the address of Kibana. If the installation succeeded, this command should print HTML to the console.
