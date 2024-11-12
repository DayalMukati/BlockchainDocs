---
cover: >-
  https://images.unsplash.com/photo-1488590528505-98d2b5aba04b?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHwyfHx0ZWNofGVufDB8fHx8MTY3Njg3MDU0OA&ixlib=rb-4.0.3&q=80
coverY: 0
---

# 👲 NoMachine Installation

## Introduction <a href="#introduction" id="introduction"></a>

Welcome to the guide for NoMachine free edition v.8 or later. This guide document is intended to provide you with step-by-step instructions on how to install, update or remove the NoMachine software on your system, initiate your first connection to the remote computer and configure settings via the User Interface (UI).

## How to set-up NoMachine <a href="#how-to-set-up-nomachine" id="how-to-set-up-nomachine"></a>

#### 1. Install, Update or Remove NoMachine <a href="#head-0" id="head-0"></a>

***

This section will guide you through all the steps to install, update and uninstall NoMachine in your environment. Please consult the appropriate section for the operating system you are installing on. There are two ways to update your installations: via the automatic updates from our repositories or updating manually using packages.

#### 1.1. Prerequisites <a href="#head-0" id="head-0"></a>

Supported Operating Systems\
Windows 32-bit/64-bit Vista/7/8/8.1/10/11\
Windows Server 2008/2012/2016/2019

Mac OS X Intel 64-bit 10.9 to 10.11/macOS Intel 10.12 to 12.x

Linux 32-bit and 64-bit\
RHEL 6.0 to RHEL 9\
CentOS 6.0 to CentOS 8.5\
CentOS Stream 8 to CentOS Stream 9\
SLED 11 to SLED 15\
SLES 11 to SLES 15\
openSUSE 11.x to openSUSE 15.x\
Fedora 10 to Fedora 36\
Debian 5 to Debian 11\
Ubuntu 8.04 to Ubuntu 22.04

Raspberry Pi 2/3/4 ARMv6/ARMv7/ARMv8

Software requirements\
A desktop environment must already be installed. This applies also to headless Linux machines.

Hardware requirements\
The software is designed to work on computers with minimal HW requirements. Although the software may work with inferior CPUs or reduced RAM, for best performance NoMachine recommends you match the listed requirements.

\- Intel Core2 Duo or AMD Athlon Dual-Core or equivalent\
\- 1 GB RAM\
\- Network connection (either a LAN, or Internet)

Size required on disk:\
Windows 130 MB\
Linux 110 MB\
Mac 100 MB\
ARMv6 95 MB\
ARMv7 85 MB\
ARMv8 105 MB

Compatibility Between Client/Server Versions\
Compatibility between NoMachine v.8 and client/server v. 7/v. 6 is fully preserved. However, it's strongly suggested to upgrade both client and server side to the last available version to benefit from the new features.

#### 1.2. Windows Installation <a href="#head-0" id="head-0"></a>

**INSTALL**\
Download the package for Windows from the NoMachine web site and install it by double-clicking on the icon of the executable: a setup wizard will take you through the installation. Accept to reboot of the machine, this is mandatory for completing the installation

Provide your administrative credentials if requested to authorize the installation.

**Step 1**: Welcome to the installer! Click on Next to start the installation.\
**Step 2**: Read and accept the License Agreement and click Next to go on.\
**Step 3**: Click Next to proceed and please wait while Setup completes the installation … You can then select where to install NoMachine or let the default location as it is. What's important to remember is that all successive updates will take place there. To change the place of installation you will have to uninstall the software and proceed with a fresh install.

**Reboot is mandatory** to complete the installation.

**Step 4**: Accept to reboot your machine to complete the installation and exit the Installer.



## Connect to NoMachine <a href="#connect-to-nomachine" id="connect-to-nomachine"></a>

#### Connect from Your Computer to a Computer with NoMachine Installed <a href="#head-70" id="head-70"></a>

**Pre-requisite** are that any of the NoMachine server products is installed on the remote computer, that you know the IP of that computer and that you have a system account there.

If you want to connect to a computer, you will need to know its IP address. To know the IP address of a specific NoMachine host, open the NoMachine User Interface on that computer, and you will see a 'Welcome' panel like this one:

<figure><img src="https://kb.nomachine.com/images_articles/documents_v8/nm-free-8/welcome_IP_external_free.jpg" alt=""><figcaption></figcaption></figure>

If you want to connect to this computer over the Internet, write down the external IP address and its port number. In our example: nx://151.1.192.128:29382.

If you are connecting on a local network instead, you just need the private IP, 192.168.2.29 in this case. Port is 4000 by default.

You will need this information when creating the connection to this host from your client device. Follow these quick steps:

Step 1: Go now to the client device from which you want to connect to this host. Install NoMachine, which can make connections as well as accept incoming connections, or NoMachine Enterprise Client.

Step 2: Run NoMachine User Interface (UI) from the programs or applications menu.

Step 3: Click on 'Add' and provide the relevant information: a name for the connection, the IP address and port number of the computer you want to connect to (you made a note of it earlier):

<figure><img src="https://kb.nomachine.com/images_articles/documents_v8/nm-free-8/address.jpg" alt=""><figcaption></figcaption></figure>

Step 4: Any available computers on your network, or connections you created appear in the 'Machines' view of the UI. For example, if you just created a connection called 'Testdrive':

<figure><img src="https://kb.nomachine.com/images_articles/documents_v8/nm-free-8/machines_settings.jpg" alt=""><figcaption></figcaption></figure>

Launch the connection by double-clicking on its icon. You will be requested to provide the username and password of a valid account for the remote computer.
