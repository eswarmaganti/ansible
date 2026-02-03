# Your Laptop is your New Cloud: Build a DevOps Lab Environment with Vagrant

### Introduction

As DevOps engineers, we constantly need test environments to experiment, validate automation, and practice configuration management. For a long time, I relied on free-tier cloud accounts to spin up virtual machines whenever I wanted to test Ansible playbooks.

While it works, the process is slow, repetitive, and often frustrating — creating accounts, managing credentials, provisioning instances, and cleaning everything up again.

Over time, I realized I was spending more effort setting up environments than actually learning and experimenting.

That’s when I switched to Vagrant.

With Vagrant, I can provision a complete multi-VM lab on my local machine in minutes. It’s fast, repeatable, and requires zero cloud setup. In this post, I’ll show you how to build a simple Vagrant-based lab that you can use to practice Ansible, networking, and automation — all on your laptop.


## Prerequisites

1. Install Ansible
2. Install Vagrant 
3. Install Oracle VirtualBox

## Building the Vagrnat Lab environment

### 1. Create the `Vagrantfile`

Create a file with name called `Vagrantfile` under your project directory with the below content

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-22.04"

  #path to your local SSH key
  SSH_KEY = File.expand_path("./vagrantkey.pub")

  NODES = [
    {name: "ansiblevagrantlab01", ip: "192.168.58.101"},
    {name: "ansiblevagrantlab02", ip: "192.168.58.102"},
    {name: "ansiblevagrantlab03", ip: "192.168.58.103"},
  ]

  # Common provisioning for all nodes
  config.vm.provision "shell", privileged: true, inline: <<-SHELL
    # Create ansible user
    id ansible &>/dev/null || useradd -m -s /bin/bash ansible
    mkdir -p /home/ansible/.ssh
    chmod 700 /home/ansible/.ssh

    # Add ansible controller public key
    cat /vagrant/vagrantkey.pub >> /home/ansible/.ssh/authorized_keys
    chmod 600 /home/ansible/.ssh/authorized_keys
    chown -R ansible:ansible /home/ansible/.ssh

    # Enable passwordless sudo
    echo "ansible ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/ansible
    chmod 400 /etc/sudoers.d/ansible

  SHELL

  # Dynamic node creation
  
  NODES.each do |node|
    config.vm.define node[:name] do |vm|
      vm.vm.hostname = node[:name]
      vm.vm.network "private_network", ip: node[:ip]

      vm.vm.provider :virtbox do |vb|
        vb.memory = 1024
        vb.cpus = 1
      end
    end
  end

end

```

### 2. Generate the SSH key pair
We need a SSH key-pair to enable passwordless key authentication from our local machine.
To generate a SSH key-pair run the below command in the project's root directory.

```bash

$ ssh-keygen -t ed25519 -f ./vagrantkey
Generating public/private ed25519 key pair.
Enter passphrase for "./vagrantkey" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in ./vagrantkey
Your public key has been saved in ./vagrantkey.pub
The key fingerprint is:
SHA256:/EXFBw7J7eKDuOp3YXXJg2Xp9z1SKL6Vz4s9m9Jt1/M eswarmaganti@Eswars-MacBook-Air.local
The key's randomart image is:
+--[ED25519 256]--+
|           ..+oo |
|            o+* .|
|            .Boo |
|       .   o+.B..|
|        S..+ooooo|
|        ..+oo+ .o|
|         o..o.= +|
|        o .. .o**|
|     .oo .   ..*E|
+----[SHA256]-----+

```

Try to list the files presnet in our project root directory, The SSH key-pair files will get generated successfully

```bash
$ ls -l
total 32
-rw-r--r--@ 1 eswarmaganti  staff  1242  3 Feb 05:53 Vagrantfile
-rw-------  1 eswarmaganti  staff   432  3 Feb 05:55 vagrantkey
-rw-r--r--  1 eswarmaganti  staff   119  3 Feb 05:55 vagrantkey.pub
```

### 3. Provision the Vagrant VM's
To provision the Vagrant VM's we can ran the below commands in our project root directory where the `Vagrantfile` present.

```bash
$ vagrant up
Bringing machine 'ansiblevagrantlab01' up with 'virtualbox' provider...
Bringing machine 'ansiblevagrantlab02' up with 'virtualbox' provider...
Bringing machine 'ansiblevagrantlab03' up with 'virtualbox' provider...
==> ansiblevagrantlab01: Importing base box 'bento/ubuntu-22.04'...
==> ansiblevagrantlab01: Matching MAC address for NAT networking...
==> ansiblevagrantlab01: Checking if box 'bento/ubuntu-22.04' version '202510.26.0' is up to date...
==> ansiblevagrantlab01: Setting the name of the VM: ubuntulab_ansiblevagrantlab01_1770079529437_97103
==> ansiblevagrantlab01: Clearing any previously set network interfaces...
==> ansiblevagrantlab01: Preparing network interfaces based on configuration...
    ansiblevagrantlab01: Adapter 1: nat
    ansiblevagrantlab01: Adapter 2: hostonly
==> ansiblevagrantlab01: Forwarding ports...
    ansiblevagrantlab01: 22 (guest) => 2222 (host) (adapter 1)
==> ansiblevagrantlab01: Booting VM...
==> ansiblevagrantlab01: Waiting for machine to boot. This may take a few minutes...
    ansiblevagrantlab01: SSH address: 127.0.0.1:2222
    ansiblevagrantlab01: SSH username: vagrant
    ansiblevagrantlab01: SSH auth method: private key
    ansiblevagrantlab01: Warning: Connection reset. Retrying...
    ansiblevagrantlab01: Warning: Remote connection disconnect. Retrying...
    ansiblevagrantlab01: 
    ansiblevagrantlab01: Vagrant insecure key detected. Vagrant will automatically replace
    ansiblevagrantlab01: this with a newly generated keypair for better security.
    ansiblevagrantlab01: 
    ansiblevagrantlab01: Inserting generated public key within guest...
    ansiblevagrantlab01: Removing insecure key from the guest if it's present...
    ansiblevagrantlab01: Key inserted! Disconnecting and reconnecting using new SSH key...
==> ansiblevagrantlab01: Machine booted and ready!
==> ansiblevagrantlab01: Checking for guest additions in VM...
==> ansiblevagrantlab01: Setting hostname...
==> ansiblevagrantlab01: Configuring and enabling network interfaces...
==> ansiblevagrantlab01: Mounting shared folders...
    ansiblevagrantlab01: /Users/eswarmaganti/Developer/Projects/Vagrant/ubuntulab => /vagrant
==> ansiblevagrantlab01: Running provisioner: shell...
    ansiblevagrantlab01: Running: inline script

[Output Trucated...]
```

We can check the status of our Vagrnt VM's once the 
provisioning step completed successfully.

```bash
$ vagrant status
Current machine states:

ansiblevagrantlab01       running (virtualbox)
ansiblevagrantlab02       running (virtualbox)
ansiblevagrantlab03       running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.

$ vagrant status ansiblevagrantlab01
Current machine states:

ansiblevagrantlab01       running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.

```

### 4. Access the Lab Environment
By default we can directly login to Vagrnat VM's using the vagrant commadline from our local machine as shown below

```
$ vagrant ssh ansiblevagrantlab01
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-160-generic aarch64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Feb  3 01:13:36 AM UTC 2026

  System load:           0.0
  Usage of /:            18.4% of 29.82GB
  Memory usage:          5%
  Swap usage:            0%
  Processes:             97
  Users logged in:       0
  IPv4 address for eth0: 10.0.2.15
  IPv6 address for eth0: fd17:625c:f037:2:a00:27ff:fed9:fa93


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento

Use of this system is acceptance of the OS vendor EULA and License Agreements.

vagrant@ansiblevagrantlab01:~$ hostname
ansiblevagrantlab01
vagrant@ansiblevagrantlab01:~$ whoami
vagrant
vagrant@ansiblevagrantlab01:~$ cat /etc/os-release 
PRETTY_NAME="Ubuntu 22.04.5 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.5 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```

But we can't directly access the VM's using SSH, the hostname's won't directly get resolved to IP addreses in our local machine unless we added those details in our `/etc/hosts` file as show below


```bash
cat /etc/hosts

#
# Vagrant Ubuntu Lab Hosts
#

192.168.58.101 ansiblevagrantlab01
192.168.58.102 ansiblevagrantlab02
192.168.58.103 ansiblevagrantlab03
```

Now you can directly reach the VM's using the hostname, we can test the connectivity by pinging the hostname

```bash
$ ping -c 5 ansiblevagrantlab01
PING ansiblevagrantlab01 (192.168.58.101): 56 data bytes
64 bytes from 192.168.58.101: icmp_seq=0 ttl=64 time=3.148 ms
64 bytes from 192.168.58.101: icmp_seq=1 ttl=64 time=0.914 ms
64 bytes from 192.168.58.101: icmp_seq=2 ttl=64 time=1.163 ms
64 bytes from 192.168.58.101: icmp_seq=3 ttl=64 time=0.912 ms
64 bytes from 192.168.58.101: icmp_seq=4 ttl=64 time=1.068 ms

--- ansiblevagrantlab01 ping statistics ---
5 packets transmitted, 5 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.912/1.441/3.148/0.859 ms
```

Test the SSH connectivity from local machine

```bash
$ ssh ansible@ansiblevagrantlab03 -i ./vagrantkey
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-160-generic aarch64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Feb  3 01:19:25 AM UTC 2026

  System load:           0.01
  Usage of /:            18.8% of 29.82GB
  Memory usage:          5%
  Swap usage:            0%
  Processes:             98
  Users logged in:       0
  IPv4 address for eth0: 10.0.2.15
  IPv6 address for eth0: fd17:625c:f037:2:a00:27ff:fed9:fa93


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento

Use of this system is acceptance of the OS vendor EULA and License Agreements.
$ whoami
ansible

$ hostname
ansiblevagrantlab03
```

### 5. Create an Ansible Inventory file

Create a file called `inventory.yaml` in your project directory and add the below ansible inventory details

```yaml
---
all:
  vars:
    ansible_user: ansible
    ansible_ssh_private_key_file: ./vagrantkey
    ansible_become: true
    ansible_become_method: sudo
    ansible_python_interpreter: "/usr/bin/python3"

  children:
    lab:
      hosts:
        ansiblevagrantlab01:
          ansible_host: 192.168.58.101
        ansiblevagrantlab02:
          ansible_host: 192.168.58.102
        ansiblevagrantlab03:
          ansible_host: 192.168.58.103
```

Now we will run an Ansible adhoc command to test the connectivity from our local Ansible Controller

```bash
$ ansible -m ping -i inventory.yaml lab

ansiblevagrantlab03 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
ansiblevagrantlab01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
ansiblevagrantlab02 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

> NOTE: For the first time, we need to accept the StrictHostKeyChecking prompt, then ansible can able to successfully connect to the target nodes.

### 6. Write a playbook to install Nginx Web Server 
We will write a playbook to test the installation of nginx web server and try to access the web page from our local machine.

Create a file called `playbook.yaml` with below content

```yaml
---
- name: Install Nginx Web Server
  hosts: all
  gather_facts: false
  vars:
    packages:
      - nginx
      - python3
      - python3-apt
      - python3-pip
      - python3-dev
      - python3-setuptools
  tasks:
    - name: Install the nginx & Python packages
      ansible.builtin.apt:
        name: "{{ packages }}"
        state: present
        update_cache: yes
      become: true

    - name: Start the Nginx Service
      ansible.builtin.service:
        name: nginx
        state: started
    
    - name: Check the Nginx Service Status
      ansible.builtin.service:
        name: nginx
      register: output
    
    - name: Log the service status
      ansible.builtin.debug:
        msg: "{{ output.status.ActiveState }}"
```

Now run the playbook aganist one target node `ansiblevagrantlab01` 

```shell
$ ansible-playbook playbook.yaml -i inventory.yaml --limit ansiblevagrantlab01

PLAY [Install Nginx Web Server] ***********************************************************************************

TASK [Install the nginx & Python packages] ************************************************************************
changed: [ansiblevagrantlab01]

TASK [Start the Nginx Service] ************************************************************************************
ok: [ansiblevagrantlab01]

TASK [Check the Nginx Service Status] *****************************************************************************
ok: [ansiblevagrantlab01]

TASK [Log the service status] *************************************************************************************
ok: [ansiblevagrantlab01] => {
    "msg": "active"
}

PLAY RECAP ********************************************************************************************************
ansiblevagrantlab01        : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
Our playbook successfully executed and configured python and nginx packages.

Now we can access the nginx webserver configured and running in the target node using a curl or directly opening the url in our local browser

```bash
$ curl --insecure http://ansiblevagrantlab01 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

![Nginx Web Server Page](images/nginx.png)
