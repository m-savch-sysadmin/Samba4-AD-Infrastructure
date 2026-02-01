# Samba4 Active Directory Domain Controller Deployment Guide
**This repository contains a complete step-by-step guide to deploying a Samba4 AD DC on Ubuntu Server and joining a **Windows 10 Pro client**.**

# Phase 1: Preparing the OS environment and installing core Active Directory components.
**Before installing Samba, we must ensure the network and hostname are correctly configured.**

** 1. Set temporary DNS for package installation **
  
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

** 2. Fix hostname resolution **

echo "127.0.0.1 localhost ubuntu-serwer" | sudo tee /etc/hosts

** 3. Install required packages **

sudo apt update
sudo apt install -y samba krb5-config krb5-user winbind libpam-winbind libnss-winbind

<img width="939" height="762" alt="Screenshot 2026-02-01 115746" src="https://github.com/user-attachments/assets/c8cf4b69-e610-4ac4-9406-0480e8b2d15e" />

# Phase 2: Activating the AD DC service and verifying Kerberos authentication.
**This is the process of creating the Active Directory database.**


sudo samba-tool domain provision --use-rfc2307 --interactive

Configuration used: (for my own project)

Realm: HIGHSEC.LAN

Domain: HIGHSEC

Server Role: dc

DNS Backend: SAMBA_INTERNAL

DNS forwarder: 8.8.8.8

<img width="944" height="1013" alt="Screenshot 2026-02-01 103444" src="https://github.com/user-attachments/assets/8f349d18-63f4-4d61-b9fa-e460e228f6fc" />

# Phase 3: Service Configuration & Kerberos Fix
**After provisioning, we link the Kerberos configuration and start the AD DC service.**

 Link the Samba-generated Kerberos config

sudo ln -sf /var/lib/samba/private/krb5.conf /etc/krb5.conf

 Unmask and enable the service

sudo systemctl unmask samba-ad-dc

sudo systemctl enable --now samba-ad-dc

Verify Administrator ticket

kinit administrator@HIGHSEC.LAN

<img width="961" height="269" alt="Screenshot 2026-02-01 123540" src="https://github.com/user-attachments/assets/09f80454-69c2-4ff4-bc99-7decf4c89279" />


The Issue: kinit failed to locate the KDC for HIGHSEC.LAN because the system was using external DNS (8.8.8.8) instead of the local Samba instance.

The Fix:

 1. Force DNS to use local Samba instance

echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf

 2. Verify Kerberos configuration

cat /etc/krb5.conf

 3. Test authentication

kinit administrator@HIGHSEC.LAN




<img width="927" height="529" alt="Screenshot 2026-02-01 123827" src="https://github.com/user-attachments/assets/830bc2d5-5f04-4309-a921-5262634f2fb3" />


# Phase 4: Windows Client Network Configuratio

Step 1: IPv4 and DNS Setup

Open Network Connections (shortcut: ncpa.cpl).

Right-click on the Ethernet adapter and select Properties.

Select Internet Protocol Version 4 (TCP/IPv4) and click Properties.

Enter the following static configuration:

IP address: 10.0.0.50

Subnet mask: 255.255.255.0

Preferred DNS server: 10.0.0.10 (The IP of the Ubuntu Domain Controller).


<img width="913" height="647" alt="Screenshot 2026-02-01 180527" src="https://github.com/user-attachments/assets/b2a002bd-e782-44d3-8f78-8f48c90ca53f" />


Step 2: Connectivity Test

Open Command Prompt (CMD) and verify name resolution:


<img width="932" height="710" alt="Screenshot 2026-02-01 180609" src="https://github.com/user-attachments/assets/c6ccf4e4-a3a1-4937-b4a3-e82f688068c7" />


###      Join the Domain

Open System Properties: Win + R, sysdm.cpl and Enter.

Change Settings: 

Computer name: change PC1.

Member of: Choose Domain and write highsec.lan.

Press Ok.



<img width="952" height="738" alt="Screenshot 2026-02-01 180850" src="https://github.com/user-attachments/assets/c9e41319-69a9-4a6b-8a3a-e4c014f0efb2" />



# SUCCESS: 


<img width="779" height="510" alt="Screenshot 2026-02-01 180953" src="https://github.com/user-attachments/assets/31ded0ed-013e-405c-9906-57c853be6cf7" />


And finally "restart now".








