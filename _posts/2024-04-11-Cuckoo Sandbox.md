---
title: "Cuckoo Sandbox Installation"
date: 2024-04-11 10:00:00 +0530
categories: [DFIR]
tags: [Sandbox, Malware Analysis]
image: /assets/img/cuckoo.jpeg
by: roota2
---

# Cuckoo Sandbox Installation

---

## Sandbox:

- Malware sandboxing is a practical application of the dynamical analysis approach instead of statically analyzing the binary file, it gets executed and monitored in real-time.

## Cuckoo sandbox:

- Cuckoo is an open source automated malware analysis system.
- It’s used to automatically run and analyze files and collect comprehensive analysis results that outline what the malware does while running inside an isolated operating system.

---

**Host-OS: Ubuntu 18.04.6 (6 gb ram, 4 cpu)**

**Guest-OS: Windows 7 (2 gb ram, 2 cpu)**

---

# Requirements:

- Python 2.7
- MongoDB
- PostgreSQL
- pip
- python-virtualenv
- python-setuptools
- python-dev
- swig
- Virtualbox-5.2
- tcpdump
- Volatility
- m2crypto 0.37.1

---

# Setting up Host-OS:

1. Download available updates

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

1. Install necessary python modules and libraries 

```bash
sudo apt-get install python python-pip python-dev libffi-dev libssl-dev
```

```bash
sudo apt-get install python-virtualenv python-setuptools
```

```bash
sudo apt-get install libjpeg-dev zlib1g-dev swig
```

1. Install MongoDB for Web interface

```bash
sudo apt-get install mongodb
```

1. Install PostgreSQL for database support

```bash
sudo apt-get install postgresql libpq-dev
```

1. Install virtualbox-5.2 
- In order to install older version of the virtualbox we need to modify the package list in ubunut

```bash
echo deb http://download.virtualbox.org/virtualbox/debian xenial contrib | sudo tee -a /etc/apt/sources.list.d/virtualbox.list
```

```bash
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
```

```bash
sudo apt-get update
```

```bash
sudo apt-get install virtualbox-5.2
```

1. Creating users and groups

```bash
sudo adduser cuckoo
```

- Adding user cuckoo to vboxusers

```bash
sudo usermod -a -G vboxusers cuckoo
```

- Creating a group named pcap and adding cuckoo to the group

```bash
sudo groupadd pcap
```

```bash
sudo usermod -a -G pcap cuckoo
```

1. Install tcpdump for dumping the network activity

```bash
sudo apt-get install tcpdump
```

- Adding the privileges to pcap group

```bash
sudo chgr pcap /usr/sbin/tcpdump
```

- Making changes to binary so that it will properly run even without the root privileges

```bash
sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
```

- Verify the result

```bash
getcap /usr/sbin/tcpdump

Result: /usr/sbin/tcpdump = cap_net_admin,cap_net_raw+eip
```

1. Install M2Crypto and swig 

```bash
sudo apt-get install swig
```

```bash
sudo pip install m2crypto==0.37.1
```

1. Install cuckoo
- Create virtual environment

```bash
virtualenv venv
```

- Enter into it

```bash
 . venv/bin/activate
```

- Install setuptools

```bash
pip install -U pip setuptools
```

- Now, install cuckoo

```bash
pip install -U cuckoo
```

---

# Setting up the Guest-OS

1. Open virtualbox and add install win7 or winxp with the name of cuckoo1
- Give at least 2GB ram, 2 CPU, and 32 GB storage
1. Create a host-only adapter in virtualbox and name it **vboxnet0**
2. Install python 2.7.17
3. Install browsers and necessary software in guest-os
4. Copy the [agent.py](http://agent.py) from (Host-OS: ~/.cuckoo/agent/)

7. In guest-os; if it is windows-7 add the [agent.py](http://agent.py) file to startup folder

1. Off the network firewall and windows auto update
2. Take a snapshot and name it Snapshot 1

---

# Cuckoo configuration

- The configuration files are available in ~/.cuckoo/conf/
- Make sure these configuration below are enabled

## Virtualbox.conf

```bash
[virtualbox]
mode = gui
machines = cuckoo1

lable = cuckoo1

platform = windows
ip = 192.168.56.101
snapshot = Snapshot 1
```

## Cuckoo.conf

```bash
[cuckoo]

memory_dump = yes
mchinery = virtualbox

[resultserver]
ip = 192.168.56.1
```

## Auxiliary.conf

```bash
[sniffer]
enabled = yes
```

## Processing.conf

```bash
[memory]
enables = yes
```

## Reporting.conf

```bash
[singlefile]
enabled = yes

[mongodb]
enabled = yes
```

---

# Routing configuration in Host-os

- Configure IP forwarding so an internet connection gets routed from the host machine to the guest VM.

```bash
sudo iptables -t nat -A POSTROUTING -o <interface-name> -s 192.168.56.0/24 -j MASQUERADE
```

```bash
sudo iptables -P FORWARD DROP
```

```bash
sudo iptables -A FORWARD -m state — state RELATED,ESTABLISHED -j ACCEPT
```

```bash
sudo iptables -A FORWARD -s 192.168.56.0/24 -j ACCEPT
```

```bash
sudo iptables -A FORWARD -s 192.168.56.0/24 -d 192.168.56.0/24 -j ACCEPT
```

```bash
sudo iptables -A FORWARD -j LOG
```

- Enabling IP forwarding in the kernel

```bash
echo 1 | sudo tee -a /proc/sys/net/ipv4/ip_forward
```

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

---

# Analyzing using cuckoo

Run following commands to start Cuckoo and the Cuckoo web interface. Run those in two separate terminal windows.

Terminal #1: **cuckoo**

Terminal #2: **cuckoo web runserver**

---
