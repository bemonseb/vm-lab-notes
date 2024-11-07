# Lab 02 - Virtual Networking

In this lab, we'll set up isolated virtual networks using libvirt and test how VMs can communicate within and across these networks.

Libvirt's isolated virtual networks are designed to restrict network access, allowing communication only among VMs on the same network. Unlike the default NAT-mode virtual network, which provides access to the host machine and potentially the internet through Network Address Translation (NAT), isolated networks don't route traffic outside of their network. For more details on libvirt's virtual networking modes, you can check the [libvirt wiki on NAT networks](https://wiki.libvirt.org/VirtualNetworking.html#network-address-translation-nat) and [isolated networks](https://wiki.libvirt.org/VirtualNetworking.html#isolated-mode).

## Steps

### Step 01 - Create Isolated Networks

In this step, we’ll create two isolated virtual networks: `iso-net-1` and `iso-net-2`.

#### Creating `iso-net-1` with `virsh` and `vim`

1. **Define the Network**:
   - Use `vim` to create the XML configuration file for `iso-net-1`:
     ```xml
     <network>
       <name>iso-net-1</name>
       <bridge name='virbr1' stp='on' delay='0'/>
       <ip address='192.168.1.1' netmask='255.255.255.0'>
         <dhcp>
           <range start='192.168.1.10' end='192.168.1.254'/>
         </dhcp>
       </ip>
     </network>
     ```
   - Save this configuration as `iso-net-1.xml`.

2. **Define, Start, and Autostart the Network**:
   - Define the network:
     ```bash
     sudo virsh net-define iso-net-1.xml
     ```
   - Start the network:
     ```bash
     sudo virsh net-start iso-net-1
     ```
   - Enable autostart:
     ```bash
     sudo virsh net-autostart iso-net-1
     ```

#### Creating `iso-net-2` using `virt-manager`

1. **Open virt-manager**:
   - Navigate to **Edit > Connection Details > Virtual Networks**.
   - Click **Add Network** and follow the prompts to create `iso-net-2` with the following settings:
     - **Network Name**: `iso-net-2`
     - **Subnet**: `192.168.2.0/24`
     - **DHCP Range**: `192.168.2.10` to `192.168.2.254`
   - Save and start the network.
   
2. **Set Autostart**:
   - In `virt-manager`, right-click on `iso-net-2` and enable **Autostart**.

### Step 02 - Create VMs and Connect to Isolated Networks

Now, we will create VMs and connect them to the isolated networks.

1. **Clone `template-fedora-workstation`**:
   - In `virt-manager`, clone `template-fedora-workstation` to create two new VMs: **alpha** and **bravo**.

2. **Connect VMs to `iso-net-1`**:
   - Open each VM’s network settings in `virt-manager` and connect **alpha** and **bravo** to the `iso-net-1` network.

3. **Set Hostnames**:
   - After cloning, change the hostname on each VM:
     - On alpha:
       ```bash
       sudo hostnamectl set-hostname alpha
       ```
     - On bravo:
       ```bash
       sudo hostnamectl set-hostname bravo
       ```

4. **Connect `server-0` to `iso-net-2`**:
   - Connect the existing `server-0` VM to `iso-net-2` using `virt-manager`.
   - Update the static IP address:
     ```bash
     sudo nmcli con mod <connection_name> ipv4.addresses "192.168.2.2/24" ipv4.gateway "192.168.2.1" ipv4.dns "192.168.2.1" ipv4.method manual
     ```
   - Bring up the connection:
     ```bash
     sudo nmcli con up <connection_name>
     ```
     - Replace `<connection_name>` with the appropriate network connection name.

5. **Add Static Hostname Entry for `server-0`**:
   - Edit `iso-net-2` to add a static hostname entry for `server-0` by modifying the network definition:
     ```bash
     sudo virsh net-edit iso-net-2
     ```
   - In the `<dns>` section, add:
     ```xml
     <host ip='192.168.2.2'>
       <hostname>server-0</hostname>
     </host>
     ```
   - Restart `iso-net-2` for changes to take effect:
     ```bash
     sudo virsh net-destroy iso-net-2
     ```
     ```bash
     sudo virsh net-start iso-net-2
     ```

### Step 03 - Test Isolation

With the VMs connected to their respective isolated networks, we’ll test the network isolation.

1. **Ping Test on `iso-net-1`**:
   - On `alpha`, ping `bravo`:
     ```bash
     ping -c 4 bravo
     ```
   - This should be successful since both VMs are on the same `iso-net-1`.

2. **Isolation Verification**:
   - Try to ping `server-0` from `alpha` and `bravo`:
     ```bash
     ping -c 4 192.168.2.2
     ```
   - This should fail since `iso-net-1` and `iso-net-2` are isolated from each other.

3. **Connect `bravo` to `iso-net-2` and Test**:
   - In `virt-manager`, change `bravo`’s network connection to `iso-net-2`.
   - Once connected, test connectivity:
     - On `bravo`, ping `server-0`:
       ```bash
       ping -c 4 server-0
       ```
     - On `server-0`, ping `bravo`:
       ```bash
       ping -c 4 bravo
       ```
   - The ping commands should now succeed, showing that `bravo` and `server-0` can communicate within `iso-net-2`.

### Step 04 - Remove Isolated Network IP Address

To make the isolated networks function more like TCP/IP Layer 2 switches (essentially switching packets without IP routing), we can remove the IP addresses assigned to `iso-net-1` and `iso-net-2`.

1. **Remove IPs from Network Definitions**:
   - Edit the network XML files for `iso-net-1` and `iso-net-2`:
     ```bash
     sudo virsh net-edit iso-net-1
     ```
     ```bash
     sudo virsh net-edit iso-net-2
     ```
   - Remove the `<ip>` sections entirely from each file.
   
2. **Restart Networks**:
   - Restart `iso-net-1`:
     ```bash
     sudo virsh net-destroy iso-net-1
     ```
     ```bash
     sudo virsh net-start iso-net-1
     ```
   - Restart `iso-net-2`:
     ```bash
     sudo virsh net-destroy iso-net-2
     ```
     ```bash
     sudo virsh net-start iso-net-2
     ```
   - With the IPs removed, the networks will now function as Layer 2 switches, meaning they won’t route IP traffic but will pass frames between VMs connected to the same network.

This concludes the setup and testing for isolated virtual networks in this lab.
