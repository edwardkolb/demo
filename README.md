# SUDOKU CI/CD TASK COMPLETION WORKFLOW

- [Vagrant configuration](#vagrant-configuration)
- [Ansible playbook](#ansible-playbook)
- [Jenkins Getting Started](#jenkins-getting-started)
- [Jenkins Global Tools Configuration](#jenkins-global-tools-configuration)
- [Jenkins Plugins Manager](#jenkins-plugins-manager)


## Vagrant configuration

Vagrant is a tool for building and distributing development environments, for more detailed information please visit Website: https://www.vagrantup.com/

I won't describe installation process, you could easily google it  :globe_with_meridians:

For the quick-start, we'll bring up a development machine on VirtualBox because it is free and works on all major platforms. Vagrant can, however, work with almost any system such as OpenStack, VMware, Docker, etc.

For example: to build virtual environment based on Ubuntu 18.04 LTS 64-bit, just run:

	vagrant init hashicorp/bionic64
	vagrant up

:memo: ***Note:** The above vagrant up command will also trigger Vagrant to download the bionic64 box via the specified URL. If you need any other box, you could search it by following link: https://app.vagrantup.com/boxes/search*

After input `vagrant init` Vagrantfile with default machine configuration will be created automatically in your vagrant working directory. The primary function of the Vagrantfile is to describe the type of machine required for a project, and how to configure and provision these machines.

For my project I changed Vagrantfile as following:
```ruby

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
# Config VM Hostname
  config.vm.hostname = "ci-server"
  config.vm.box_check_update = false
# Config public IP address to establish connection with Jenkins web-interface
  config.vm.network "public_network", ip: "192.168.100.100"
# Config timezone
  if Vagrant.has_plugin?("vagrant-timezone")
    config.timezone.value = "Europe/Minsk"
  config.vm.provider "virtualbox" do |vb|
# By default Vagrant provides 512Mb of RAM, so I changed it to 1024Mb
     vb.memory = "1024"
# Vagrant VM's automatic naming is some kind of crazy, so I decided to change it as well
     vb.name = "Sudoku"
     end
  end

# Config VM provisioning by Ansible
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
  end
 
end
```
:memo: ***Note:** If you want to use a specific time zone in a Vagrant VM, or if a third party base box comes with a non-standard time zone, Vagrant timezone plugin is to the rescue. Install the plugin:* `vagrant plugin install vagrant-timezone`


## Ansible playbook

```python
---
- hosts: all
  become: True

  tasks:
    - name: UPGRADE PACKAGES
      apt: upgrade=dist update_cache=yes cache_valid_time=3600

    - name: INSTALL BASE PACKAGES
      apt: 
        name:
          - sudo
          - git
          - python3
          - python-pip
          - python3-pip
          - default-jre
        state: present
        autoclean: yes

    - name: ADD JENKINS REPOSITORY KEY
      apt_key:
        url: "https://pkg.jenkins.io/debian/jenkins.io.key"
        state: present

    - name: CLEAN APT LISTS
      file:
        path: "/var/lib/apt/lists/*"
        state: absent

    - name: UPDATE APT FOLLOWING APT MERGELISTS FIX
      apt: update_cache=yes

    - name: ADD JENKINS REPOSITORY
      apt_repository:
        repo: "deb http://pkg.jenkins.io/debian-stable binary/"
        state: present

    - name: INSTALL JENKINS
      apt: name=jenkins update_cache=yes cache_valid_time=3600

    - name: ALLOW SUDO GROUP TO HAVE PASSWORDLESS SUDO
      lineinfile:
        dest: /etc/sudoers
        state: present
        regexp: '^%sudo'
        line: '%sudo ALL=(ALL) NOPASSWD: ALL'

    - name: WAIT FOR JENKINS TO STARTUP
      uri:
        url: "http://192.168.100.100:8080/login?from=%2F"
        status_code: 200
        timeout: 5
      register: jenkins_service_status
      # Keep trying for 5 mins in 5 sec intervals
      retries: 60
      delay: 5
      until: >
         'status' in jenkins_service_status and
         jenkins_service_status['status'] == 200

    - name: RETRIEVE JENKINS UNLOCK CODE
      shell: "cat /var/lib/jenkins/secrets/initialAdminPassword"
      register: jenkins_unlock

    - debug: msg="Jenkins unlock code (install admin password) is {{ jenkins_unlock.stdout }}"
```

## Jenkins Getting Started

I'm going to use Jenkins via public IP address I configured for VM. To start use Jenkins just paste the URL http://192.168.100.100:8080 in a browser.

#### Step 1: Unlock Jenkins
![Alt-текст](https://github.com/edwardkolb/demo/blob/master/Pics/Unlock%20page.png)
:memo: ***Note:** To skip necessity check initialAdminPasswordlogin on server I configured following tasks in ansible playbook: "WAIT FOR JENKINS TO STARTUP" and "RETRIEVE JENKINS UNLOCK CODE". Just copy unlock code from task result and paste as Administrator password on "Unlock Jenkins" web-page*

#### Step 2: Install suggested plugins
You can install either the suggested plugins or selected plugins you choose. To keep it simple, we will install the suggested plugins.
![Alt-текст](https://github.com/edwardkolb/demo/blob/master/Pics/install%20pluging.png)
:memo: ***Note:** You can install either the suggested plugins or selected plugins you choose. To keep it simple, we will install the suggested plugins.*

#### Step 3: Create First Admin User
The next thing that you should do is create an Admin user for Jenkins. Then, enter your details and click **“Save and Continue”**.
![Alt-текст](https://github.com/edwardkolb/demo/blob/master/Pics/Admin%20pwd.png)

#### Step 4: Instance Configuration
Click **“Save and Finish”** to complete the Jenkins installation.
![Alt-текст](https://github.com/edwardkolb/demo/blob/master/Pics/Instance.png)

#### Step 5: Unlock Jenkins is ready!
Now, click “Start using Jenkins” to start Jenkins.
![Alt-текст](https://github.com/edwardkolb/demo/blob/master/Pics/Start%20using.png)


## Jenkins Global Tools Configuration
Go to Jenkins configuration to configure global configuration tools. Configure JDK and Maven.

![Alt-текст](https://github.com/edwardkolb/demo/blob/master/Pics/globalconfig.png)

![Alt-текст](https://github.com/edwardkolb/demo/blob/master/Pics/jdk.png)
:memo: ***Note:** To Install Oracle Java SE Development Kit, you should have oracle account.*

![Alt-текст](https://github.com/edwardkolb/demo/blob/master/Pics/mvn.png)


## Jenkins Plugins Manager
Additional plugins I used:
- [Warnings Next Generation Plugin] - warnings-ng plugin replace CheckStyle;
- [Green Balls] - because green is better than blue! 





