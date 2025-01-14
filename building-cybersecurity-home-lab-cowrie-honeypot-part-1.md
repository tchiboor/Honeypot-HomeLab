Building a Cybersecurity Home Lab: Installing and Exposing a Cowrie Honeypot on the Internet Part 1
===================================================================================================

[![Trevor Henry Chiboora](https://miro.medium.com/v2/da:true/resize:fill:88:88/0*9R-fyHsdeMGAbwPD)](https://medium.com/?source=post_page---byline--8391f82c4312--------------------------------)

[Trevor Henry Chiboora](https://medium.com/?source=post_page---byline--8391f82c4312--------------------------------)

·

[Follow](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fuser%2F722e97a489e5&operation=register&redirect=https%3A%2F%2Fchiboorat.medium.com%2Fbuilding-a-cybersecurity-home-lab-installing-and-exposing-a-cowrie-honeypot-on-the-internet-part-1-8391f82c4312&user=Trevor+Henry+Chiboora&userId=722e97a489e5&source=post_page-722e97a489e5--byline--8391f82c4312---------------------post_header-----------)

7 min read

·

1 day ago

[nameless link](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fp%2F8391f82c4312&operation=register&redirect=https%3A%2F%2Fchiboorat.medium.com%2Fbuilding-a-cybersecurity-home-lab-installing-and-exposing-a-cowrie-honeypot-on-the-internet-part-1-8391f82c4312&user=Trevor+Henry+Chiboora&userId=722e97a489e5&source=---header_actions--8391f82c4312---------------------clap_footer-----------)

--

[nameless link](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2F8391f82c4312&operation=register&redirect=https%3A%2F%2Fchiboorat.medium.com%2Fbuilding-a-cybersecurity-home-lab-installing-and-exposing-a-cowrie-honeypot-on-the-internet-part-1-8391f82c4312&source=---header_actions--8391f82c4312---------------------bookmark_footer-----------)

Listen

Share

Have you ever wondered what kind of cyberattacks are lurking out there, just waiting for vulnerable systems? **_Me too. Turns out, it’s not if they find you — it’s how fast. Spoiler alert: faster than pizza delivery._**

Understanding how attackers operate in real-world conditions is critical for improving cybersecurity defenses in today's digital landscape. Setting up a public-facing honeypot provides a unique window into their methods.

When I first installed Cowrie on my home network, I was excited to see it in action. But because it was confined to a private environment, the only activity I saw came from the test attacks I ran myself. While this was useful for understanding the basics, it lacked the richness and unpredictability of real-world attack data. That’s when I decided to take things further: setting up a Cowrie honeypot, exposing it to the internet, and diving into the real attack data to gain insights and gather meaningful threat intelligence.

What is Cowrie?
---------------

Cowrie is a medium-to-high-interaction SSH and Telnet honeypot designed to log brute force attacks and attacker shell interactions. In medium interaction mode (shell), it emulates a UNIX system in Python. In contrast, in high interaction mode (proxy), it acts as an SSH and Telnet proxy to observe attacker behavior on another system.

Previously, I used Cowrie in medium interaction mode, but this time, I will deploy it in high interaction mode to see what happens when I give attackers a more “realistic” playground.

> **Pro Tip:**
> Make sure to have coffee on standby — nothing adds suspense to installation and configurations like wondering if something will break halfway through!
> 
> While you’re at it, **regularly take snapshots of your VM** during key stages of setup. Snapshots are lifesavers when something goes wrong (and trust me, it happens). They allow you to roll back to a stable state without starting from scratch, saving time and a lot of frustration.

1. Set Up the Host Machine
---------------------------

*   Install an Ubuntu server virtual machine using your preferred virtualization software
*   Ensure the host machine meets the requirements for Cowrie: At least 8 GB RAM and a quad-core processor for running Cowrie and multiple VMs.
*   Ubuntu 20.04 or later is recommended as the OS.

For this lab, I am using Ubuntu 24.04 deployed on ProxMox.

Why ProxMox? Aside from its fancy name that makes me feel like a tech wizard, ProxMox is an excellent choice for labs like this. It’s flexible, and scalable, and enables efficient deployment and management of virtual machines.

This is my first time using Proxmox, which makes it even more exciting. It’s not just a tool for setting up my lab; it’s also a chance for me to explore and learn how to leverage its capabilities. Thanks to [Learn Linux TV](https://www.youtube.com/watch?v=5j0Zb6x_hOk&list=PLT98CRl2KxKHnlbYhtABg6cF50bYa8Ulo), I’m learning the ins and outs of Proxmox, which is helping me build my skills. As I grow more familiar with it, I plan to expand my lab by adding more VMs, configurations, and security services — such as firewalls and SIEM tools — to build a comprehensive and robust environment.

![Virtual machine setup on Proxmox to deploy and host Cowrie honeypot services](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*d2UDby69OxpTGczs69MJLA.png)

2. SSH into the VM and Proceed with Cowrie Installation
--------------------------------------------------------

![Screenshot of the user “cowrie” connected to the VM through an SSH session. The shell is ready for commands](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*8py7aT20YVrVwq6YVWV8Ag.png)

3. Install Cowrie and all Dependences
--------------------------------------

First, we need to update the system :

```
sudo apt update && sudo apt upgrade -y
```

Then we install all the dependencies of Cowrie :

```
sudo apt install git python-virtualenv libssl-dev build-essential libpython-dev python2.7-minimal authbind -y
```![Installation of required dependencies for setting up Cowrie](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*mLcTRUSY1M5Lwa8WS0UzPg.png)

Next, we need to create a new user, cowrieuser. It’s strongly recommended to run Cowrie with a dedicated, non-root user account.

Why? Because running a honeypot as root is like leaving your front door wide open and then wondering why raccoons are having a feast in your kitchen. A dedicated user minimizes potential damage if the honeypot gets compromised (very unlikely), adding an extra layer of security to your setup.

```
sudo adduser --disabled-password cowrieuser
```![Terminal displaying the creation of a new user account cowrieuser](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*UCwrZi8ZIjHlYI74MDb2gw.png)

Switch to the newly created user and download the Cowrie code

```
sudo su - cowrieuser
git clone http://github.com/cowrie/cowrie
```![Terminal displaying switching to the new user and downloading the Cowrie code from GitHub](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*LCVd5GQabyjCoOGdCSa5Pw.png)

Now we need to create a virtual environment for Python and Cowrie to run from and activate it:

```
cd cowrie/
python3 -m venv cowrie-env
source cowrie-env/bin/activate
```![Creating python virtual environment and activating it.](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*1XZwSuVlWsluKmbYx0brhA.png)

Then we need to install the packages of Python that Cowrie needs to run :

```
pip install — upgrade pip
pip install — upgrade -r requirements.txt
```![Installing the packages of Python that Cowrie needs to run.](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*EIBJOCHBlpdBnzBA1O3Erg.png)

4. Install the configuration file
----------------------------------

The configuration for Cowrie is stored in cowrie.cfg.dist and cowrie.cfg (located in cowrie/etc). The .dist file contains default settings and should not be edited. Instead, copy it. Both files are read on startup, where entries from cowrie.cfg takes precedence. Upgrades can overwrite the .dist file, cowrie.cfg will not be touched.

> **Pro Tip**:
> 
> Do not edit the cowrie.cfg.dist file unless you enjoy redoing work after an update. Think of it like writing on a whiteboard that gets erased every week.

**# This command copies the** _cowrie.cfg.dist_ **to** _cowrie.cfg_ **which we can edit:**

```
cp cowrie.cfg.dist cowrie.cfg
```

Telnet is disabled by default. To enable telnet, for example, edit the cowrie.cfg and change the value false to true:

```
nano etc/cowrie.cfg
```![Enabling telnet service](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*iiu23Q1oeSKrlnczpQlWMQ.png)

5. Enable Cowrforie to start automatically after reboot
--------------------------------------------------------

Return to privileged user and open a new service file for Cowrie:

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Uj6QjbWTukhxZFfT_OCbyg.png)

Add the following content to the file:

```
[Unit]
Description=Cowrie SSH/Telnet Honeypot
After=network.target
[Service]
User=cowrieuser
Group=cowrieuser
WorkingDirectory=/home/cowrieuser/cowrie
ExecStart=/bin/bash -c 'source /home/cowrieuser/cowrie/cowrie-env/bin/activate && /home/cowrieuser/cowrie/bin/cowrie start --nodaemon'
Restart=always
Environment="PATH=/home/cowrieuser/cowrie/cowrie-env/bin:/usr/bin:/bin"
[Install]
WantedBy=multi-user.target
```

Replace `/home/cowrieuser/cowrie` with the path to your Cowrie installation if it's different

Reload Systemd, Enable and start the service

```
sudo systemctl daemon-reload
sudo systemctl enable cowrie
sudo systemctl start cowrie
```![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*KeSvhSWNvFyp8FTSMB2v9g.png)

Check and verify if the service is running

```
sudo systemctl status cowrie
```![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ltMznA4wgu_PaGAnHZsXkQ.png)

6. Listening on ports 22 and 23 (OPTIONAL)
-------------------------------------------

By default, Cowrie runs its SSH and Telnet services on port 2222 and port 2223, respectively. To make Cowrie accessible on the default SSH port (22), you can use iptables to forward traffic from port 22 to Cowrie’s port 2222.

However, to avoid conflicts, you’ll need to move your system’s standard SSH service to a different port (e.g., port 3333). The same process applies to forwarding Telnet traffic from port 23 to Cowrie’s port 2223.

> Why forward traffic to Cowrie’s ports? Because hackers love default ports like a moth loves a flame.

Change your host`s SSH standard port in sshd_config. Edit the SSH config file, uncomment the line, and change port number 22 to the port number of your choice. Then save the file.

```
sudo nano /etc/ssh/sshd_config
```![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*Cdmx6gyV-aGpPkdP)

Redirect traffic:

```
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
sudo iptables -t nat -A PREROUTING -p tcp --dport 23 -j REDIRECT --to-port 2223
```

Restart SSH

```
sudo systemctl restart sshd
OR 
sudo systemctl restart ssh
```

Reconnect using the new port

```
ssh -p 2222 username@your_server_ip
```

To watch the logs of your honeypot

```
cat var/log/cowrie/cowrie.json 
cat var/log/cowrie/cowrie.log 
```![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*cqtgJ46f93NyMri3AAbbTA.png)

Congratulations! Your Cowrie honeypot is now live. Think of it as a welcome mat for the internet’s mischief-makers — with the caveat that their every move is being watched and logged

Taking Cowrie to the Next Level: Unlocking High-Interaction Honeypot Capabilities
=================================================================================

At this stage, Cowrie is running as a medium-interaction honeypot. The next step is to configure Cowrie as a proxy and set up a backend pool to make it high-interaction.

In the next part of this series, we will dive into setting up the backend pool for Cowrie. This involves exploring how to configure and manage a pool of QEMU-emulated virtual machines to enhance the realism and effectiveness of your honeypot. Stay tuned as we build upon this setup to create a more dynamic and scalable environment for capturing and analyzing malicious activities.