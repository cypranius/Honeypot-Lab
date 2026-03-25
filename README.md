# Honeypot
Hands-on honeypot lab: Ubuntu + Cowrie + Kali on an internal network. Captures SSH attacks, logs in JSON, and documents real troubleshooting.

# 🍯 Honeypot Security Lab

## 📃 Purpose
I understood the concept of a honeypot in theory, but I wanted to see what it actually looked like in practice, how it behaves, how attackers interact with it, and how they get caught. This project was my way of bridging that gap between theory and hands-on experience. What started as curiosity turned into an ambitious deep dive into virtualization, networking, and security.

## 📋 Overview
A fully isolated cybersecurity lab built in VirtualBox, featuring a Cowrie SSH honeypot and a Kali Linux attacker machine. The environment is completely air-gapped using VirtualBox's internal network mode, allowing for safe attack simulation and log analysis without any risk to the host system or external networks.

## 🚀 Skills Demonstrated
- **Virtualization** – Created and managed VMs in VirtualBox, configured network modes (NAT, Internal Network), diagnosed and resolved VM stability issues
- **Linux System Administration** – User management, file permissions, network configuration, recovery mode troubleshooting, password recovery
- **Network Configuration** – Set up isolated internal networks, assigned static IPs, configured VM network adapters for lab isolation
- **Honeypot Deployment** – Installed and configured Cowrie to capture SSH attacks, managed authbind for privileged port access
- **Troubleshooting** – Diagnosed and resolved port conflicts (systemd socket activation), filesystem corruption, permission errors, password hash corruption, and VM instability
- **Documentation** – Created comprehensive technical documentation for reproducibility, including challenges and solutions

## 🛠️ Tools & Technologies
- **VirtualBox 7.x** – Virtualization platform
- **Ubuntu Server 24.04** – Honeypot host (192.168.1.10)
- **Kali Linux 2025.4** – Attacker machine (192.168.1.20)
- **Cowrie 2.9.0** – Medium-interaction SSH honeypot
- **Authbind** – Allows non-root process to bind to privileged port 22
- **JSON** – Structured logging format for attack data

## 🔧 Key Features

- **Isolated Internal Network** – Created a dedicated internal network in VirtualBox (`honeypot-net`) allowing only the Kali attacker and Ubuntu honeypot to communicate, completely air-gapped from the internet
- **Cowrie SSH Honeypot** – Deployed Cowrie, an open-source medium-interaction honeypot that simulates authentic SSH services and captures all attacker interactions in a safe, controlled environment
- **JSON Structured Logging** – All attack data including login attempts, commands, and connection details, is logged in JSON format for easy parsing, analysis, and integration with other tools
- **Static IP Configuration** – Assigned fixed IP addresses (`192.168.1.10` for Ubuntu, `192.168.1.20` for Kali) ensuring reliable, predictable communication between the honeypot and attacker machine
- **Authbind for Privileged Port Access** – Configured authbind to allow the non-root `cowrie` user to bind to port 22, enabling the honeypot to receive SSH traffic directly without running as root
**Manual SSH Service Disable**, Disabled systemd's default SSH socket to free port 22 for the honeypot, showing understanding of Linux service management
- **Dual‑VM Attack Simulation** - Dedicated Kali machine enables safe, repeatable attack simulations without contaminating the honeypot host

## 🗺️ Network Architecture
| Component | IP Address | Role |
|:-----------|:------------|:------|
| **Kali Linux** | 192.168.1.20 | Attacker machine |
| **Ubuntu Server** | 192.168.1.10 | Honeypot host |
| **Cowrie** | Port 22 | SSH honeypot |
| **Virtual Network** | `honeypot-net` | Isolated internal network |

## 🧠 How The Honeypot Works
A honeypot is a decoy system designed to look like a legitimate server, tricking attackers into connecting and interacting with it. While they explore the fake environment, every action they take, login attempts, commands, file access, is logged for analysis. In this lab, Cowrie acts as the honeypot by simulating a real SSH server on port 22. When the Kali attacker connects, it is presented with a realistic (but fake) Linux shell. Behind the scenes, Cowrie records all activity in structured JSON logs, providing visibility into attacker behavior without any risk to the real system.

## 📝 Step-by-Step Process
1. Downloaded and set up the [Kali Linux VM](https://github.com/cypranius/KaliVM) as shown in the guide.
2. Updated and upgraded the Kali VM.
3. Flushed the existing IP address to prepare for a static assignment:
   ```bash
   sudo ip addr flush dev eth0
   ```
4. Assigned a static IP address:
   ```bash
   sudo ip addr add 192.168.1.20/24 dev eth0
   ```
5. Activated the new IP address:
   ```bash
   sudo ip link set eth0 up
   ```
6. Verified the new IP address was recognized by the VM:
   ```bash
   sudo ip addr show
   ```
7. Powered off the VM to change the network adapter from NAT to Internal Network.
8. Downloaded the **Server install image** from the [official releases page](https://releases.ubuntu.com/24.04.3/) (file: `ubuntu-24.04.3-live-server-amd64.iso`)
9. Created Ubuntu VM with 2GB RAM, 2 cores, 25GB storage, installed Ubuntu Server with OpenSSH.
10. Assigned static IP to Ubuntu:
    ```bash
   sudo ip addr flush dev enp0s3
   sudo ip addr add 192.168.1.10/24 dev enp0s3
   sudo ip link set enp0s3 up
    ```
11. Created cowrie user
    ```bash
    sudo adduser --disabled-password --gecos "" cowrie
    ```
12. moved to cowrie user via:
    ```bash
    sudo su - cowrie
    ```
13. Installed the open source honeypot (cowrie)
    ```bash
    git clone https://github.com/cowrie/cowrie.git
    ```
14. cd to cowrie and creates the virtual environment
    ```bash
    python3 -m venv cowrie-env
    ```
15. activate cowrie
    ```bash
    source cowrie-env/bin/activate
    ```
16. Updated pip, installed the required Python packages for Cowrie, and installed Cowrie as an editable package:
    ```bash
    pip install --upgrade pip
    pip install -r requirements.txt
    pip install -e .
    ```
17. Copied the default config and modified it to listen on port 22 (changed from 2222 to 22):
    ```bash
    cp etc/cowrie.cfg.dist etc/cowrie.cfg
    nano etc/cowrie.cfg 
    ```
18. Configured authbind to allow Cowrie to bind to privileged port 22:
    ```bash
    sudo apt install authbind -y
    sudo touch /etc/authbind/byport/22
    sudo chown cowrie:cowrie /etc/authbind/byport/22
    sudo chmod 500 /etc/authbind/byport/22
    ```
19. Disabled the systemd SSH service to free up port 22:
    ```bash
    sudo systemctl stop ssh.socket
    sudo systemctl disable ssh.socket
    sudo systemctl stop ssh.service
    sudo systemctl disable ssh.service
    ```
20. Started Cowrie with authbind:
    ```bash
    authbind --deep cowrie start
    ```
21. Verified Cowrie was running and listening on port 22:
    ```bash
    cowrie status
    sudo ss -tlnp | grep :22
    ```
22. From Kali, tested the honeypot:
    ```bash
    ssh root@192.168.1.10
    ```
23. Located the Cowrie log file and viewed live attacks:
    ```bash
    find /home -name "cowrie.json" 2>/dev/null
    cd /home/cowrie/cowrie/var/log/cowrie/
    sudo tail -f cowrie.json
    ``` 
## 🐛 Challenges & Lessons Learned
### Virtual Machine Configuration
- **Snapshots are essential** – Before making significant changes (network config, Cowrie installation), I took snapshots. This made it easy to revert when something broke, saving hours of rework.
### Network & Isolation
- **Plan ahead before isolating** – Before switching the VM to Internal Network (no internet), I made sure all necessary packages, updates, and dependencies were installed. This avoided the time‑wasting cycle of constantly flipping network modes back and forth to grab missing components.
- **Static IPs are essential for lab consistency** – Assigning static IP addresses (`192.168.1.10` for Ubuntu, `192.168.1.20` for Kali) ensured reliable communication between the honeypot and attacker machine. Without them, I'd have to track changing DHCP addresses, which would break the lab every time the VMs rebooted.
- **Verify with multiple methods, don't trust a single status** – After starting Cowrie, `cowrie status` reported it was running, but connections were still failing. I learned to verify with concrete checks: `sudo ss -tlnp | grep :22` to see if the port was actually listening, and testing with an actual SSH connection. A service can report "running" without being functional.
- **Understand how systemd manages ports** – I discovered that systemd's socket activation (`ssh.socket`) was still holding port 22 even after disabling the SSH service. This prevented Cowrie from binding to the port. Disabling both `ssh.service` and `ssh.socket` finally freed it. Lesson: stopping a service doesn't always free its port.
### Troubleshooting
- **Password reset didn't stick** – Following [this guide](https://linuxconfig.org/ubuntu-14-04-lost-password-recovery), I reset the password via recovery mode, but it didn't save. AI suggested checking the filesystem mount — recovery mode mounts read‑only. Running `mount -o remount,rw /` before `passwd` fixed it.
- **Privileged ports need special handling** – When Cowrie failed to bind to port 22 with "Permission denied," I learned that non‑root users can't use ports below 1024. Authbind solved this by granting the `cowrie` user specific permission to use port 22 without running as root.
### Planning & Process
- **Start with a roadmap** – I dove straight into the honeypot setup, only to realize later I needed to rebuild my Kali VM. Juggling both tasks led to more troubleshooting than building. A 20‑minute plan upfront would have saved hours.
- **Simple passwords are fine for isolated labs** – I thought I'd forgotten my Ubuntu password, but after verifying the account existed and even resetting it once, login still failed. AI suggested the password hash might be corrupted. For an isolated lab, a simple password is sufficient. If you use complex passwords, store them in a password manager to avoid recovery headaches.

## 📸 Screenshots
| Screenshot | Description |
|:-----------|:------------|
| ![Kali IP](https://github.com/user-attachments/assets/0e565e33-4e1d-4ce9-817b-22aa1d6c818b) | Kali VM assigned static IP `192.168.1.20` |
| ![Ubuntu IP](https://github.com/user-attachments/assets/5492879f-22c0-47d9-bcd4-791562a3ba31) | Ubuntu VM assigned static IP `192.168.1.10` |
| ![Ping Test](https://github.com/user-attachments/assets/73319ebb-9ae0-410f-a060-92208b432166) | Kali successfully pings Ubuntu network connectivity |
| ![Cowrie Status](https://github.com/user-attachments/assets/62abd88f-2700-415a-9fdd-02739964ffea) | `cowrie status` shows the honeypot is running |
| ![Cowrie on Port 22](https://github.com/user-attachments/assets/30d042b3-0796-4907-8e16-e997eeea2af2) | `ss -tlnp` confirms Cowrie is listening on port 22 |
| ![SSH to Honeypot](https://github.com/user-attachments/assets/d3c91c9b-a1af-4618-be91-ff74c21f4878) | SSH connection from Kali lands in Cowrie's fake shell (`root@svr04`) |
| ![Locating Log File](https://github.com/user-attachments/assets/2c4f35c0-343e-4ca0-aed3-2bd922f33e94) | Using `find` to locate `cowrie.json` |
| ![Attack Logs](https://github.com/user-attachments/assets/179539cc-287b-45a4-8977-58bf9eb77acf) | Live attack logs showing captured SSH attempts |

## 📚 References
- [Cowrie Official Documentation](https://docs.cowrie.org/) - Complete guide for installation, configuration, and features 
- [VirtualBox Networking Modes](https://www.virtualbox.org/manual/ch06.html) - Detailed explanation of NAT, Internal Network, Host-Only, and Bridged modes
- [authbind Manual](https://manpages.debian.org/authbind) - How to allow non-root processes to bind to privileged ports (<1024)
- [Ubuntu Password Recovery Guide](https://linuxconfig.org/ubuntu-14-04-lost-password-recovery) – Step‑by‑step instructions for resetting a lost Ubuntu password via recovery mode. This guide got me into the system, but I needed additional steps (remounting the filesystem as read‑write) to make the change persist, which I solved with AI assistance.
