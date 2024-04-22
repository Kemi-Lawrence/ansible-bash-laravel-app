# Automating LAMP Stack Deployment with Vagrant and Ansible

## Introduction

This documentation outlines the steps to automate the provisioning of two Ubuntu-based servers named "Master" and "Slave" using Vagrant and Ansible. The Master node will host a LAMP (Linux, Apache, MySQL, PHP) stack with a PHP application deployed from GitHub. The Slave node will execute a bash script deployed via Ansible and verify the accessibility of the PHP application.

## GitHub Repository

The code for this project can be found in the following GitHub repository: 
! [https://github.com/Kemi-Lawrence/ansible-bash-laravel-app]

## Provisioning Servers with Vagrant

1. Clone the GitHub repository to your local machine.
2. Navigate to the project directory.
3. Run `vagrant init` to create the vagrant file.
4. Inside the vagrant file, define the configuration for both master and slave servers respectively as stated below.

 The master is defined in the Vagrant file

 `  config.vm.define "master" do |master|
    master.vm.box = "ubuntu/focal64"
    master.vm.network "private_network", ip: "192.168.50.14"
end `

5. Inside the Slave configuration, User and right were granted in the vagrant file. This allows us to automate the slave node with user whenever we ssh into the slave .

`# Define slave
  config.vm.define "slave" do |slave|
    slave.vm.box = "ubuntu/focal64"
    slave.vm.network "private_network", ip: "192.168.50.15"
    slave.vm.provision "shell", inline: <<-SHELL
      # Create ansible user
      useradd -m -s /bin/bash ansible
      # Set password for ansible user
      echo "ansible:ansiblePasswd" | chpasswd
     # Grant sudo access without password to the ansible user
        echo "ansible ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/ansible
     # Allow password authentication in SSH server configuration
        sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
     # Restart SSH service
        systemctl restart ssh
     SHELL
end`

6. Run `vagrant up` to provision the Master and Slave servers respectively.
7. Once provisioning is complete, log in to the Master server with `vagrant ssh master` and to the Slave server with `ssh ansible@192.168.50.15` from the master server which is explicitly stated in this markdown file.

## Automating LAMP Stack Deployment on Master Node

1. On the Master node, a bash script named `lampdeploy.sh` automates the deployment of the LAMP stack and PHP application.
2. The script clones the PHP application from GitHub, installs necessary packages, and configures Apache web server and MySQL.
3. The bash script is reusable and readable, ensuring easy maintenance and modification. `cat lampdeploy.sh` file to view or modify content of bash script.

## Executing Bash Script on Slave Node with Ansible

1. Using an Ansible playbook named `playbook.yml`, execute the `lampdeploy.sh` script on the Slave node with the below commands.
`ssh ansible@192.168.50.15`this would allow us log into the slave server from the master's and a user password created earlier would be required for successful login.

2. Verify the accessibility of the PHP application through the Slave node's IP address using `ansible-playbook playbook.yml`.


## Creating Cron Job to Check Server's Uptime

1. Create a cron job on the Slave node to check the server's uptime every day at 12 am. This is done by inserting the below name cmd line to the task in the `playbook.yml` file.
` - name: Create cron job to check server's uptime
      cron:
        name: Check Uptime
        minute: 0
        hour: 0
        job: /path/to/script/or/command >> /var/log/uptime.log 2>&1`

2. Ensure the cron job is configured correctly to execute the desired command.

## Screenshots

### Command lines for Master, Slave Node Bash Script Execution and Application Accessibility Verification
![Slave Node Bash Script Execution and Application Accessibility Verification using `ssh ansible@192.168.50.15`](/images/User_in_Slave_Node.png)

### Cron Job Configuration
![Cron Job Configuration](/images/playbook.yml.png)

### Server Uptime Check
![Server Uptime Check](/images/Evidence_of_running_playbook.png)


### Laravel Application Check
![Laravel Application Check](/images/Laravel_app_running.png)

## Command lines history
![command lines](/images/cmd_lines_history.png)

## Conclusion

This project showcases the automated provisioning of Ubuntu servers, deployment of a LAMP stack and PHP application, execution of bash scripts using Ansible, and setup of cron jobs. The documentation includes a detailed, step-by-step guide accompanied by screenshots to facilitate easy replication and comprehension.
