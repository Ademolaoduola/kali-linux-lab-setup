🔐 Cybersecurity Lab Environment Setup

Building an isolated virtual lab for penetration testing and ethical hacking practice

📌 Project Overview

This project focuses on setting up a virtual cybersecurity and penetration-testing laboratory using VirtualBox and Kali Linux.

The purpose of the lab is to create a controlled environment where cybersecurity tools, network scanning, reconnaissance, vulnerability assessment, and other security-testing activities can be performed safely and repeatedly.

The lab is configured on a private virtual network so that additional machines can be added later and used as targets for authorized security testing.

🎯 Objectives

The main objectives of this project are to:

Install and configure VirtualBox.
Install/import Kali Linux as a virtual machine.
Create a private NAT Network for the cybersecurity lab.
Configure network connectivity for Kali Linux.
Assign a consistent static IP address to the Kali VM.
Enable shared clipboard, drag-and-drop, and shared folders between host and guest.
Verify network connectivity and DNS resolution.
Take a clean VM snapshot for recovery.
Document the complete setup process, including issues encountered.
Prepare the environment for future cybersecurity projects.

🛡️ Purpose of the Lab

The lab provides an isolated and controlled environment for cybersecurity learning and authorized security testing.

It can be used for activities such as:

Network reconnaissance
Port scanning
Vulnerability assessment
Packet analysis
Web security testing
Exploitation practice
Security-tool experimentation

⚠️ Important: This laboratory must only be used for systems that you own or have explicit permission to test. Do not use the lab or its tools to attack unauthorized systems.

🏗️ Lab Architecture

Kali Linux runs as the attacking machine on a private VirtualBox NAT Network, isolated from the main host network but with outbound internet access via NAT. Additional target machines can be added to the same virtual network in future projects.

(Insert lab architecture / VirtualBox network diagram screenshot here)

⚙️ Lab Configuration

🧩 Component	⚙️ Configuration

🖥️ Host OS	Windows 10

🧰 Hypervisor	VirtualBox 7.2.4

🐉 Security OS	Kali Linux 2026.2 (upgraded from 2025.4)

🧠 Kali RAM	4096 MB

⚙️ Kali CPUs	2

🌐 Virtual Network	Custom NAT Network

📡 Network Address	10.0.0.0/24

🐧 Kali IP Address	10.0.0.2/24 (static)

🚪 Default Gateway	10.0.0.1

🌍 DNS Server	8.8.8.8

📁 Shared Folder	Host Downloads → /media/sf_Downloads

📋 Clipboard/Drag-Drop	Bidirectional

🔮 Future VM Range	10.0.0.3 – 10.0.0.99

🪜 Lab Setup Procedure

Step 1. Install 7-Zip

7-Zip was installed to extract the Kali Linux virtual-machine package.

Tool: 7-Zip

Step 2. Install VirtualBox

VirtualBox 7.2.4 was installed as the hypervisor on the host machine.

Step 3. Create the NAT Network

A dedicated NAT Network was created in VirtualBox.

Configuration:

Network Name: NatNetwork
IPv4 Prefix:  10.0.0.0/24
DHCP:         Enabled

A NAT Network was selected (rather than standard NAT) because multiple virtual machines connected to the same NAT Network can communicate with one another while also having outbound internet connectivity — allowing future attacker and target VMs to communicate within the lab.

Step 4. Import and Configure Kali Linux

The Kali Linux virtual machine was downloaded from the official Kali Linux site and imported into VirtualBox.

The VM's network adapter was configured as follows:

Adapter 1
Attached to:   NAT Network
Network:       NatNetwork
Adapter Type:  Intel PRO/1000 MT Desktop

The VM was allocated:

RAM:        4096 MB
Processors: 2

A shared folder (host Downloads folder) and bidirectional clipboard/drag-and-drop were also configured under Settings → General → Advanced and Settings → Shared Folders, to allow easy file transfer between the host and the Kali VM.

Step 5. Configure the Kali Linux Network

Kali Linux's network was configured with a consistent, static IPv4 address using nmcli:

bash
sudo nmcli connection modify "Wired connection 1" ipv4.addresses 10.0.0.2/24
sudo nmcli connection modify "Wired connection 1" ipv4.gateway 10.0.0.1
sudo nmcli connection modify "Wired connection 1" ipv4.dns 8.8.8.8
sudo nmcli connection modify "Wired connection 1" ipv4.method manual
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"

Final configuration:

IP Address:   10.0.0.2
Subnet Mask:  255.255.255.0
Gateway:      10.0.0.1
DNS:          8.8.8.8

A consistent IP address makes it easier to document the lab and reference the Kali machine in future exercises.

Step 6. Create a Clean VM Snapshot

After completing the initial configuration, a VirtualBox snapshot was created to serve as a clean baseline.

Snapshot Name: Clean Setup - 10.0.0.2 NatNetwork

If a future exercise changes or damages the VM configuration, the machine can be restored to this baseline. A second snapshot was later taken before upgrading Kali Linux, and a third after the upgrade completed successfully — giving multiple recovery points across the lab's history.

🔎 Lab Verification
✅ Test	🧾 Command	🎯 Expected Result
🌐 Check IP address	ip a	Correct Kali IP displayed (10.0.0.2/24)
📡 Test gateway	ping 10.0.0.1	Successful replies
🌍 Test Internet connectivity	ping 8.8.8.8	Successful replies
🔎 Test DNS resolution	nslookup networkwalks.com	Domain resolves
📁 Verify shared folder	ls /media/sf_Downloads	Host Downloads files listed
🧰 Verify OS version	cat /etc/os-release	Correct Kali version displayed
🔄 Verify snapshot	Restore snapshot and run ip a	Baseline configuration restored
Example Results
IP Address: 10.0.0.2/24
Gateway:    10.0.0.1
DNS:        8.8.8.8
Ping:       661 packets transmitted, 652 received, 1.36% packet loss
🐞 Problems Encountered & Solutions

Documenting problems is an important part of the project — real-world lab building rarely goes perfectly on the first try.

Problem 1. NAT Network Activation Failure (DAD Timeout)

After configuring the static IP, the network connection failed to activate with the error:

Error: Connection activation failed: IP configuration could not be reserved
(no available address, timeout, etc.)

This is a known issue on newer Kali Linux / VirtualBox v7 combinations, caused by Duplicate Address Detection (DAD) blocking interface activation.

Solution:

bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"

Note: Network interface and connection names may differ between systems. Identify your actual connection name (nmcli connection show) before running these commands.

Problem 2. IPv6 Unreachable During Package Downloads

While running a full system upgrade (apt full-upgrade), package downloads repeatedly failed with:

Cannot initiate the connection to kali.download:80 (2606:4700::6811:fdef).
- connect (101: Network is unreachable)

VirtualBox's NAT Network mode does not fully support IPv6, so apt was attempting IPv6 connections that could never succeed instead of falling back to the working IPv4 path.

Solution: Force IPv4 for all package operations:

bash
sudo apt update -o Acquire::ForceIPv4=true
sudo apt full-upgrade -o Acquire::ForceIPv4=true
Problem 3. Host Disk Full Mid-Upgrade

During a full Kali Linux upgrade (2025.4 → 2026.2, ~2,200+ packages), VirtualBox threw a non-fatal error and paused the VM:

Error ID: BLKCACHE_IOERR
The I/O cache encountered an error while updating data in medium "ahci-0-0"
(rc=VERR_DISK_FULL).

The host machine's physical drive had run critically low on free space (under 500 MB), leaving no room for the virtual disk to grow as new packages were installed.

Solution:

Freed up space on the host drive (Disk Cleanup, removing unused installers/ISOs, uninstalling unused software) — approx. 8 GB recovered.
Right-clicked the paused VM in VirtualBox Manager and selected Resume.
The upgrade continued and completed successfully from where it left off, with no data loss.

Lesson: VirtualBox's virtual disk grows dynamically as packages are installed — always ensure the host machine (not just the guest) has sufficient free space before running large upgrades.

💡 What I Learned

Through this project, I learned how to build, configure, and troubleshoot a virtual environment for cybersecurity practice from the ground up.

1. NAT vs NAT Network

A standard NAT configuration and a NAT Network serve different purposes. A NAT Network allows multiple VMs connected to the same virtual network to communicate with one another while still providing network address translation for external connectivity — making it the right choice for a multi-machine cybersecurity lab.

2. Virtual Machine Networking

I learned how VirtualBox virtual network adapters connect VMs to different network types, and how DHCP-assigned addresses differ from manually configured static addresses.

3. Static IP Configuration via nmcli

I learned how to configure and verify IPv4 addressing, subnet masks, gateways, and DNS settings in Kali Linux directly from the command line.

4. Diagnosing Real Network Failures

I learned how to distinguish between different classes of connectivity failures — a broken NAT Network configuration, a DAD timeout bug, and an IPv6-specific unreachability issue each look similar on the surface but require different fixes.

5. Host Resource Management

I learned that a VM's stability depends on host resources too — disk space, in particular, is easy to overlook until it causes a hard failure mid-operation.

6. VM Snapshots

I learned that a clean snapshot should be created before performing risky or experimental activities (including OS upgrades), providing a known-good recovery point.

7. Documentation

I learned that documenting commands, configuration, screenshots, problems, and solutions — including the mistakes and dead ends — is an essential part of a professional cybersecurity project.

🔐 Security & Ethical Use

This laboratory is intended strictly for educational purposes. All testing activities are performed exclusively within the isolated lab environment against systems owned by the lab operator.

🔗 Tools & Resources
7-Zip: https://7-zip.org/download.html
VirtualBox: https://virtualbox.org/wiki/Downloads
Kali Linux: https://kali.org/get-kali
👤 Author

Ademola Oduola Field Service Engineer, Globacom | Transitioning into Network Security & SOC Cybersecurity & Ethical Hacking Program — NetworkWalks

📌 Project Information

Program Name: Cybersecurity at NetworkWalks | Week: 01 | Project: Cybersecurity & Pentesting Lab Setup | Repository: GitHub
