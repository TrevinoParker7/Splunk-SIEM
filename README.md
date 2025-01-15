# Splunk Lab

## Objective

This lab is all about getting hands-on with Active Directory and security monitoring. I've set up Windows Server 2022 AD domain controller and a Ubuntu Splunk server, then used Kali Linux to play the role of an attacker in the network. The goal is to see what kind of events get generated from these attacks and learn how to configure and use Splunk to catch them. I aim to dive into domain environments and sharpen threat detection skills.

This is by no means a comprehensive step-by-step of my process, but rather a few key processes that are interesting or will be useful for my future reference.
### Skills Learned

- Configuring and managing Active Directory domain environments
- Setting up and utilizing Splunk for security event monitoring and analysis
- Simulating bruteforce attack using Kali Linux
- Performing cybersecurity tests using Atomic Red Team.
- Identifying and interpreting security events generated from attacks via Splunk
- Analyzing and correlating security events in Splunk with the MITRE ATT&CK framework.

### Tools Used

- Windows Server 2022 (for Active Directory domain controller)
- Ubuntu Server (for Splunk server)
- Splunk (for security event monitoring and analysis)
- Kali Linux (for simulating bruteforce attack)
- Windows 10 Client (for interacting with the domain environment)
- Atomic Red Team (for simulating various attacks mapped to the MITRE ATT&CK framework)
- VirtualBox (for virtualising the lab environment)

## Network Diagram

[![NetworkDiagram-ActiveDirectoryProject drawio](https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/f44ce545-f677-4006-8a65-b447b6d2978f)](https://imgur.com/ib3A0G1)

## Table of Contents

  1. [Objective](#objective)
  2. [Skills Learned](#skills-learned)
  3. [Tools Used](#tools-used)
  4. [Network Diagram](#network-diagram)
  5. [Setting up Splunk Server and Forwarders](#setting-up-splunk-server-and-forwarders)
      - [Setting Static IP Address and Default Route](#setting-static-ip-address-and-default-route)
      - [Install Splunk Enterprise](#install-splunk-enterprise)
      - [Setting Up Splunk Forwarder](#setting-up-splunk-forwarder)
      - [Installing Sysmon](#installing-sysmon)
      - [Configuring Inputs for Splunk Forwarder](#configuring-inputs-for-splunk-forwarder)
      - [Restarting Splunk Forwarder Service](#restarting-splunk-forwarder-service)
      - [Connecting to Splunk Web Interface](#connecting-to-splunk-web-interface)
  6. [Setting up Active Directory and Provisioning Users](#setting-up-active-directory-and-provisioning-users)
  7. [Performing a Brute Force Attack on Target_PC and Reviewing Events via Splunk](#performing-a-brute-force-attack-on-target_pc-and-reviewing-events-via-splunk)
  8. [Installing Atomic Red Team, Performing a Test, and Reviewing Events in Splunk](#installing-atomic-red-team-performing-a-test-and-reviewing-events-in-splunk)
  9. [Conclusion](#conclusion)

---

## Setting up Splunk server and Forwarders

### Setting Static IP Address and Default Route:

- Configured a static IP address for the Splunk server and defined a default route with the gateway 192.168.10.10.
```sudo nano /etc/netplan/00-installer-config.yaml```

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/99ee31d7-40c0-4739-8581-c6d72613b066" alt="Splunk server running in Oracle VirtualBox" width="635" />

Apply the changes:
```sudo netplan apply```

### Install Splunk Enterprise:

- Installed Splunk Enterprise on the Splunk server and configure it to start at boot.

```tsoc@splunk:/opt/splunk/bin$ sudo ./splunk enable boot-start -user splunk```

### Setting Up Splunk Forwarder:
Installed and configured Splunk Forwarder on ADDC01 and target-PC (Windows 10) to send data to the Splunk server as a receiving indexer.

![2024-06-28 22_10_10-ADDC01  Running  - Oracle VM VirtualBox](https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/54bdd7fe-3954-4c2e-8613-6ec049293241)

### Installing Sysmon:

Installed Sysmon to enhance event logging capabilities.

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/306160cf-69bd-4322-b2ee-50dbd73b66eb" width="700" />

### Configuring Inputs for Splunk Forwarder:

Created an inputs.conf file in C:\Program Files\SplunkUniversalForwarder\etc\system\local on ADDC01 and target-PC, configuring settings as per.

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/7b973c10-11a6-4620-949e-bc233cf55736" width="700" />
<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/dc816365-9ca2-4435-af25-8d3e7e35aece" width="700" />

### Restarting Splunk Forwarder Service:

Restarted the Splunk Forwarder service on ADDC01 and set to log on as local system account.

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/4aa87cdf-4e3e-49a8-a2fe-a0eae7de8f4f" width="700" />

### Connecting to Splunk Web Interface:

Accessed the Splunk server's web interface at port 8000, then created an index named endpoint as specified in the inputs.conf file. I repeated this process for both ADDC01 and target-PC to ensure the Splunk server receives events from both sources.

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/fa26209e-7ad2-4136-a5f5-7ec7fba9c232" width="700" />

---

## Setting up Active Directory and provisioning users

Install Active Directory Domain Services on ADDC01

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/80c01684-787a-4639-8f61-a3a8ab7270a7" width="700" />

Promote ADDC01 to Domain Controller

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/39653cdf-11f8-4527-ac06-588629e50e05" width="700" />

I joined target_PC to the domain and tinkered around with users, groups and permissions. I used this script from Josh Madakor's video to create around 1000 users.

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/64f8e66c-9404-40de-bb38-5e15b0b4d69a" width="700" />
<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/5501aaa0-120e-4540-8630-00087fb0f77d" width="700" />

---

## Performing a Brute force attack on target_PC and reviewing events via Splunk

I used crowbar to launch a brute force dictionary attack on target_PC from the Kali Linux machine. I had enabled RDP on target_PC beforehand so this attack would be feasable.

![2024-06-29 22_33_32-kali-linux-2024 1-virtualbox-amd64  Running  - Oracle VM VirtualBox](https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/bc6e4d08-5653-45e0-a549-00d88246d945)

After running the attack, we can see that Splunk recorded 42 events with event code 4265, which indicates failed login attempts. This corresponds to the 22 passwords in the wordlist I used for the attack, which was run twice.

Among these, there are two events with event code 4264, representing successful login attempts. This outcome is expected since one of the passwords in the wordlist was of course correct.

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/2d796524-390b-449e-86cc-615b9cad0b3a" width="700" />
<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/643611dd-b8e9-41d4-9784-60ffa72060ae" width="600" />

Here we can see that the attack indeed came from the Kali machine at 192.168.10.250

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/2770fcbd-b06a-4ba2-8b40-10d6a2ca347f" width="600" />

---

## Installing Atomic Red Team, Performing a Test, and Reviewing Events in Splunk

Atomic Red Team is an open-source project that offers a collection of tests to simulate cyberattacks based on the MITRE ATT&CK framework.

Before installing Atomic Red Team (ATR) on target_PC, I excluded the C: drive (where ATR will be installed) from Microsoft Defender Anti-Virus scans. Note: This exclusion is not recommended for normal circumstances.

To allow PowerShell scripts to run without restrictions for the current user, I used the command:
```Set-ExecutionPolicy Bypass -Scope CurrentUser```

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/3df29cb5-77f4-4d34-8db9-e112b23f2c04" width="800" />
<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/69ef5c78-2b1e-4ac4-82d3-2b66eef8e699" width="700" />

Next, I installed ATR using the following commands:

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/853de60c-1d63-4a9b-b9a1-43e50401b827" width="800" />

Now we can view all the tests available in Atomic Red Team. Each test is named after the corresponding MITRE ATT&CK technique. For example, I ran the T1136.001 test, which corresponds to the "Create Account: Local Account" persistence technique in MITRE ATT&CK.

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/189a3b58-e48d-426a-bfca-9e2ad1da5d1e" width="700" />
<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/febce88b-103b-4330-ba3a-80458c62cd0d" width="500" />

Running the test created a user called NewLocalUser, added it to the local administrators group, and finally deleted the user.

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/9426fff8-1b57-4ae5-8d69-5aa5f1b00959" width="700" />

We see these events in Splunk.

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/d9d4a3d4-a2ca-4542-bda1-e31a7db4472a" width="700" />

Here are the corresponding event codes:
- 4798: A user's local group membership was enumerated.
- 4738: A user account was changed.
- 4720: A user account was created.
- 4722: A user account was enabled.
- 4724: An attempt was made to reset an account's password.
- 4726: A user account was deleted.

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/06f90944-7158-4630-b824-e3f742759c78" width="700" />

Below is the final event showing "NewLocalUser" being deleted

<img src="https://github.com/PaulMiguelSec/Active-Directory-Lab/assets/174075754/feb94ab2-9222-4a1a-99b4-020f09bfda07" width="700" />

---

# Conclusion

This lab has been a fantastic learning experience for me. I've set up Splunk Enterprise on Ubuntu, deployed Splunk forwarders, performed tests with Atomic Red Team, and analyzed results in Splunk following the MITRE ATT&CK framework. I also used Kali Linux to simulate a brute force attack and reviewed the outcome in Splunk.

Through setting all this up, I've sharpened my skills in virtualization with VirtualBox, general Windows and Linux processes and learned the essentials of setting up Active Directory on Windows Server.

I'm excited to keep building on this lab. My next steps include creating Splunk alerts, tweaking group policies, and just generally tinkering around in the domain environment. There's always more to learn, and I'm looking forward to it!
