# cyber_homelab
Isolated Kali Linux/Ubuntu VM lab for practicing and documenting attack detection techniques

Overview

This lab consists of two virtual machines running fully isolated from my host network, communicating only with each other:

Role	OS	Purpose
Attacker	Kali Linux (ARM64)	Runs offensive tools (nmap, hydra, etc.)
Victim Ubuntu Server (ARM64)	Target host running SSH and a vulnerable web app (DVWA) for exploitation practice

Hypervisor: UTM (QEMU-based, native Apple Silicon virtualization) Network mode: Isolated/shared virtual network — no exposure to the host's physical LAN

Architecture

Kali Linux (Attacker):
192.168.64.x
Nmap
Hydra
Firefox
         

Ubuntu Server (Victim):
192.168.64.3
OpenSSH
Docker (DVWA)
System logs (auth.log)        
