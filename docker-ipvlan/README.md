# Using ipvlan Interfaces with Docker Containers

These files were created to allow users to use Vagrant ([http://www.vagrantup.com](http://www.vagrantup.com)) to build an environment for working with Docker containers using ipvlan interfaces. This configuration was tested using Vagrant 1.8.1, VMware Fusion 8.1.0, and version 4.0.5 of the Vagrant VMware plugin.

## Contents

* **machines.yml**: This is a YAML file containing the configuration data used by Vagrant when creating and provisioning VMs. This particular Vagrant environment requires six (6) values in this file for each VM: `name` (name to be assigned to the box as well as used for hostname), `box` (the name of the Vagrant box), `ram` (desired RAM), `vcpu` (number of virtual CPUs), `ip_addr` (the private IP address to be assigned to the second network interface; also controls whether a second interface is provisioned), and `docker` (set to "true" or "false"; controls whether Vagrant provisions Docker onto the VM).

* **README.md**: The file you're currently reading.

* **Vagrantfile**: This file is used by Vagrant to spin up the virtual machines for this environment. No changes need to be made to this document, as all the configuration data is found in other files (like `machines.yml`). However, if you are using a virtualization solution _other_ than VMware Fusion, you might need to make changes to this file.

## Instructions

These instructions assume you've already installed VMware Fusion, Vagrant, and the Vagrant VMware plugin. Please refer to the documentation for those products for more information on installation or configuration. Note that Internet access is required when using `vagrant up` to create this environment.

1. Use `vagrant box add` to install _both_ an Ubuntu 15.04 x86_64 box _and_ an Ubuntu 14.04 x86_64 Vagrant box. I have an Ubuntu 14.04 x86_64 base box you can use; to use my base box, add the box with `vagrant box add slowe/ubuntu-trusty-x64`. A Vagrant box you might consider using for Ubuntu 15.04 is the `bento/ubuntu-15.04` box.

2. Place the files from the `docker-ipvlan` directory of this GitHub repository into a directory on your local system. You can clone the entire "learning-tools" repository (using `git clone`) or just download the specific files from the the `docker-ipvlan` folder.

3. If necessary, edit `machines.yml` to specify the name of the Vagrant boxes downloaded in step 1. The VM definition for "docker-01" **must** use Ubuntu 15.04; the other VM may use any version of Ubuntu you prefer. You may also make changes to the private IP addresses or the RAM/CPU settings, although no changes are typically required.

4. Run `vagrant up` to instantiate the learning environment. This will spin up two (2) VMs based on the Vagrant boxes you downloaded in step 1 and specified in `machines.yml` in step 3. Vagrant will also appropriately configure each VM and start the necessary services. Depending on the speed of your system and your Internet connection, this may take a few minutes.

5. Use `vagrant ssh docker-01` to connect to the first Docker host and run the following commands to download and install the ipvlan plugin for Docker:

        git clone https://github.com/gopher-net/ipvlan-docker-plugin.git
        cd ipvlan-docker-plugin/binaries
        sudo ./ipvlan-docker-plugin-0.2-Linux-x86_64 \
        --ipvlan-subnet "192.168.100.0/24" \
        --gateway "192.168.100.1" --mode "l2" \
        --host-interface "eth1" -d &

    If you changed the private IP addresses in `machines.yml`, be sure to supply the appropriate IP address ranges from that file in the command above.

6. Run this command to create a network using the ipvlan plugin:

        docker network create -d ipvlan --subnet=192.168.100.0/24 \
        --gateway=192.168.100.1 -o host_iface=eth1 -o mode=l2 testnet

    Note that the subnet, gateway, host interface, and mode specified here need to match what was provided when the plugin was launched in step 5.

7. Launch a Docker container attached to this new network with this command line:

        docker run -it --rm --net=testnet ubuntu

    This will drop you at a root prompt for an Ubuntu container. Use `ip addr list` to determine the IP address assigned to this container.

8. In a separate terminal window, switch to the directory where the files for this environment are stored and connect to the remote VM using `vagrant ssh remote-01`.

9. In the remote VM, ping the IP address of the container (obtained in step 7 when you launched the container).

10. From the container prompt, ping the IP address assigned to the remote VM (by default, 192.168.100.101).

Enjoy!
