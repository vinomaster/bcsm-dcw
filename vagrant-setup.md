# Local Environment Setup

This page provides step-by-step instructions for setting up an Ubuntu virtual machine on your Windows, Linux, or Mac PC for experimentation with Docker in a consistent environment. There are many ways to accomplish this end. These instructions describe an approach using VirtualBox and Vagrant.

## Check System Requirements

Docker requires a 64-bit architecture. Test that your machine is 64-bit before you proceed.
* Windows: [Follow these steps](http://support.microsoft.com/kb/827218)
* Mac/Linux: 

  ```
  $uname -m        #you should see something similar to x86_64
  ```

## Install Virtualbox

VirtualBox is an open source virtualizer, an application that can run an entire operating system within its own virtual machine. We will use it to run a virtual machine (VM) containing a pre-built copy of Docker on our local machines.
Do the following to install VirtualBox 4.3.6, the latest stable version tested for these instructions.

1. Download the installer for your laptop operating system using the links below.
  * [VirtualBox 4.3.6 for Windows hosts](http://download.virtualbox.org/virtualbox/4.3.6/VirtualBox-4.3.6-91406-Win.exe)
  * [VirtualBox 4.3.6 for OS X hosts](http://download.virtualbox.org/virtualbox/4.3.6/VirtualBox-4.3.6-91406-OSX.dmg)
  * [VirtualBox 4.3.6 for Linux hosts](https://www.virtualbox.org/wiki/Linux_Downloads) (requires that you pick your distro)
2. Run the installer, choosing all of the default options.
  * Windows: Grant the installer access every time you receive a security prompt.
  * Mac: Enter your admin password.
  * Linux: Enter your root password if prompted.
3. Reboot your laptop if prompted to do so when installation completes.
4. Close the VirtualBox window if it pops up at the end of the install.

## Install Vagrant

Vagrant is an open source command line utility for managing reproducible developer environments. While we could use the VirtualBox GUI to juggle virtual machines, their settings, and their distribution, Vagrant hides some of its complexity behind a few simple commands.

Do the following to install Vagrant 1.4.1, the latest stable version tested for these instructions.

1. Download the installer for your laptop operating system using the links below.
  * [Vagrant 1.4.3 for Windows hosts](https://dl.bintray.com/mitchellh/vagrant/Vagrant_1.4.3.msi)
  * [Vagrant 1.4.3 for OS X hosts](https://dl.bintray.com/mitchellh/vagrant/Vagrant-1.4.3.dmg)
  * [Vagrant 1.4.3 for Linux hosts](http://www.vagrantup.com/downloads.html) (requires that you pick your distro)
2. Run the installer, choosing all defaults.
3. Reboot your laptop if prompted to do so when installation completes.

## Windows Users Only: SSH

If you are running Windows on your laptop and have not installed Cygwin or the like, you’ll need to perform a few additional steps before Vagrant will be useful to you. Namely, you need to get a command line SSH (Secure SHell) client in order to connect to the virtual machine running on your laptop.

Installing Cygwin just to get SSH is overkill for our needs. A lower-overhead solution is to install git for Windows. This Windows installer includes a few common Unix command line utilities including the necessary ssh. (We’re not actually going to use git on Windows: we just want this package for its bundled copy of ssh.)

1. Visit [http://git-scm.com/download/win](http://git-scm.com/download/win).
2. If the installer does not download automatically, click to download it.
3. Run the installer.
  * Choose the defaults until prompted about adjusting your PATH..
  * Select the last option "Run Git and included Unix tools from the Windows Command Prompt"
  * Continue choosing defaults until the installer completes.

## Get the Docker Vagrantfile

With VirtualBox and Vagrant installed, we can now get a Vagrantfile that instructs Vagrant on how to create, configure, and provision a virtual machine for running Docker. We'll use Vagrantfile included in the Docker project itself.

1. Open a terminal window.
  * Windows: In the Start Menu, search for and run the Command Prompt application (cmd.exe). If you have Cygwin installed, you can run the Cygwin Bash Shell instead.
  * Mac: Run Terminal in the Applications folder.
  * Linux: You know what to do.
2. Make a folder that will serve as a shared directory between your host machine and the virtual machine containing Docker. Some suggestions:
  * Windows: 
```
    mkdir \Users\your_username\projects\docker_sandbox
```
  * Mac/Linux: 
```
    mkdir -p ~/projects/docker_sandbox
```
3. Download [https://raw.github.com/dotcloud/docker/v0.8.1/Vagrantfile](https://raw.github.com/dotcloud/docker/v0.8.1/Vagrantfile) and put it in the folder you created.
  * Windows: Use your web browser and "Save Link As". Make sure you strip off any extension (e.g., .txt) that your browser sticks on the file you downloaded.
  * Mac/Linux: 
```
    curl -o Vagrantfile https://raw.github.com/dotcloud/docker/v0.8.1/Vagrantfile
```
**NOTE**: The Vagrantfile linked above was removed from the master branch of the dotcloud/docker GitHub repository around 2014-02-23. We're linking to the version that was included in the last tagged release. See [https://github.com/dotcloud/docker/commit/67d55860a52bec8b1a1327355b4f27674ec912a](https://github.com/dotcloud/docker/commit/67d55860a52bec8b1a1327355b4f27674ec912aa) for the commit message. The reasoning: they're beginning to favor boot2docker.


## Bring-Up the Vagrant Box

Now that we have the Vagrantfile, we can use it to start a VM containing Docker.

1. In your terminal, bring up the VM with the following command(s):
  * Windows:
  ```
    * set VAGRANT_RAM=2048
    * set PRIVATE_NETWORK=192.168.44.44
    * vagrant up
  ```
2. Mac/Linux: VAGRANT_RAM=2048 PRIVATE_NETWORK=192.168.44.44 vagrant up​
Wait while Vagrant sets up the VM instance.
Vagrant will download an Ubuntu VM image and save it off in its own directory for safe keeping.
Vagrant will make a hidden copy of the VM image in the folder you created.
Vagrant will launch, configure, and provision an instance of the VM.
The Vagrantfile defaults to 512 MB of RAM. We want at least 2 GB for our follow-along sessions.
The Vagrantfile defaults to no networking between the host and VM. We want a private network for testing web apps and similar from our host machine web browsers. You can pick whatever IP address you like, but make sure it doesn't conflict with other IPs on your local network.
After many log messages, Vagrant returns you to the command prompt.
Vagrant informs you that folder sharing between your host machine and the VM is not yet active. Execute the following to enable it.
vagrant halt
Then repeat the commands you entered in step #1 of this section. (Don't forget the RAM and network variables!)
Enter the command vagrant ssh to start a shell session on the VM.
NOTE: Make sure you allow communication to your VM if/when prompted by your firewall.

Troubleshooting: Vagrant forces you to run init first

Vagrant might prompt you to do a vagrant init before you do a vagrant up. If you do, delete the Vagrantfile that the init command creates and replace it with the Vagrantfile from step 3. The vagrant init command creates a generic config while we want to use the one from the Docker project.

Troubleshooting: VM halts immediately

If you get an error about the VM being halted right after bringing it up, you likely need to enable support for virtualization on your laptop. This involves rebooting it, going into the BIOS setup, and finding the setting that says something like “Enable Virtualization Support”. Unfortunately, the steps for doing this vary widely across machines. Ask for help in the community forum if you're stuck.  

6. Check the Docker Daemon

We're now sitting in a shell on the virtual machine and can now use the docker command line to talk to the Docker daemon running on the VM. Let's try a few commands to test if everything is working properly. Enter the commands in bold. You should see output similar to the other lines. (The exact IDs may differ due to updates by the Docker team.)

vagrant@precise64:~$ docker pull busybox
Pulling repository busybox
769b9341d937: Download complete
511136ea3c5a: Download complete
bf747efa0e2f: Download complete
48e5f45168b9: Download complete

vagrant@precise64:~$ docker images
REPOSITORY                 TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
busybox                    latest              769b9341d937        8 days ago          2.489 MB

vagrant@precise64:~$ docker run busybox echo 'Hello from a busybox container!'
Hello from a busybox container!

vagrant@precise64:~$ docker ps -l
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
b1ff8741c523        busybox:latest      echo Hello from a bu   36 seconds ago      Exit 0                                  clever_mclean

vagrant@precise64:~$ docker run busybox ping -c 1 192.168.44.44
PING 192.168.44.44 (192.168.44.44): 56 data bytes
64 bytes from 192.168.44.44: seq=0 ttl=64 time=0.095 ms

--- 192.168.44.44 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.095/0.095/0.095 ms

vagrant@precise64:~$ docker run busybox ping -c 1 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=61 time=48.009 ms

--- 8.8.8.8 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 48.009/48.009/48.009 ms


It doesn't seem like much now, but stay tuned. The first few sessions should hopefully show you how powerful containerization can be.

Troubleshooting: dial unix /var/run/docker.sock: no such file or directory

If you receive the above error when trying the Docker commands above, the Docker daemon is not running. It sometimes fails to start after executing the vagrant halt && vagrant up commands. To start it, enter sudo service docker start and then try the Docker commands again.  

Troubleshooting: "docker run busybox ping  -c 1 192.168.44.44" has 100% packet loss


If this ping command fails, it likely means you forget to set the PRIVATE_NETWORK=192.168.44.44 environment variable when you invoked vagrant up. Follow the instructions in the prior section again to halt the VM and restart it with a host-local network IP address assigned.

Managing the VM

You can manage the Vagrant VM by returning to the folder containing the Vagrantfile on your host machine and using any of the available Vagrant commands in the terminal. If you "halt" or "destroy" the VM, make sure that you set the environment variables VAGRANT_RAM=2048 PRIVATE_NETWORK=192.168.44.44 on your next vagrant up.

Alternatives for the Adventurous

There are many other ways to install Docker, particularly if you have a native Linux system or an OS X machine. See the "Installation" section on the left side at http://docs.docker.io/en/latest/ for details.
