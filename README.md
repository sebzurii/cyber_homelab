# cyber_homelab
Isolated Kali Linux/Ubuntu VM lab for practicing and documenting attack detection techniques

Overview

This lab consists of two virtual machines running fully isolated from my host network, communicating only with each other:

Role	OS	Purpose
Attacker	Kali Linux (ARM64)	Runs offensive tools (nmap, hydra, etc.)
Victim	Ubuntu Server (ARM64)	Target host running SSH and a vulnerable web app (DVWA) for exploitation practice

Hypervisor: UTM (QEMU-based, native Apple Silicon virtualization) Network mode: Isolated/shared virtual network — no exposure to the host's physical LAN

Architecture
┌─────────────────────┐         ┌──────────────────────┐
│   Kali Linux         │         │   Ubuntu Server        │
│   (Attacker)          │ ------> │   (Victim)              │
│   192.168.64.x         │         │   192.168.64.3           │
│                        │         │                          │
│   - nmap               │         │   - OpenSSH              │
│   - hydra               │         │   - Docker → DVWA         │
│   - Firefox              │         │   - system logs (auth.log) │
└─────────────────────┘         └──────────────────────┘
              Isolated virtual network (UTM)
Exercises

Each folder below is a self-contained write-up: objective, steps taken, evidence collected, detection logic, and remediation.

#	Exercise	Skills demonstrated
01	SSH Brute-Force Detection	Log analysis, attack recognition, detection rule design
02	(coming soon)	
Why I built this

I'm working toward my first cybersecurity role (SOC analyst / security analyst track) and wanted hands-on evidence of practical skills beyond certifications — specifically the ability to set up infrastructure, execute realistic attacks safely, read and interpret logs, and think through detection and remediation like an analyst would.
