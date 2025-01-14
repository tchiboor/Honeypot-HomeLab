# Building a Cybersecurity Home Lab: Installing and Exposing a Cowrie Honeypot on the Internet (Part 1)

Have you ever wondered what kind of cyberattacks are lurking out there, just waiting for vulnerable systems? **_Me too. Turns out, it’s not if they find you — it’s how fast. Spoiler alert: faster than pizza delivery._**

Understanding how attackers operate in real-world conditions is critical for improving cybersecurity defenses in today's digital landscape. Setting up a public-facing honeypot provides a unique window into their methods.

When I first installed Cowrie on my home network, I was excited to see it in action. However, confined to a private environment, the only activity I saw came from my own test attacks. While useful for understanding the basics, it lacked the richness and unpredictability of real-world attack data. So, I decided to take things further: set up a Cowrie honeypot, expose it to the internet, and analyze real attack data for meaningful threat intelligence.

---

## What is Cowrie?

Cowrie is a medium-to-high-interaction SSH and Telnet honeypot designed to log brute force attacks and attacker shell interactions.

- **Medium Interaction Mode (Shell):** Emulates a UNIX system in Python.
- **High Interaction Mode (Proxy):** Acts as an SSH/Telnet proxy to observe attacker behavior on another system.

Previously, I used Cowrie in **medium interaction mode**, but this time, I’ll deploy it in **high interaction mode** for a more “realistic” playground.

> **Pro Tip:**  
> Make sure to have coffee on standby. Nothing adds suspense to installation and configuration like wondering if something will break halfway through!  
> Also, **regularly take snapshots of your VM** during key setup stages. Snapshots save time and frustration by letting you roll back to a stable state if something goes wrong.

---

## 1. Set Up the Host Machine

- Install an **Ubuntu server virtual machine** using your preferred virtualization software.
- Ensure the host machine meets the requirements for Cowrie:
  - At least 8 GB RAM and a quad-core processor for running Cowrie and multiple VMs.
  - Ubuntu 20.04 or later is recommended.

For this lab, I’m using **Ubuntu 24.04 deployed on Proxmox**.  
Why Proxmox? It’s flexible, scalable, and enables efficient VM management. Thanks to [Learn Linux TV](https://www.youtube.com/watch?v=5j0Zb6x_hOk&list=PLT98CRl2KxKHnlbYhtABg6cF50bYa8Ulo), I’m learning Proxmox’s ins and outs while building my skills.

![Virtual machine setup on Proxmox to deploy and host Cowrie honeypot services](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*d2UDby69OxpTGczs69MJLA.png)

---

## 2. SSH into the VM and Proceed with Cowrie Installation

![Screenshot of the user “cowrie” connected to the VM through an SSH session. The shell is ready for commands](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*8py7aT20YVrVwq6YVWV8Ag.png)

---

## 3. Install Cowrie and All Dependencies

1. Update the system:

   ```bash
   sudo apt update && sudo apt upgrade -y
