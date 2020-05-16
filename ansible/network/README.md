=============================================================================
==  Setting up network for your Pi Cluster  =================================
=============================================================================

The Ansible playbook main.yml is there to initialize the network for a cluster
of Raspberry Pis, with static IP addresses.
My setup uses a separate subnet in my home network for the Pis. That subnet
(10.0.0.0/24) is not physically separated from my main subnet (192.168.1.1/24),
and I use a VirtualBox Linux VM with two bridged network interfaces to perform
as the router connecting the Pi subnet to the rest of the world. Theoretically
that should allow me to disconnect my cluster and plug it somewhere else - but
that's not something I'm planning to do and I haven't tested it. So you can also
decide to allocate a set of static IPs to your Pis in your main subnet, just
make sure you reserve them with your DHCP so that you avoid a change of IP or
some collision in your network.

Steps to setup your cluster network is as follows:

-------------------------------------------------------------------------------
A: Booting each Pi - For each Raspberry Pi in your cluster:

- Step 1: Get latest Raspbian (lite or full, depending on the needs) from there: https://www.raspberrypi.org/downloads/raspbian/
- Step 2: Prepare a micro-sd card with the raspbian image
- Step 3: In the boot directory of the micro-sd, do the following changes:
             create an empty file called ssh  -   that's to be able to ssh to the headless pi
             update config.txt, and uncomment the line saying hdmi_force_hotplug=1  -  that's to enable the HDMI even if no monitor is plugged in at boot time, it can be useful to connect a screen after boot.
- Step 4: put the Micro-sd in the Pi, connect Ethernet cable and power supply.

-------------------------------------------------------------------------------
B: Controller host network config (Note: You can probably skip this and simply decide to put the Pis on the same subnet as the rest of your home network - just make sure you have some static IPs available for the Pis).

- Step 1: Identify the network interfaces of your controller host. In my setting, it has two network interfaces:
            - One on the general LAN of my home network (192.168.1.0/24), using DHCP from my home router
            - One on a separate network that I will use just for the Pi cluster (10.0.0.0/24), without dhcp. This network has no connection to the internet, initially
- Step 2: Make the controller host my gateway for the Pi cluster:

          On the controller host:
          Check page in https://www.thomaslaurenson.com/blog/2018/07/05/building-a-ubuntu-linux-gateway/, I might have forgotten steps below:
          # If the above returned 0, reboot the controller host
          # Below, replace <MAINETH> by the interface to the subnet managed by DHCP (the main home network)
          sudo iptables -t nat -A POSTROUTING -o <MAINETH> -j MASQUERADE
          sudo apt install iptables-persistent
          # Restart the host

-------------------------------------------------------------------------------
C: Initial Config of the Pis
- Step 1: Find the IP addresses of your Pis. I connect to my DHCP UI where I could find the leases given to hosts "Raspberry Pi".There are other ways to do it.
- Step 2: SSH to each Pi (user: pi, password: raspberry), and run:
    sudo raspi-config
    Use the interface to change the password of user Pi
- Step 3: Enable ssh by key:
   - On the manager host, run:
    ssh-keygen -t rsa      # Accept all default choices
   - Still on the host manager, for each Pi, run:
    ssh-copy-id -i ~/.ssh/id_rsa.pub pi@<pi address>    # You'll have to provide the password of each Pi there
- Step 4: Check SSH is setup properly
   - On the manager host, run for each Pi:
    ssh pi@<pi address>  # The first time you will have to confirm, but no password should be requested


-------------------------------------------------------------------------------
D: Prepare the Ansible playbooks
- Step 1: On the manager host, install Ansible:
     sudo apt install ansible
- Step 2: Edit the ansible files:
     git clone TODO
     cd pideploys/ansible/network
     cp example.inventory inventory.init
     vi inventory.init    # Under the [pi] line, list the current IP addresses of all your Pis - remove the example ones
     cp example.vars.yml vars.yml
     vi vars.yml          # Under mac_address_mapping, list the MAC addresses of your Pis, with name and ip set to the hostname and static IP address that you want them to take. dns_nameservers is probably ok as it is, but if you have a more local one you might want to change the values there. Set cluster_gateway to the IP of the gateway for that subnet
     # NOTE: The MAC addresses are case sensitive in this playbook, make sure the chars are lower case
     cp inventory.init inventory
     vi inventory.init    # Under the [pi] line, list the current IP addresses of all your Pis - remove the example ones
     cp example.vars.yml vars.yml
     vi vars.yml          # Under mac_address_mapping, list the MAC addresses of your Pis, with name and ip set to the hostname and static IP address that you want them to take. dns_nameservers is probably ok as it is, but if you have a more local one you might want to change the values there. Set cluster_gateway to the IP of the gateway for that subnet
     # NOTE: The MAC addresses are case sensitive in this playbook, make sure the chars are lower case
     cp inventory.init inventory
     vi inventory         # Replace the IP addresses by the ones that your Pis will have (as inserted in vars.yml). This inventory file will be the one to use after the network setup has been done.
     vi templates/hosts.js  # You might need to change the list of hosts there to match your IPs
- Step 3: Run the first network playbook
     ansible-playbook -C -i inventory.init main.yml
     # The above was a dry run. If there is any error, correct it. Then you can run:
     ansible-playbook -i inventory.init main.yml
     # You need to restart all Pis:
     ansible all -i inventory.init -m shell -a "sleep 1s; shutdown -r now" -b -B 60 -P 0

- Step 4: Checking the Pis
     Wait 30 seconds, then try and connect to the Pis with their new addresses:
     ssh pi@10.0.0.101, etc.... For each of them when that sudo works normally ("sudo vi /etc/hosts" should show you the file immediately. Otherwise edit the file so that the Pi knows itself)
     Then change the /etc/hosts of your manager host, so that it has a name of each pi
     sudo vi /etc/hosts
     Now you can reach your Pis through simple "ssh pi@pi1" commands. Congratulations !!


