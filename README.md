# Splunk Lab

## Objective

This lab is all about getting hands-on with Active Directory and security monitoring. I've set up Windows Server 2022 AD domain controller and a Ubuntu Splunk server, then used Kali Linux to play the role of an attacker in the network. The goal is to see what kind of events get generated from these attacks and learn how to configure and use Splunk to catch them. I aim to dive into domain environments and sharpen threat detection skills.

This is by no means a comprehensive step-by-step of my process, but rather a few key processes that are interesting or will be useful for my future reference.
### Skills Learned

- Configuring and managing Active Directory domain environments
- Setting up and utilizing Splunk for security event monitoring and analysis
- Simulating bruteforce attack using Kali Linux
- Performing cybersecurity tests using Atomic Red Team.
- Identifying and interpreting security events generated from attacks via Splunk
- Analyzing and correlating security events in Splunk with the MITRE ATT&CK framework.

### Tools Used

- Windows Server 2022 (for Active Directory domain controller)
- Ubuntu Server (for Splunk server)
- Splunk (for security event monitoring and analysis)
- Kali Linux (for simulating bruteforce attack)
- Windows 10 Client (for interacting with the domain environment)
- Atomic Red Team (for simulating various attacks mapped to the MITRE ATT&CK framework)
- VirtualBox (for virtualising the lab environment)

## Network Diagram

![image](https://github.com/user-attachments/assets/b6374919-90d9-4131-bda0-b14602db5c8c)

## Table of Contents

  1. [Objective](#objective)
  2. [Skills Learned](#skills-learned)
  3. [Tools Used](#tools-used)
  4. [Network Diagram](#network-diagram)
  5. [Setting up Splunk Server and Forwarders](#setting-up-splunk-server-and-forwarders)
      - [Setting Static IP Address and Default Route](#setting-static-ip-address-and-default-route)
      - [Install Splunk Enterprise](#install-splunk-enterprise)
      - [Setting Up Splunk Forwarder](#setting-up-splunk-forwarder)
      - [Installing Sysmon](#installing-sysmon)
      - [Configuring Inputs for Splunk Forwarder](#configuring-inputs-for-splunk-forwarder)
      - [Restarting Splunk Forwarder Service](#restarting-splunk-forwarder-service)
      - [Connecting to Splunk Web Interface](#connecting-to-splunk-web-interface)
  6. [Setting up Active Directory and Provisioning Users](#setting-up-active-directory-and-provisioning-users)
  7. [Performing a Brute Force Attack on Target_PC and Reviewing Events via Splunk](#performing-a-brute-force-attack-on-target_pc-and-reviewing-events-via-splunk)
  8. [Installing Atomic Red Team, Performing a Test, and Reviewing Events in Splunk](#installing-atomic-red-team-performing-a-test-and-reviewing-events-in-splunk)
  9. [Conclusion](#conclusion)

---

## Setting up Splunk server and Forwarders

### Setting Static IP Address and Default Route:

- Configured a static IP address for the Splunk server and defined a default route with the gateway 192.168.10.10.
```sudo nano /etc/netplan/00-installer-config.yaml```

![image](https://github.com/user-attachments/assets/116866a8-7450-41d1-a864-7b63b19eac05)

Apply the changes:
```sudo netplan apply```

### Install Splunk Enterprise:

- Installed Splunk Enterprise on the Splunk server and configure it to start at boot.

```tsoc@splunk:/opt/splunk/bin$ sudo ./splunk enable boot-start -user splunk```

### Setting Up Splunk Forwarder:
Installed and configured Splunk Forwarder on ADDC01 and target-PC (Windows 10) to send data to the Splunk server as a receiving indexer.

![image](https://github.com/user-attachments/assets/82174875-8894-4dfa-a315-da7de8180325)

### Installing Sysmon:

Installed Sysmon to enhance event logging capabilities.

![image](https://github.com/user-attachments/assets/a6ebc509-6c1e-42cc-8dd9-71de538b00cc)

### Configuring Inputs for Splunk Forwarder:

Created an inputs.conf file in C:\Program Files\SplunkUniversalForwarder\etc\system\local on ADDC01 and target-PC, configuring settings as per.

![image](https://github.com/user-attachments/assets/02f7be87-89b9-480d-bf1c-7d2ec3f41725)
![image](https://github.com/user-attachments/assets/268c0406-55a8-4895-a701-7514c03345da)

### Restarting Splunk Forwarder Service:

Restarted the Splunk Forwarder service on ADDC01 and set to log on as local system account.

![image](https://github.com/user-attachments/assets/c20212e7-288e-4a9b-ba93-480a20d3a1da)

### Connecting to Splunk Web Interface:

Accessed the Splunk server's web interface at port 8000, then created an index named endpoint as specified in the inputs.conf file. I repeated this process for both ADDC01 and target-PC to ensure the Splunk server receives events from both sources.

![image](https://github.com/user-attachments/assets/eaf41b63-d39f-4c7b-99e7-48f63d10387f)

---

## Setting up Active Directory and provisioning users

Install Active Directory Domain Services on ADDC01

![image](https://github.com/user-attachments/assets/4ae4f280-96f6-4be5-9f32-ff051dfed381)

Promote ADDC01 to Domain Controller

![image](https://github.com/user-attachments/assets/ad505284-2217-4875-9d85-70dddc9012f9)

I joined target_PC to the domain and tinkered around with users, groups and permissions. I used this script from Josh Madakor's video to create around 1000 users.

![image](https://github.com/user-attachments/assets/17e12a8c-8c2b-4740-89c0-3d2c8dc7260b)
![image](https://github.com/user-attachments/assets/6b2a4d31-2144-4d91-94b1-7f845551ae6a)

---

## Performing a Brute force attack on target_PC and reviewing events via Splunk

I used crowbar to launch a brute force dictionary attack on target_PC from the Kali Linux machine. I had enabled RDP on target_PC beforehand so this attack would be feasable.

![image](https://github.com/user-attachments/assets/702ed5c2-43ee-4ad7-9e37-af9dd8a53783)

After running the attack, we can see that Splunk recorded 42 events with event code 4265, which indicates failed login attempts. This corresponds to the 22 passwords in the wordlist I used for the attack, which was run twice.

Among these, there are two events with event code 4264, representing successful login attempts. This outcome is expected since one of the passwords in the wordlist was of course correct.

![image](https://github.com/user-attachments/assets/9ee0b899-a988-409f-a247-2fa2b2b97ba7)
![image](https://github.com/user-attachments/assets/c3cbda96-b694-4161-851b-0ed241454218)

Here we can see that the attack indeed came from the Kali machine at 192.168.10.250

![image](https://github.com/user-attachments/assets/c2fddb32-9dba-449a-8a39-56d89ef411c9)

---

## Installing Atomic Red Team, Performing a Test, and Reviewing Events in Splunk

Atomic Red Team is an open-source project that offers a collection of tests to simulate cyberattacks based on the MITRE ATT&CK framework.

Before installing Atomic Red Team (ATR) on target_PC, I excluded the C: drive (where ATR will be installed) from Microsoft Defender Anti-Virus scans. Note: This exclusion is not recommended for normal circumstances.

To allow PowerShell scripts to run without restrictions for the current user, I used the command:
```Set-ExecutionPolicy Bypass -Scope CurrentUser```

![image](https://github.com/user-attachments/assets/ef8bd613-2f4c-4793-bc62-292cbefd04d9)
![image](https://github.com/user-attachments/assets/d80423a2-6935-4b59-aedf-41177894af3a)

Next, I installed ATR using the following commands:

![image](https://github.com/user-attachments/assets/f45a5257-c796-402b-bf75-208a7b01ded9)

Now we can view all the tests available in Atomic Red Team. Each test is named after the corresponding MITRE ATT&CK technique. For example, I ran the T1136.001 test, which corresponds to the "Create Account: Local Account" persistence technique in MITRE ATT&CK.

![image](https://github.com/user-attachments/assets/ae03652b-1642-41ad-b566-343fb54ed921)
![image](https://github.com/user-attachments/assets/7a2eeddd-8b09-4634-ada3-3fac60511ec5)


Running the test created a user called NewLocalUser, added it to the local administrators group, and finally deleted the user.

![image](https://github.com/user-attachments/assets/b3db767e-0ae3-4c01-80b3-5e9cd46cbb3d)

We see these events in Splunk.

![image](https://github.com/user-attachments/assets/a22d3a1b-b43b-4b8c-9934-efac7d765768)

Here are the corresponding event codes:
- 4798: A user's local group membership was enumerated.
- 4738: A user account was changed.
- 4720: A user account was created.
- 4722: A user account was enabled.
- 4724: An attempt was made to reset an account's password.
- 4726: A user account was deleted.

![image](https://github.com/user-attachments/assets/1fece8c6-2615-4dff-895b-d25b59df7879)

Below is the final event showing "NewLocalUser" being deleted

![image](https://github.com/user-attachments/assets/19e5c522-9791-479c-9836-3f055916388f)

---

# Conclusion

This lab has been a fantastic learning experience for me. I've set up Splunk Enterprise on Ubuntu, deployed Splunk forwarders, performed tests with Atomic Red Team, and analyzed results in Splunk following the MITRE ATT&CK framework. I also used Kali Linux to simulate a brute force attack and reviewed the outcome in Splunk.

Through setting all this up, I've sharpened my skills in virtualization with VirtualBox, general Windows and Linux processes and learned the essentials of setting up Active Directory on Windows Server.

I'm excited to keep building on this lab. My next steps include creating Splunk alerts, tweaking group policies, and just generally tinkering around in the domain environment. There's always more to learn, and I'm looking forward to it!
