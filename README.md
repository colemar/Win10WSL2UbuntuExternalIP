# Recipe: Persistent Network Configuration in WSL 2 using Hyper-V Virtual Switch

## Problem Description
Connecting to services running in WSL 2 from external sources can be challenging due to the instances being on a different network. This guide offers a solution to replace the internal virtual switch of WSL 2 with an external version in Windows 20H2 (WSL 2.0) and configure it for better networking control.

**Unfortunately the [original recipe](https://github.com/Unsigned-Char/WSL2HyperVSwitch) doesn't work in Windows 10 (at least for me) because setting `networkingMode = bridged` causes an error at WSL boot.**

## Solution Overview
This recipe uses a Hyper-V virtual switch to bridge the WSL 2 network, providing improved control and visibility of Windows' network adapters within Ubuntu. The configuration supports both dynamic and static IP addressing, eliminating the need for port forwarding and simplifying network setup.

### Steps
1. **Enable Hyper-V and Management PowerShell Features:**
   - Open PowerShell as administrator and run:
     ```powershell
     Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V
     Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-Management-PowerShell
     ```

3. **Change WSL Virtual Switch to External in Hyper-V:**
   - Open PowerShell as administrator and run:
     ```powershell
     Set-VMSwitch WSL -SwitchType External
     ```

4. **Modify the WSL Configuration:**
   - **Do not** use `networkingMode = bridged` in the `.wslconfig` file in your user profile directory (`$env:USERPROFILE/.wslconfig`).
   - Instead, use `networkingMode = NAT` (the default) or remove it altogether.

5. **Enable systemd in the WSL Distribution and flush ip configuration at boot:**
   - Edit the `/etc/wsl.conf` file in your WSL distribution and add the following lines:
     ```plaintext
     [boot]
     systemd=true
     # remove NAT related ip configuration applied by WSL at boot
     command = ip address flush dev eth0
     [network]
     generateResolvConf = false
     ```

6. **Configure Network Addressing:**
   - For dynamic address configuration, ensure the following is present in `/etc/systemd/network/10-eth0.network`:
     ```plaintext
     [Match]
     Name=eth0
     [Network]
     DHCP=yes
     ```
   - For static address configuration, use:
     ```plaintext
     [Match]
     Name=eth0
     [Network]
     Address=192.168.x.xx/24
     Gateway=192.168.x.x
     DNS=192.168.x.x
     ```

7. **Link systemd Resolv.conf:**
   - Create a symbolic link to link resolv.conf from systemd:
     ```bash
     ln -sfv /run/systemd/resolve/resolv.conf /etc/resolv.conf
     ```

8. **Verification:**
   - Restart the WSL 2 instance and verify the network configuration with:
     ```bash
     ip addr show eth0
     ip route
     ```

