---
layout: post
title: "Cyber Kill Chain"
slug: "cbc"
date: "2023-09-08"
comments: false
categories:
  - cybersecurity
tags:
  - Lockheed Martin
  - Cyber Kill Chain
  - Attack phase
---
The Cyber Kill Chain is a framework created by Lockheed Martin to define and describe the steps use by an adversary to compromise and exfiltrate data in the cyberspace. The different phases involve in the kill chain can help us understand the adversary better and mitigate their various attacks successfully by breaking the chain.

![Cyber Kill Chain](/images/cyberkillchain/cyber-kill-chain.png)
img: [Lockheed Martin](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)

The attack phases involve in the kill chain are:
- Reconnaissance
- Weaponization
- Delivery
- Exploitation
- Installation
- Command and Control
- Actions on Objectives.

#### More on the Cyber Kill Chain

**Reconnaissance**

The task of discovering and  collecting information about the system and victims from various sources either passively or actively falls under reconnaissance. 

OSINT (Open-Source Intelligence) is integral to this phase of the kill chain and the amount of information collected and gathered in this phase can drastically influence the success of the next phases. 

Harvesting of emails, credentials and other PII can expose a company to social engineering attacks and phishing, which can easily set them up to the next phases of the kill chain. OSINT tools like the [OSINT Framework](https://osintframework.com/) and [theHarvester](https://github.com/laramies/theHarvester) are popular tools used for gathering information passively. A treasure of information can also be gathered form social media sites as well. 

Knowing information of the services and applications that are running in the victim's machine along with their versions and other information can offer a lot of insight into the infrastructure and can greatly speed up the movement through the next phases of the attacks.

**Weaponization**

In the next phase of the kill chain, according to information obtained from the previous phase an adversary can create their own tool to exploit the infrastructure of the victim, either through malwares or software to exploit the vulnerabilities that exist in the system.  Malwares, exploits like shell script and macros all fall under the weaponization phase.

**Delivery**

The act of getting the weapon into the victim hands/infrastructure for further exploit is included in the delivery phase. Phishing, distributing of malware as downloadable file, using a USB to distribute the malware are all ways of delivering our weapon.

**Exploitation**

After the payload/weapon has been delivered to the victim.  Exploitation of the system occurs when the user interacts with the payload/weapon. 

**Installation**

After the system has been exploited and access has been granted to the adversary, the adversary tries to implement persistent access to the system that can exist even after reboot. To achieve persistence the adversary installs a backdoor in the system through various means like creating a web shell, installing of programs in the startup folder. Window services can also be modified and cron jobs can also be created to create persistence.

**Command and Control**

After exploiting and getting persistence into the system of the victim, a communication channel with the adversary is establish with a server controlled by the adversary also known as the command and control server. This C2 is used to issue commands to the victims system and also to exfiltrate data from the victim. Nowadays communication with the victim and C2 is usually done using HTTP request and DNS request so as to hide the itself with legitimate traffic and avoid detection.

**Actions on Objectives**

In the last phase of the kill chain, the adversary performs actions on the objectives which usually is exfiltration of data such as collecting of user credentials, collecting PII, financial data and other sensitive information and files.




