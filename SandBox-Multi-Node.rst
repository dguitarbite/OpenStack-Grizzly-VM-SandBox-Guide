==========================================================
  OpenStack Grizzly VM SandBox and Install Guide
==========================================================

:Version: 0.8 (Beta-1)
:Source: https://github.com/mseknibilel/OpenStack-Grizzly-Install-Guide
:Keywords: Single node OpenStack, Grizzly, Quantum, Nova, Keystone, Glance, Horizon, Cinder, LinuxBridge, KVM, Ubuntu Server 12.04 (64 bits).

Authors
==========

Copyright (C) Pranav Salunke <dguitarbite@gmail.com>


Table of Contents
=================

::

  0. What is it?
  1. Requirements
  2. Setup Your VM Environment
  3. Add Virtual Networks
  4. Install SSH and FTP
  5. Install Your VM's Instances
  6. Its about to get sticky
  7. Control Node 
  8. Compute 
  9. Quantum
  10. Launch OpenStack Horizon Dashboard
  11. Word Of Advice
  12. Licensing
  13. Contacts
  14. Acknowledgment
  15. Credits
  16. To do

0. What is it?
==============
Well this guide will help you to setup your own OpenStack Cloud on Virtual Machines with Virtual Networks. 
These Virtual Machines and Virtual Networks will be given equal privilege as a physical machine on a physical network.

**Note :** This Guide is not responsible for teaching OpenStack, Networking , Virtualization and Linux related concepts.

For learning more follow these links:

OpenStack:
  1.I am using `**OpenStack Grizzly Install Guide** <https://github.com/mseknibilel/OpenStack-Grizzly-Install-Guide/>`_ by  **SkiBLE mseknibilel** as it is well written, easy and tested by Stackers, with regular updates. 
  
  2.If you want to blow your brains out then you can refer the OpenStack Official Website which contains all the related Documentation, API's, guides , Wiki's etc. Access the OpenStack Official Website `here <http://www.openstack.org/>`_


Networking:
  1.Basic Networking concepts are necessary but can be ignored, if you want to dig into the field of networking I would suggest `Computer Networks (5th Edition) by Andrew S. Tanenbaum <http://www.amazon.com/Computer-Networks-5th-Andrew-Tanenbaum/dp/0132126958>`_  is an awesome one to learn networking 
  
  2.For learning Virtual Networking or Networking for Virtual Machines the following guide by Virtual Box `here <http://www.virtualbox.org/manual/ch06.html>`_  should suffice.
  **Note :** This is required as further in the guide these Concepts will be handy and you need to know what kind of networks you are setting up as there will be nesting of networks , meaning Virtual Networks inside Virtual Networks.

Virtualization:
  1.To Learn Virtualization, go through the Virtual Box Guide `here <http://www.virtualbox.org/manual/UserManual.html>`_, this should be sufficient to provide a fair idea on Virtualization related concepts.
  
  2.Virtual Box provides User Interface and API for using it, although API will provide more flexibility, UI will have lesser learning curve, its up to you. I will try to provide both if time permits but I have to remind my-selves that this guide is meant for OpenStack sand-boxing :).
  You can access the API's for advanced networking `here <https://www.virtualbox.org/wiki/Advanced_Networking_Linux>`_.

Linux:
  1.You will need some basic knowledge of Linux` and Shell Programming otherwise you will go through tremendous torture of blindly following these Guides and if in case you end up with an error/dead lock, you will get stuck for silly reasons. There are many books, docs available and I don't know which one to recommend so please `Google <https://www.google.com/>`_ it.


Version 1.0

Status: RC


1. Requirements
=============

* Operating Systems - Either Ubuntu Server 12.04 LTS or Ubuntu Server 13.04,
  
    **Note :** Ubuntu 12.10 is not supporting OpenStack Grizzly Packages. Ubuntu team has decided not to package Grizzly Packages for Ubuntu 12.10. Although OpenStack Folsom will be supported as usual.

* Recommended Requirements.
  

  :VT Enabled PC: Intel ix or Amd QuadCore
  :4GB Ram: DDR2/DDR3

* Minimum Requirements.
  
  
  :Non-VT PC's: Intel Core 2 Duo or Amd Dual Core
  :2GB Ram: DDR2/DDR3

* If you don't know weather your processor is VT enabled, you could check it by installing **cpu checker**
  ::
    $sudo apt-get install cpu-checker
    $sudo kvm-ok
  
* If your device does not support VT it will show
  ::
    INFO: Your CPU does not support KVM extensions
    KVM acceleration can NOT be used
          
* Don't worry you will still be able to use Virtual Box but it will be very slow, so I must consider putting the requirements to be Patience or VT enabled processor ;).

* Well there are many ways to configure your OpenStack Setup but I am going to follow `OpenStack-Grizzly-Install-guide <https://github.com/mseknibilel/OpenStack-Grizzly-Install-Guide/blob/OVS_SingleNode/OpenStack_Grizzly_Install_Guide.rst>`_

* This time I am going to cover all types of networks/installations that are covered by the above mentioned guide. Although it is pretty obvious and easy to deploy other types of installations once you understand what happens in this one, but still I have my exams and nothing better to do!


There are two different types of configurations that are possible for setting up of Virtual Networks.

**1. Bridged Connections :** 
------------
* Bridged Connection connects your VM as if its a physical machine. This means that your machine will be able to use Internet and can be traced from other machines from Internet. So if your network has a physical switch or you can spare a few IP addresses then I would suggest bridged connection.

* Advantage of bridged connections is that your networks remain the same and you are free of the hassles of creating virtual networks.


  :Node Role: NICs
  :Single Node: eth0 (10.10.10.51), eth1 (192.168.100.51)

**Note:** If you are using bridged connections you may skip this section (2. Host-Only )as there is no need to set up host-only connections.

**2. Host Only Connections:** 
------------
* Host only connections provide an Internet network between your host and the Virtual Machine instances up and running on your host machine. This network is not traceable by other networks.

* The following are the host only connections that you will be setting up later on :

  1. vboxnet0 - OpenStack Management Network - Host static IP 10.10.10.1 
  2. vboxnet1 - VM Conf. Network - Host Static IP 10.20.20.1
  3. vboxnet2 - VM External Network Access (Host Machine)


2. Setup Your VM Environment
==============

* Well a few of these sections will be full of screen-shots because it is essential for people to understand some of the networking related configurations so please bear with me since its quite necessary to put it up.

* Before you can start configuring your Environment you need to download some of the following stuff:

  1. `Oracle Virtual Box <https://www.virtualbox.org/wiki/Downloads>`_
        Note: You cannot set up a amd64 VM on a x86 machine. 
        
  2. `Ubuntu 12.04 Server or Ubuntu 12.10 Server <http://www.ubuntu.com/download/server>`_
        Note: You need a x86 image for VM's if kvm-ok fails, even though you are on amd64 machine.

  3. For testing I'm Using these machines - 
        * **Machine 1** -My host machine is Ubuntu 12.04 amd64 (Core2duo (VT not supported),4GB Ram DDR2)
          * For Testing this guide on a Non-VT enabled Machine.
        * **Machine 2** -Ubuntu 12.10 amd64 (Intel i5 2nd gen (VT enabled), 8GB Ram DDR3)
          * For Testing this guide on a VT enabled Machine.
        **Note :** I'm using only one machine for Deploying OpenStack. These two machines are for Testing.

        * Please do consider using quad core processors as they are VT enabled. Which is required for virtualization.
          At the worst case go for a dual core processor.

**Note:** Even Though I'm using Ubuntu as Host, the same is applicable to Windows, Mac and other Linux Hosts. 

* If you have i5 or i7 2nd gen processor you can have VT technology inside VM's provided by VmWare. This means that your OpenStack nodes(Which are in turn VM's) will give positive result on KVM-OK. (I call it - Nesting of type-2 Hypervisors). Rest of the configurations remain same except for the UI and few other trivial differences.

3. Configure Virtual Networks 
==============

* This section of the guide will help you setup your networks for your Virtual Machine.

* **Note :** If you are using Bridged Connections you may skip this section.

  1. Click on **File >> Preferences** present on the menu bar of Virtual Box.
  2. Select the **Network** tab.
  3. On the right side you will see an option to add *Host-Only* networks.
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/1.%20Virtual%20Network/Create%20Host%20Only%20Network.png
  4. Create three Host-Only Network Connections.
  5. Edit the Host-Only Connections to have the following settings.
      
    * Vboxnet0

      +----------------------------+-----------------------+
      | Option                     |  Value                |
      +============================+=======================+
      | IPv4 Address:              | 10.10.10.1            |
      +----------------------------+-----------------------+
      | IPv4 Network Mask:         | 255.255.255.0         |
      +----------------------------+-----------------------+
      | IPv6 Address:              | **Can be Left Blank** |
      +----------------------------+-----------------------+
      | IPv6 Network Mask Length : | **Can be Left Blank** |
      +----------------------------+-----------------------+

      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/1.%20Virtual%20Network/Configure%20Vboxnet0.png
      
    * Vboxnet1

      +----------------------------+-----------------------+
      | Option                     |  Value                |
      +============================+=======================+
      | IPv4 Address:              | 10.20.20.1            |
      +----------------------------+-----------------------+
      | IPv4 Network Mask:         | 255.255.255.0         |
      +----------------------------+-----------------------+
      | IPv6 Address:              | **Can be Left Blank** |
      +----------------------------+-----------------------+
      | IPv6 Network Mask Length : | **Can be Left Blank** |
      +----------------------------+-----------------------+

      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/1.%20Virtual%20Network/Configure%20Vboxnet1.png
            
    * Vboxnet2

      +----------------------------+-----------------------+
      | Option                     |  Value                |
      +============================+=======================+
      | IPv4 Address:              | 192.168.100.1         |
      +----------------------------+-----------------------+
      | IPv4 Network Mask:         | 255.255.255.0         |
      +----------------------------+-----------------------+
      | IPv6 Address:              | **Can be Left Blank** |
      +----------------------------+-----------------------+
      | IPv6 Network Mask Length : | **Can be Left Blank** |
      +----------------------------+-----------------------+

      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/1.%20Virtual%20Network/Configure%20Vboxnet2.png
      

4. Install SSH and FTP
==============

* **This is for beginners ... **

* You may benifit by installing SSH and FTP so that you could use your remote shell to login into the machine and use your terminal which is more convenient that using the Virtual Machines tty through the Virtual Box's  UI. You get a few added comforts like copy - paste commands into the remote terminal which is not possible directly on VM.

* FTP is for transferring files to and fro ... you can also use SFTP or install FTPD on both HOST and VM's.

* Installation of SSH and FTP with its configuration is out of scope of this GUIDE and I may put it up but it depends upon my free time. If someone wants to contribute to this - please do so. 

**Note:** Please set up the Networks from inside the VM before trying to SSH and FTP into the machines. I would suggest setting it up at once just after the installation of the Server on VM's is over.


5. Install Your VM's Instances
==============

* During Installation of The Operating Systems you will be asked for Custom Software to Install , if you are confused or not sure about this, just skip this step by pressing **Enter Key** without selecting any of the given Options.

**Warning -** Please do not install any of the other packages except for which are mentioned below unless you know what you are doing. There is a good chance that you may end up getting unwanted errors, package conflicts ... due to the same.


1. Control Node: Install **SSH server** when asked for **Custom Software to Install**. Rest of the packages are not required and may come in the way of OpenStack packages - like DNS servers etc. (not necessary). Unless you know what you are doing.

   Configure the networks 
  
    * Host-Only
    
     Network Adapter | Host-Only Adapter Name |IP Address
    -----------------|------------------------|-----------
     eth0            | Vboxnet0               |10.10.10.51
     eth1            | Vboxnet2               |10.20.20.51
     eth2            | NAT                    |DHCP

    1. Adapter 0 (Vboxnet0)
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Control%20Node/Host%20Only/CN%20Network1.png
    
    2. Adapter 1 (Vboxnet2)
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Control%20Node/Host%20Only/CN%20Network2.png
    
    3. Adapter 2 (NAT)
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Control%20Node/Host%20Only/CN%20Network3.png

   * Bridged
    
     Network Adapter | IP Address
    -----------------|-------------
     eth0            |  10.10.10.51
     eth1            |  192.168.100.51
    
    1. Adapter 0
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Control%20Node/Bridged/CN%20Network1.png
    
    2. Adapter 1
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Control%20Node/Bridged/CN%20Network2.png


2. Network Node: Same as Control Node.

   Configure the networks 
  
    * Host-Only
    
     Network Adapter | Host-Only Adapter Name |IP Address
    -----------------|------------------------|-----------
     eth0            | Vboxnet0               |10.10.10.52
     eth1            | Vboxnet1               |10.20.20.52
     eth2            | Vboxnet2               |192.168.100.51
     eth3            | NAT                    |DHCP


    1. Adapter 0 (Vboxnet0)
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Network%20Node/Host%20Only/NN%20Network%201.png
    
    2. Adapter 1 (Vboxnet1)
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Network%20Node/Host%20Only/NN%20Network2.png
    
    3. Adapter 2 (Vboxnet2)
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Network%20Node/Host%20Only/NN%20Network3.png
    
    4. Adapter 3 (NAT)
    
      ..image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Network%20Node/Host%20Only/NN%20Network4.png
      
      
    * Bridged
    
     Network Adapter | IP Address
    -----------------|-------------
     eth0            |  10.10.10.52
     eth1            |  10.20.20.52
     eth2            |  192.168.100.52
    
    1. Adapter 0
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Network%20Node/Bridged/NN%20Briged1.png
    
    2. Adapter 1
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Network%20Node/Bridged/NN%20Bridged2.png

    3. Adapter 2
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Network%20Node/Bridged/NN%20Bridged3.png

3. Compute Node: Same as Control Node.

   Configure the networks 
  
    * Host-Only
    
     Network Adapter | Host-Only Adapter Name |IP Address
    -----------------|------------------------|-----------
     eth0            | Vboxnet0               |10.10.10.53
     eth1            | Vboxnet1               |10.20.20.53
     eth2            | NAT                    |DHCP

    1. Adapter 0 (Vboxnet0)
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Compute%20Node/Host%20Only/CN%20Network1.png
    
    2. Adapter 1 (Vboxnet1)
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Compute%20Node/Host%20Only/CN%20Network2.png
    
    3. Adapter 2 (NAT)
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Compute%20Node/Host%20Only/CN%20Network3.png

   * Bridged
    
     Network Adapter | IP Address
    -----------------|-------------
     eth0            |  10.10.10.51
     eth1            |  10.20.20.53
    
    1. Adapter 0
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Compute%20Node/Bridged/CN%20Network1.png
    
    2. Adapter 1
    
      .. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/2.%20Setup%20VM/Multi%20Node/Compute%20Node/Bridged/CN%20Network2.png


* Well there are a few warnings that I must give you out of experience due to stupid habits that normal Users like me have -
    1. Sometimes shutting down your Virtual Machine may lead to malfunctioning of OpenStack Services. Try not to direct shutdown your VMs. Saving your VM's State can save some time.
    2. If you are using bridged connection over a different physical router and have a separate Internet connection/network ... then you can put up additional network interface NAT connections on your VM's for giving them Internet Access.
    3. In case your VM's dont get internet
    ::
        // Use ping command to see whether Internet is on.
        $ping www.google.com
        // If its not connected restart networking service-
        $sudo service networking restart
        // Now Ping again
        $ping www.google.com
* This should reconnect your network about 99% of the times. If you are really unlucky you must be having some other problems or your Internet connection itself is not functioning... well try to avoid immature decisions. Believe me you don't want to mess up your existing setup.

**Note :** There are known bugs with the `ping` under NAT. Although the latest versions of Virtual Box have better performance, sometimes ping may not work even if your Network is connected to internet.

**If you have Reached till here, I would suggest a coffee break because now the Virtual Machines installation is nearly over and OpenStack's installation part is going to start**
-------------

7. Control Node
==============

7.1. Preparing Ubuntu 13.04/12.04
------------

**Note :** Please Skip this (7.1) for Ubuntu 13.04. Ubuntu 12.10 dosent support Grizzly.

* After you install Ubuntu 12.04 Server 64bits, Go in sudo mode and don't leave it until the end of this guide::

   sudo su

* Add Grizzly repositories::

   apt-get install ubuntu-cloud-keyring python-software-properties software-properties-common python-keyring
   echo deb http://ubuntu-cloud.archive.canonical.com/ubuntu precise-updates/grizzly main >> /etc/apt/sources.list.d/grizzly.list

* Update your system::

   apt-get update
   apt-get upgrade
   apt-get dist-upgrade


7.2.Networking
------------

Configure your network by editing :: /etc/network/interfaces file

* Only one NIC on the controller node need Internet access::
  
    # NAT should be preconfigured otherwise can copy the following ...
    # Important -- NAT (eth2) is not required for Bridged Connection.
    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).

    # The loopback network interface - for Host-Only
    auto lo
    iface lo inet loopback
    
    # The primary network interface - Virtual Box NAT connection
    auto eth2
    iface eth2 inet dhcp
    
    # Virtual Box vboxnet0 - OpenStack Management Network
    auto eth0
    iface eth0 inet static
    address 10.10.10.51
    netmask 255.255.255.0
    gateway 10.10.10.1
  
    # Virtual Box vboxnet2 - for exposing OpenStack API over external network
    auto eth1
    iface eth1 inet static
    address 192.168.100.51
    netmask 255.255.255.0
    gateway 192.168.100.1



For the remaining Installation Follow `OpenStack-Grizzly-Install-guide <https://github.com/mseknibilel/OpenStack-Grizzly-Install-Guide/blob/OVS_SingleNode/OpenStack_Grizzly_Install_Guide.rst#23-mysql--rabbitmq>`_

8. Network Node
==============

8.1. Preparing Ubuntu 13.04/12.0re4
------------

**Note :** Please Skip this (8.1) for Ubuntu 13.04. Ubuntu 12.10 dosent support Grizzly.

* After you install Ubuntu 12.04 Server 64bits, Go in sudo mode and don't leave it until the end of this guide::

   sudo su

* Add Grizzly repositories::

   apt-get install ubuntu-cloud-keyring python-software-properties software-properties-common python-keyring
   echo deb http://ubuntu-cloud.archive.canonical.com/ubuntu precise-updates/grizzly main >> /etc/apt/sources.list.d/grizzly.list

* Update your system::

   apt-get update
   apt-get upgrade
   apt-get dist-upgrade


8.2.Networking
------------

Configure your network by editing :: /etc/network/interfaces file

* Only one NIC on the controller node need Internet access::
  
    # NAT should be preconfigured otherwise can copy the following ...
    # Important -- NAT (eth2) is not required for Bridged Connection.
    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).

    # The loopback network interface
    auto lo
    iface lo inet loopback
    
    # The primary network interface - Virtual Box NAT connection - for Host-Only
    auto eth3
    iface eth2 inet dhcp
    
    # Virtual Box vboxnet0 - OpenStack Management Network
    auto eth0
    iface eth0 inet static
    address 10.10.10.52
    netmask 255.255.255.0
    gateway 10.10.10.1
    
    # Virtual Box vboxnet1 - for VM Internal Communication
    auto eth1
    iface eth1 inet static
    address 10.20.20.52
    netmask 255.255.255.0
    gateway 10.20.20.1
  
    # Virtual Box vboxnet2 - for exposing OpenStack API over external network
    auto eth2
    iface eth1 inet static
    address 192.168.100.51
    netmask 255.255.255.0
    gateway 192.168.100.1



For the remaining Installation Follow `OpenStack-Grizzly-Install-guide <https://github.com/mseknibilel/OpenStack-Grizzly-Install-Guide/blob/OVS_MultiNode/OpenStack_Grizzly_Install_Guide.rst>`_

9. Compute Node
==============

9.1. Preparing Ubuntu 13.04/12.04
------------

**Note :** Please Skip this (9.1) for Ubuntu 13.04. Ubuntu 12.10 dosent support Grizzly.

* After you install Ubuntu 12.04 Server 64bits, Go in sudo mode and don't leave it until the end of this guide::

   sudo su

* Add Grizzly repositories::

   apt-get install ubuntu-cloud-keyring python-software-properties software-properties-common python-keyring
   echo deb http://ubuntu-cloud.archive.canonical.com/ubuntu precise-updates/grizzly main >> /etc/apt/sources.list.d/grizzly.list

* Update your system::

   apt-get update
   apt-get upgrade
   apt-get dist-upgrade


9.2.Networking
------------

Configure your network by editing :: /etc/network/interfaces file

* Only one NIC on the controller node need Internet access::
  
    # NAT should be preconfigured otherwise can copy the following ...
    # Important -- NAT (eth2) is not required for Bridged Connection.
    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).

    # The loopback network interface
    auto lo
    iface lo inet loopback
    
    # The primary network interface - Virtual Box NAT connection - for Host-Only
    auto eth2
    iface eth2 inet dhcp
    
    # Virtual Box vboxnet0 - OpenStack Management Network
    auto eth0
    iface eth0 inet static
    address 10.10.10.53
    netmask 255.255.255.0
    gateway 10.10.10.1
  
    # Virtual Box vboxnet1 - VM Internal Communication Network
    auto eth1
    iface eth1 inet static
    address 10.20.20.53
    netmask 255.255.255.0
    gateway 10.20.20.1

For the remaining Installation Follow `OpenStack-Grizzly-Install-guide <https://github.com/mseknibilel/OpenStack-Grizzly-Install-Guide/blob/OVS_SingleNode/OpenStack_Grizzly_Install_Guide.rst#23-mysql--rabbitmq>`_


9.3 KVM
------------------

* your hardware does not support virtualization because it is a virtual machine it-selves ::

   apt-get install cpu-checker
   kvm-ok

* If you are using VMWare then you may get a good response. install 

* Edit /etc/nova/nova-compute.conf file again and change 'kvm' to 'qemu' leave the rest as it is::
   
   [DEFAULT]
   libvirt_type=qemu
   
* Now if you try to launch virtual machine instances they will work. 

**Note :** This is for Sand Boxing purposes only. Ideal for learning and testing, checking out OpenStack. If you want proper working you must have physical machines working.

10. Launch OpenStack Horizon Dashboard
==============
Open browser on your Host Machine and paste the following link http://192.168.100.51/horizon and you should see login page.


.. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/3.Horizon%20Dashboard/Horizon1.png

.. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/3.Horizon%20Dashboard/Horizon%202.png

.. image:: https://raw.github.com/dguitarbite/OpenStack-Grizzly-VM-SandBox-Guide/master/Images/ScreenShots/3.Horizon%20Dashboard/Horizon3.png


11. Word Of Advice.
==============

* On any condition do not restart - shutdown your VM's directly(Power Off Option), just Save the machine state if you dont have the time or paitence to shut down the nodes properly.
* Try not to modify virtual machines LAN card's mac address, it will require you to modify your network interfaces page.


12. Licensing
============

OpenStack Grizzly VM Sand Box Guide by Pranav Salunke is licensed under a Creative Commons Attribution 3.0 Unported License.

.. image:: http://i.imgur.com/4XWrp.png
To view a copy of this license, visit [ http://creativecommons.org/licenses/by/3.0/deed.en_US ].

13. Contacts
===========

Pranav Salunke: dguitarbite@gmail.com
Bilel Msekni: bilel.msekni@telecom-sudparis.eu

14. Acknowledgment
=================

This work has been supported by:

* `Aptira <http://www.aptira.com>`_
  
  .. image:: http://aptira.com/images/logo.jpg
    

15. Credits
=================

This work has been based on:

* Bilel Msekni's Grizzly install gudie [https://github.com/mseknibilel/OpenStack-Grizzly-Install-Guide]
* Bilel Msekni's Folsom install gudie [https://github.com/mseknibilel/OpenStack-Folsom-Install-guide/blob/master/OpenStack_Folsom_Install_Guide_WebVersion.rst]
* Emilien Macchi's Folsom guide [https://github.com/EmilienM/openstack-folsom-guide]
* OpenStack Documentation [http://docs.openstack.org/]
* OpenStack Quantum Install [http://docs.openstack.org/trunk/openstack-network/admin/content/ch_install.html]

16. To do
=======

This guide is just a startup. Your suggestions are always welcomed.

There are other ways of configuring your VM's. You can also use Physical Servers with Virtual Servers. Contact me for more details.
