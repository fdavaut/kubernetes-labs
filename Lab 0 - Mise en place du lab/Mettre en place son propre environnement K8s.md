# Lab Setup for Section 4 - Understand Kubernetes components

For this lesson, we will need to launch 2 VMs (One K8s master and one K8s worker nodes) which composed our cluster.
The files are inspired from https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/

**Step 1 :** Make sure these 3 softwares are installed:
* [Vagrant](https://www.vagrantup.com/downloads) : Direct link to the version (2.3.0) I'm using in my demo ==> [Link](https://releases.hashicorp.com/vagrant/2.3.0/)
* [VirtualBox](https://www.virtualbox.org/wiki/Downloads) : depending on your Operating System, install the corresponding one. I'm using VirtualBox for MacOs Intel ==> Direct [link](https://download.virtualbox.org/virtualbox/6.1.36/VirtualBox-6.1.36-152435-OSX.dmg)
* [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html): Ansible can't be install on Windows without WSL2. Please make sure to have Ansible correctely installed


**Step 2 :** Download Lab files and Launch the Lab environment

* Download the content of the directory `Lab files`
* Place yourself in the directory where the contents are
* Then Launch the Lab with vagrant command `$ vagran up`

## Lab setup on Mac OS

I'm using Intel Core MacOS version 12.5. These are the steps I followed

**Step 1 :** Homebrew installation

```
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Step 2 :** Vagrant installation

```
$ brew install vagrant
$ vagrant --version
Vagrant 2.3.0
```

**Step 3 :** VirtualBox installation

```
$ brew install virtualbox
```

**Step 4 :** Ansible installation

```
$ brew install ansible
$ ansible --version
ansible [core 2.13.3]
  config file = None
  configured module search path = ['/Users/mpakoupete/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/Cellar/ansible/6.3.0/libexec/lib/python3.10/site-packages/ansible
  ansible collection location = /Users/mpakoupete/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.10.6 (main, Aug 11 2022, 13:49:25) [Clang 13.1.6 (clang-1316.0.21.2.5)]
  jinja version = 3.1.2
  libyaml = True
```

**Step 5 :** Lauching Lab

```
$ pwd
/mastering-Kubernetes-from-Zero-to-Hero/Section 2 - Docker and Container refresher/Lab files

$ ls
Vagrantfile                docker-playbook.yaml       instructions_and_materials

$ vagrant up
```
<details><summary>result of the command</summary>

```
Bringing machine 'docker-server' up with 'virtualbox' provider...
==> docker-server: Importing base box 'bento/ubuntu-20.04'...
==> docker-server: Matching MAC address for NAT networking...
==> docker-server: Checking if box 'bento/ubuntu-20.04' version '202206.03.0' is up to date...
==> docker-server: Setting the name of the VM: Labfiles_docker-server_1661429724635_52484
==> docker-server: Clearing any previously set network interfaces...
==> docker-server: Preparing network interfaces based on configuration...
    docker-server: Adapter 1: nat
==> docker-server: Forwarding ports...
    docker-server: 22 (guest) => 2222 (host) (adapter 1)
==> docker-server: Running 'pre-boot' VM customizations...
==> docker-server: Booting VM...
==> docker-server: Waiting for machine to boot. This may take a few minutes...
    docker-server: SSH address: 127.0.0.1:2222
    docker-server: SSH username: vagrant
    docker-server: SSH auth method: private key
    docker-server: 
    docker-server: Vagrant insecure key detected. Vagrant will automatically replace
    docker-server: this with a newly generated keypair for better security.
    docker-server: 
    docker-server: Inserting generated public key within guest...
    docker-server: Removing insecure key from the guest if it's present...
    docker-server: Key inserted! Disconnecting and reconnecting using new SSH key...
==> docker-server: Machine booted and ready!
==> docker-server: Checking for guest additions in VM...
==> docker-server: Setting hostname...
==> docker-server: Mounting shared folders...
    docker-server: /vagrant => /Users/mpakoupete/Documents/Mawaki - Private/preparing-cka/mastering-Kubernetes-from-Zero-to-Hero/Section 2 - Docker and Container refresher/Lab files
    docker-server: /lab_data => /Users/mpakoupete/Documents/Mawaki - Private/preparing-cka/mastering-Kubernetes-from-Zero-to-Hero/Section 2 - Docker and Container refresher/Lab files/instructions_and_materials
==> docker-server: Detected mount owner ID within mount options. (uid: 1000 guestpath: /lab_data)
==> docker-server: Detected mount group ID within mount options. (gid: 1000 guestpath: /lab_data)
==> docker-server: Running provisioner: ansible...
    docker-server: Running ansible-playbook...

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [docker-server]

TASK [Install aptitude] ********************************************************
changed: [docker-server]

TASK [Install required system packages] ****************************************
changed: [docker-server]

TASK [Add Docker GPG apt Key] **************************************************
changed: [docker-server]

TASK [Add Docker Repository] ***************************************************
changed: [docker-server]

TASK [Update apt and install docker-ce] ****************************************
changed: [docker-server]

TASK [Install Docker Module for Python] ****************************************
changed: [docker-server]

TASK [Pull default Docker image] ***********************************************
changed: [docker-server]

TASK [Create default containers] ***********************************************
changed: [docker-server] => (item=1)
changed: [docker-server] => (item=2)
changed: [docker-server] => (item=3)
changed: [docker-server] => (item=4)

PLAY RECAP *********************************************************************
docker-server              : ok=9    changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

</details>

```
$ vagrant ssh docker-server
```
<details><summary>result of the command</summary>

```
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-110-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu 25 Aug 2022 12:23:54 PM UTC

  System load:  0.0                Processes:                124
  Usage of /:   14.6% of 30.63GB   Users logged in:          0
  Memory usage: 16%                IPv4 address for docker0: 172.17.0.1
  Swap usage:   0%                 IPv4 address for eth0:    10.0.2.15


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento
Last login: Thu Aug 25 12:17:16 2022 from 10.0.2.2
vagrant@docker-server:~$ 
```

</details>