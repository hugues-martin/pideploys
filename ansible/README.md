In this directory I'll put all the ansible playbooks to setup the
Raspberry Pi cluster from a control node


Playbooks should be used the following way:

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


C: Initial Config of the Pis
- Step 1:


