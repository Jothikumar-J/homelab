# Offensive Security Virtual Homelab: Build & Configuration Guide

## **1. Overview & Architecture**

This guide outlines the deployment of a modular, isolated offensive security virtual homelab. This environment is designed to provide hands-on experience with real-world attack techniques and adversary tactics in a safe, controlled setting.

The lab utilizes an isolated network architecture with controlled, two-way communication. It can be dynamically configured—either as a strictly sandboxed environment for testing new exploits without risk, or as a simulated enterprise network to practice lateral movement and pivot techniques using customized firewall rules.

### **Core Networking Concepts**

- **Type-2 Hypervisor:** Software (VirtualBox) that runs on a host operating system, providing a software layer to run multiple isolated virtual machines (VMs).
    
- **Internal Network:** A VirtualBox networking mode that creates an isolated software-based network visible only to VMs connected to it, completely hidden from the host machine and the outside internet.
    
- **VLAN/Subnet Isolation:** Grouping specific machines (Attackers, Victims, Containers) into their own subnets to mimic corporate network segments.
    

## **2. Prerequisites & Tools**

|**Tool**|**Purpose**|**Core Concept / Role**|
|---|---|---|
|**VirtualBox**|Type-2 Hypervisor|The foundation software used to host and manage all virtual machines.|
|**Kali Linux**|Attacker OS|The primary machine used to launch exploits and perform network mapping.|
|**Metasploitable2**|Victim OS|An intentionally vulnerable Linux machine used as a primary target.|
|**Debian & Docker**|Container Host|Used to host vulnerable web applications (like JuiceShop) efficiently.|
|**pfSense**|Router / Firewall|An open-source firewall acting as the central gateway to route traffic between isolated subnets.|

## **3. Base Hypervisor Configuration**

### **VirtualBox Installation**

**Step 1:** Download the Windows host installer and Extension Pack from the official VirtualBox website.
![[Attachments/2026-07-07_16-18.jpeg]]

**Step 2:** Run the installer, accept the license agreement, and set the installation path.
![[Attachments/2026-07-07_16-22.jpeg]]

**Step 3:** Accept the network disconnection warning (your host internet may temporarily disconnect as virtual network drivers are installed).
![[Attachments/2026-07-07_16-22_1.jpeg]]

**Step 4:** Install required dependencies and complete the installation.
![[Attachments/2026-07-07_16-22_2.jpeg]]

### **Extension Pack Installation**

**Step 1:** Open the downloaded Extension Pack file; it will automatically open in VirtualBox.
![[Attachments/2026-07-07_16-25.jpeg]]

**Step 2:** Click **Upgrade** or **Install** and accept the license terms.
![[Attachments/2026-07-07_16-26.jpeg]]
## **4. Attacker Environment (Kali Linux)**

### **Kali Installation & Network Setup**

**Step 1:** Download the pre-built Kali Linux VirtualBox (.vbox) package from the official Offensive Security website.
![[Attachments/2026-07-07_16-44.jpeg]]

**Step 2:** Extract the archive and open the virtual machine file in VirtualBox.
![[Attachments/2026-07-07_17-14.jpeg]]

**Step 3:** Open the VM **Settings**, navigate to the **Network** tab.

**Step 4:** Keep Adapter 1 as NAT (for internet access/updates) and enable **Adapter 2**.

**Step 5:** Set Adapter 2 to **Internal Network** and name it `Attacker-network`.
![[Attachments/kali network.jpeg]]
### **Assigning a Static IP**

**Step 1:** Boot into Kali and log in using the default credentials (`kali` / `kali`).
![[Attachments/2026-07-07_17-20.jpeg]]

**Step 2:** Open a terminal and type `nmtui` to open the Network Manager Text User Interface.
![[Attachments/2026-07-07_18-05.jpeg]]

**Step 3:** Select **Edit a connection** and choose the second adapter (your internal network).
![[Attachments/2026-07-07_18-05_1.jpeg]]

**Step 4:** Change the IPv4 configuration from Automatic to **Manual**.

**Step 5:** Assign your designated network address (e.g., `10.0.10.100/24`) and save the configuration. 
![[Attachments/2026-07-07_18-17 1.jpeg]]

Verify both adapters are enabled.
![[Attachments/2026-07-07_18-22 1.jpeg]]

*note: the static ip is only configured to troubleshoot until the pfsense is configured with dhcp server*
## **5. Victim Environment Setup**

### **Metasploitable2 Integration**

**Step 1:** Download the Metasploitable2 zip file from VulnHub and extract the `.vmdk` disk file.
![[Attachments/2026-07-07_16-55.jpeg]]

**Step 2:** Create a new VM in VirtualBox, setting the OS type to Linux.
![[Attachments/2026-07-07_18-32.jpeg]]

**Step 3:** Choose **Use an existing virtual hard disk file** and select the extracted `.vmdk` file.
![[Attachments/2026-07-07_18-34.jpeg]]

**Step 4:** Open the VM **Settings**, navigate to **Network**.

**Step 5:** Change Adapter 1 from NAT to **Internal Network** and name it `Victim-network`.
![[Attachments/metasploitable network.jpeg]]

**Step 6:** Start the VM, edit the `/etc/network/interfaces` file, and assign a static IP address to isolate it properly.
![[Attachments/2026-07-07_18-49.jpeg]]

**Step 7:** Restart the networking service to apply changes.
![[Attachments/2026-07-07_18-50.jpeg]]

*note: the static ip is only configured to troubleshoot until the pfsense is configured with dhcp server*
### **Adding Additional Vulnyx Machines**

**Step 1:** Download a vulnerable machine from Vulnyx and import the `.ova` or VM file into VirtualBox.
![[Attachments/2026-07-09_10-11_1.jpeg]]

**Step 2:** Change the network adapter settings from NAT to **Internal Network** (`Victim-network`).
![[Attachments/2026-07-09_10-47.jpeg]]

**Step 3:** Because a DHCP server will be configured on pfSense for this subnet, the machine will automatically grab an IP upon boot.
![[Attachments/2026-07-09_10-50.jpeg]]

## **6. Container Lab Setup (Debian + Docker)**

_Core Concept: Docker uses **MACVLAN** to assign unique MAC and IP addresses directly to containers, making them appear as physical devices on the network. **Promiscuous Mode** must be enabled on the VM adapter so it accepts packets not explicitly addressed to its own MAC address, allowing the MACVLAN traffic to flow properly._

### **Debian Base Installation**

**Step 1:** Download the Debian `amd64` network installer ISO.
![[Attachments/2026-07-07_17-02.jpeg]]

**Step 2:** Create a new VM, allocate resources, and attach the ISO.
![[Attachments/2026-07-07_18-53.jpeg]]

**Step 3:** Set Adapter 1 to **Internal Network** and name it `Container-network`.
![[Attachments/2026-07-07_19-01.jpeg]]

**Step 4:** Set Adapter 2 to **NAT** (to download packages during setup).
![[Attachments/debian network nat 1.jpeg]]

**Step 5:** Boot the VM, select Graphical Install, and follow the standard localized setup (language, keyboard, timezone).

**Step 6:** Select the NAT adapter as your primary network interface for the installation process.
![[Attachments/2026-07-07_19-20.jpeg]]

**Step 7:** Set the hostname, root password, and create a standard user account.
![[Attachments/2026-07-07_19-20_1.jpeg]]

**Step 8:** Use guided partitioning (All files in one partition) and write changes to the disk.
![[Attachments/2026-07-07_19-24_1.jpeg]]

**Step 9:** Install the base system, skip media scanning, select a package mirror, and install the GRUB bootloader to the primary drive. Reboot and log in.
![[Attachments/2026-07-07_19-49.jpeg]]

![[Attachments/2026-07-07_19-50.jpeg]]
### **Docker & JuiceShop Deployment**

**Step 1:** Install Docker on Debian using the official repository and verify the service is running.

**Step 2:** Create a MACVLAN network in Docker to allow containers direct access to the `Container-network` subnet.
![[Attachments/2026-07-09_08-40.jpeg]]

**Step 3:** Start a JuiceShop container using Docker and attach it to the MACVLAN network.
![[Attachments/2026-07-09_10-06.jpeg]]

**Step 4:** In VirtualBox VM network settings, change the Promiscuous Mode on the internal adapter to **Allow All**.

**Step 5:** Verify you can access the JuiceShop web interface from your Kali Linux machine.
![[Attachments/2026-07-09_10-07.jpeg]]
*note: it'll be accessible only after configuring firewall rules

## **7. Centralized Routing (pfSense Setup)**

### **Installation**

**Step 1:** Download the Netgate installer ISO from the official shop (requires a free account).
![[Attachments/2026-07-07_16-53.jpeg]]

**Step 2:** Create a new VM named "pfSense" with 2 GB RAM and a 16 GB virtual hard disk.
![[Attachments/2026-07-08_08-46 1.jpeg]]

![[Attachments/2026-07-08_08-46_1 1.jpeg]]

![[Attachments/2026-07-08_08-47 1.jpeg]]

**Step 3:** Configure 4 Network Adapters:

- Adapter 1: NAT (WAN)
  | ![[Attachments/Pasted image 20260712031307.png]]
    
- Adapter 2: Internal Network -> `Attacker-network` (LAN)    
  ![[Attachments/2026-07-08_08-48 1.jpeg]]

- Adapter 3: Internal Network -> `Victim-network` (OPT1)
  | ![[Attachments/2026-07-08_08-49 1.jpeg]]
    
- Adapter 4: Internal Network -> `Container-network` (OPT2)
  | ![[Attachments/2026-07-08_08-50 1.jpeg]]
    
**Step 4:** Boot the VM, accept the terms, and proceed with the installation using standard UFS/ZFS settings. Reboot once complete.

![[Attachments/2026-07-08_08-51 1.jpeg]]

Select Install option
![[Attachments/2026-07-08_08-51_1 1.jpeg]]

Set the first interface em0 as WAN (NAT adapter)
![[Attachments/2026-07-08_08-52 1.jpeg]]

Select continue
![[Attachments/2026-07-08_08-52_1 1.jpeg]]

Set the second interface em1(attacker network) as Primary LAN
![[Attachments/2026-07-08_09-01 1.jpeg]]

Change the mode to static and set the network address and DHCP range
![[Attachments/2026-07-08_09-06 1.jpeg]]

Confirm the options
![[Attachments/2026-07-08_09-09 1.jpeg]]

Select install CE to install commuity edition
![[Attachments/2026-07-08_09-10 1.jpeg]]

Select continue to set file system type
![[Attachments/2026-07-08_09-10_1 1.jpeg]]

Select ok to configure redundancy
![[Attachments/2026-07-08_09-11 1.jpeg]]

Select the VBOX harddisk
![[Attachments/2026-07-08_09-11_1 1.jpeg]]

Confirm writing changes
![[Attachments/2026-07-08_09-12 1.jpeg]]

Select the Current stable version
![[Attachments/2026-07-08_09-13 1.jpeg]]

Select OK to continue
![[Attachments/2026-07-08_09-28 1.jpeg]]

Select reboot to boot into the machine
![[Attachments/2026-07-08_09-28_1 1.jpeg]]

Power off the vm and remove the installer iso from storage
![[Attachments/2026-07-08_10-49.jpeg]]

Restart the vm to assign interfaces
### **Interface Assignment**

**Step 1:** In the pfSense console menu, choose **Assign Interfaces** and skip VLAN setup.
![[Attachments/2026-07-08_11-08.jpeg]]

**Step 2:** Assign `em0` to WAN, `em1` to Primary LAN, `em2` to OPT1, and `em3` to OPT2.
![[Attachments/2026-07-08_11-14.jpeg]]

**Step 3:** Choose **Set interface IP address** from the menu.

**Step 4:** Set a static IPv4 address for your Attacker LAN (e.g., `10.0.10.1/24`). Enable the DHCP server if you want tools to grab IPs automatically.

**Step 5:** Repeat the IP and DHCP configuration for OPT1 (Victim) and OPT2 (Container), using distinct subnets (e.g., `10.0.20.1/24` and `10.0.30.1/24`). Disable IPv6 on all internal interfaces.
![[Attachments/2026-07-08_11-30.jpeg]]

## **8. Firewall Configuration (Simulating the Enterprise)**

_Core Concept: **Aliases** in pfSense are custom lists of IP addresses or network ports. Using them simplifies rule creation. pfSense operates on a **Deny-by-Default** basis, meaning if a rule does not explicitly allow traffic, it is dropped._

### **Configuring Inter-VLAN Routing**

**Step 1:** Open a web browser in Kali Linux and navigate to the pfSense LAN gateway IP (e.g., `10.0.10.1`). 
![[Attachments/2026-07-09_05-35.jpeg]]

**Step 2:** Log in, go to **Firewall > Aliases**, and create a new alias named `Lab_networks`. Add all your internal subnet ranges to this alias. 
![[Attachments/2026-07-09_05-40.jpeg]]

**Step 3:** Go to **Firewall > Rules** and select the **LAN** tab. **Step 4:** Create a new rule:

- **Action:** Pass
    
- **Protocol:** Any
    
- **Source:** Single Host or Alias -> `Lab_networks`
    
- **Destination:** Single Host or Alias -> `Lab_networks` 
  | ![[Attachments/2026-07-09_05-43.jpeg]]

**Step 4:** Replicate this exact rule for the **OPT1** and **OPT2** tabs to ensure total two-way visibility between all lab segments for testing.
  | ![[Attachments/2026-07-09_05-44.jpeg]]

### **Internet Access Management**

**Step 1:** To allow your Container network to download updates/Docker images, create a temporary rule in the **OPT2** tab. 

**Step 2:** Set Action to **Pass**, Source to `OPT2 subnets`, and Destination to `Any`. 
![[Attachments/2026-07-09_07-16.jpeg]]

**Step 3:** Once your containers are successfully running, disable this rule and apply changes. Verify that the Debian machine can no longer ping the outside internet, successfully sandboxing the environment.
![[Attachments/2026-07-09_10-53_1.jpeg]]

![[Attachments/2026-07-09_10-54.jpeg]]

