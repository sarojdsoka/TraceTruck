---

# Project 6: Configuring and Testing Firewall Rules Using UFW

## 1. Introduction
The goal of this project is to configure a firewall on a Linux virtual machine (VM) using UFW (Uncomplicated Firewall) to secure network traffic by allowing only SSH (port 22) and HTTP (port 80) traffic while blocking all other incoming traffic, such as ping or unauthorized port access. The project includes simulating blocked traffic (ping and port scans) and verifying the firewall rules using nmap and system logs. This exercise demonstrates fundamental network security practices for controlling traffic to a Linux system.

## 2. Abstract
This project implements a firewall on an Ubuntu Linux VM using UFW to restrict incoming network traffic to only SSH and HTTP protocols, ensuring secure access for remote administration and web services while blocking unauthorized access. By simulating a ping and port scan from an external machine, the project tests the firewall’s effectiveness and verifies the results through nmap scans and UFW logs. This is significant for learning how to protect systems from unwanted network activity, a critical skill in system administration and cybersecurity.

## 3. Tools Used
- **Operating System**: Kali  Linux 2025.3
- **Tools**:
  - **UFW**: Uncomplicated Firewall for managing firewall rules.
  - **Nmap**: Network exploration tool for scanning open ports.
  - **Apache2**: Web server to test HTTP traffic (optional).
  - **Netcat (nc)**: For testing blocked ports.
- **Packages**:
  - `ufw`: Firewall management tool (pre-installed on Ubuntu).
  - `nmap`: For port scanning.
  - `apache2`: For hosting a test web server.
  - `netcat`: For testing connections to blocked ports.
- **Environment**: A Linux VM (e.g., VirtualBox, VMware, or cloud-based like AWS/GCP) and a second machine/VM for external testing.
- **Access**: Root or sudo privileges required for firewall configuration and log access.

## 4. Steps Involved in Building the Project
Below is a detailed step-by-step guide to configure and test the firewall rules.

### Step 1: Set Up the Linux VM
- Launch an Ubuntu 22.04 VM with a network interface (e.g., NAT or bridged adapter for external access).
- Update the system to ensure all packages are current:
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```

### Step 2: Install and Verify UFW
- Confirm UFW is installed (pre-installed on Ubuntu):
  ```bash
  sudo apt install ufw -y
  ```
- Check UFW status:
  ```bash
  sudo ufw status
  ```
  Expected output: `Status: inactive`

### Step 3: Configure Firewall Rules
- Reset UFW to clear any existing rules:
  ```bash
  sudo ufw reset
  ```
- Allow SSH (port 22) to ensure remote access:
  ```bash
  sudo ufw allow ssh
  ```
- Allow HTTP (port 80) for web traffic:
  ```bash
  sudo ufw allow http
  ```
- Set default policy to deny all incoming traffic:
  ```bash
  sudo ufw default deny incoming
  ```
- Allow all outgoing traffic for usability:
  ```bash
  sudo ufw default allow outgoing
  ```
- Enable UFW to activate the firewall:
  ```bash
  sudo ufw enable
  ```
- Verify the rules:
  ```bash
  sudo ufw status
  ```

### Step 4: Install Apache2 for HTTP Testing (Optional)
- Install and start Apache2 to test HTTP traffic:
  ```bash
  sudo apt install apache2 -y
  sudo systemctl start apache2
  sudo systemctl enable apache2
  ```
- Verify Apache is running:
  ```bash
  curl http://localhost
  ```

### Step 5: Simulate Blocked Traffic
- **Block ping (ICMP)**:
  - From a second machine/VM, ping the VM’s IP:
    ```bash
    ping <VM_IP_ADDRESS>
    ```
    Expected: No response due to UFW’s default ICMP block.
- **Simulate a port scan**:
  - Install nmap on the second machine:
    ```bash
    sudo apt install nmap -y
    ```
  - Run a port scan against the VM:
    ```bash
    nmap <VM_IP_ADDRESS>
    ```

### Step 6: Verify Rules and Logs
- Enable UFW logging:
  ```bash
  sudo ufw logging on
  ```
- Check UFW logs for blocked traffic:
  ```bash
  sudo tail -f /var/log/ufw.log
  ```
- Test allowed traffic:
  - SSH to the VM:
    ```bash
    ssh <user>@<VM_IP_ADDRESS>
    ```
  - Access the web server:
    ```bash
    curl http://<VM_IP_ADDRESS>
    ```
  - Test a blocked port (e.g., telnet, port 23):
    ```bash
    nc -zv <VM_IP_ADDRESS> 23
    ```

### Step 7: Perform Before-and-After Testing
- **Before enabling UFW**:
  - Run nmap:
    ```bash
    nmap <VM_IP_ADDRESS>
    ```
  - Ping the VM:
    ```bash
    ping <VM_IP_ADDRESS>
    ```
- **After enabling UFW**:
  - Repeat the nmap scan and ping test to compare results.

## 5. Screenshots/Outputs
Below are sample outputs from key steps (replace `<VM_IP_ADDRESS>` with your VM’s IP, e.g., `192.168.1.100`).

### UFW Status After Configuration
```bash
$ sudo ufw status
Status: active
To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
```

### Nmap Scan Before UFW
```bash
$ nmap 192.168.1.100
Starting Nmap 7.80 ( https://nmap.org ) at 2025-09-06 18:30 IST
Nmap scan report for 192.168.1.100
Host is up (0.00045s latency).
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
445/tcp open  microsoft-ds
Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
```

### Nmap Scan After UFW
```bash
$ nmap 192.168.1.100
Starting Nmap 7.80 ( https://nmap.org ) at 2025-09-06 18:35 IST
Nmap scan report for 192.168.1.100
Host is up (0.00050s latency).
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
Nmap done: 1 IP address (1 host up) scanned in 0.06 seconds
```

### Ping Test Before UFW
```bash
$ ping 192.168.1.100
PING 192.168.1.100 (192.168.1.100) 56(84) bytes of data.
64 bytes from 192.168.1.100: icmp_seq=1 ttl=64 time=0.5 ms
64 bytes from 192.168.1.100: icmp_seq=2 ttl=64 time=0.4 ms
```

### Ping Test After UFW
```bash
$ ping 192.168.1.100
PING 192.168.1.100 (192.168.1.100) 56(84) bytes of data.
(no response)
```

### UFW Log Sample (Blocked Traffic)
```bash
$ sudo tail /var/log/ufw.log
Sep 06 18:30:01 ubuntu-vm kernel: [UFW BLOCK] IN=eth0 OUT= MAC=... SRC=192.168.1.101 DST=192.168.1.100 PROTO=ICMP TYPE=8 CODE=0
Sep 06 18:30:02 ubuntu-vm kernel: [UFW BLOCK] IN=eth0 OUT= MAC=... SRC=192.168.1.101 DST=192.168.1.100 PROTO=TCP SPT=12345 DPT=23
```

### Testing Blocked Port
```bash
$ nc -zv 192.168.1.100 23
nc: connect to 192.168.1.100 port 23 (tcp) failed: Connection refused
```

## 6. Conclusion
This project successfully configured a UFW firewall on an Ubuntu VM to allow only SSH and HTTP traffic while blocking all other incoming traffic, including ping and unauthorized port access. The before-and-after nmap scans confirmed that only ports 22 and 80 remained open, and UFW logs verified blocked traffic, such as ICMP and attempts to access port 23. The project reinforced the importance of firewall configuration in securing a system and provided hands-on experience with UFW, nmap, and log analysis. Key learnings include the need to carefully plan firewall rules to avoid locking out essential services and the value of logging for monitoring blocked traffic.

## 7. Challenges Faced
- **Accidental SSH Lockout**: Initially, enabling UFW without first allowing SSH (port 22) caused a loss of remote access to the VM. This was resolved by accessing the VM through the virtual machine console and adding the SSH rule before enabling UFW.
- **Log Visibility**: UFW logging was disabled by default, requiring `sudo ufw logging on` to capture blocked traffic details.
- **Nmap Permissions**: Running nmap on the scanning machine sometimes required `sudo` for full port scans (e.g., `sudo nmap -sS -p- <VM_IP_ADDRESS>`), which was initially overlooked.

## 8. References
- Ubuntu UFW Documentation: https://help.ubuntu.com/community/UFW
- Nmap Official Documentation: https://nmap.org/book/man.html
- Apache2 Installation Guide: https://ubuntu.com/tutorials/install-and-configure-apache
- Linux Firewall Tutorial: https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu

---
