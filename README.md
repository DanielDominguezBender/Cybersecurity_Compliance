# Cybersecurity_Compliance
Compliance homelab

## Intro

In this lab I recreated a corporate Domain Controler using a Windows Server 2019 machine and a Client machine with a W10 Pro edition (both machines are in a Virtual Environment using Virtualbox).
Domain Controler machine (DC01), will handle the departments, users, client machine and the GPOs.
The Client machine (Client01), will be a simple windows machine managed by the different user accounts.
Both machines connect each other in a Host-Only network using a network adapter called <i>vboxnet1</i>, with a static IP in both machines.

I recreated all in a MacBook Pro late 2011. Reason? This laptop still uses Intel Chip, which for many other labs I've done, recreating such scenarios was much easier than in Mac's with Apple chip (for example, GNS3 labs).

## ✅ Step-by-Step Cybersecurity Compliance Home Lab Setup
	
### 🧰 **Step 0: Requirements**

**Hardware:**

- MacBook Pro late 2011: **16 GB RAM** and **100 GB free disk** (or more)
- Virtualization platform: **VirtualBox**

**Software/ISOs:**

- Windows Server 2019 ISO (for Domain Controller)
- Windows 10 ISO (client)

**Optional**
- Ubuntu Server 22.04 ISO (Linux server)
- Tools: Wazuh, Nessus, OpenVAS, Snort, Lynis, etc.

---

### 🏗️ **Step 1: Create the Virtual Network**

Iv'e used **VirtualBox Host-Only Network** (or NAT Network + internal networking) so VMs can communicate with each other.

1. Open **VirtualBox**.
2. Go to **File > Host Network Manager**.
3. Create a new Host-Only Adapter (`vboxnet1`).
4. Enable DHCP (optional).
5. Assign static IPs inside each VM (e.g., DC01: 192.168.56.10, Client01: 192.168.56.20).

---

### 🖥️ **Step 2: Set Up the Domain Controller (Windows Server 2019), called DC01**

This will be the **Active Directory** and **Group Policy** testing machine.

**Steps:**

1. Create a VM, assign 4 GB RAM, 2 CPUs, 40 GB disk.
2. Install Windows Server 2019.
3. Set static IP (e.g., `192.168.56.10`).
4. Rename the computer (`DC01`).
5. Promote it to a Domain Controller:

    - Open Server Manager → Add Roles & Features.
    - Select **Active Directory Domain Services**.
    - Promote the server to a domain controller (`lab.local`).

6. Create OUs and test users (I created `Alice`, `Bob` and `Dani`).
7. Create different departmenrs such as: `Finance`, `HR` and `IT`.

8. Configure **Group Policies**:
	- Password complexity
  - Account lockout policy
  - Audit logs

---

### 💻 **Step 3: Set Up Windows 10 Client, called Client01**

This machine will be used to simulata an employee workstation for testing **endpoint security**, GPOs, and vulnerability scans.

**Steps:**

1. Create a VM, assign 4 GB RAM, 2 CPUs, 25 GB disk.
2. Install Windows 10 (the Pro version).
3. Set static IP (`192.168.56.20`).
4. Join it to your domain `lab.local`.
5. Log in with a domain user to test policies.

---

### 🖥️ **Step 4: Rename the Machines**

In order to rename the Domain Controller and Client machines, I did the following.

Opened PowerShell or Command Prompt as Admin:

In case of DC01:

```Powershell
rename-computer -newname DC01 -force
```

In case of Client01:

```Powershell
rename-computer -newname Client01 -force
```

Then I restarted the VM's.

---

### 🔁 **Step 5: Install Active Directory Domain Services (AD DS)**

I Opened Server Manager:

    Click “Add Roles and Features”

    Click Next through the wizard until Server Roles

    Check Active Directory Domain Services

    Accept any prompts and dependencies

    Click Next → Install

    Wait for install to complete

### 🏛️ **Step 6: Promote Server to a Domain Controller**

After installation, a flag will appear in Server Manager. I clicked on “Promote this server to a domain controller.”

Choose:

    Add a new forest

    Root domain name: lab.local

Then:

    Set DSRM (Directory Services Restore Mode) password

    Leave default paths

    Review + install

    VM will restart automatically after promotion

---

### 👥 **Step 7: Create Organizational Units (OUs) & Users**

I opened Active Directory Users and Computers:

    Create a new OU: LabUsers

    Add sample users:

        Alice Analyst

        Bob Auditor

        Set complex passwords, check “User must change password at next logon” (optional)

---

So far, I have explained how to create and set upthe VM's used in this lab
Now it's time to set up some GPO's.

---

### 🛡️ **Step 8: Configure Group Policy (GPO)**

Now I opened the Group Policy Management Console (GPMC).

Some examples in how to create and link a new GPO to the domain the OU could be:

✅ Password Policy:

    Path: Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy

        Minimum length: 12

        Enforce complexity: Enabled

        Max password age: 30 days

✅ Account Lockout:

    Lockout after 5 failed attempts

    Lockout duration: 15 minutes

✅ Audit Policy:

    Enable audit logon events (Success/Failure)

    Enable account management auditing

---

### :pencil: **Step 9: Example of GPO I created:

 - User Alice cannot install software (as she is in Finance dept)
 - User Dani is allowed to install software (as he is in IT dept)

In the DC01 machine, I created two different GPO's under the `Finance` and the `IT`department:

 - Allow Software installation (without admin rights) - applied to IT
 - Cannot install software - applied to Finance

To allow the user `Dani` in the IT department to install software on his machine:

1. Open Group Policy Management on DC01.
2. Right-click the IT OU → choose Create a GPO in this domain, and Link it here.
  - Name it: Add IT Users to Local Admins
3. Right-click the GPO → Edit
4. Navigate to:

```linux
Computer Configuration → Preferences → Control Panel Settings → Local Users and Groups
```

5. Right-click → New → Local Group

- Action: Update
- Group Name: Administradores ← in my case I chosed this name as the **Client01** had SPANISH as O.S. language.
- Click Add… next to Members
  
```linux
Click Browse… → search and select user(s): DOMAIN\user_name
```

✅ It should look like:

```pgsql
Group: Administradores
Action: Update
Members: LAB\dbender
```

![image1](imgs/image1.png)  

Then I made following steps under **Client01**.

Forced Group Policy Update on CLIENT01, by logging in as any user (I did as Alice).
Opened CMD and run:

```Powershell
gpupdate /force
```

![image2](imgs/image2.png)  

Then I checked the membership by running:

```Powershell
net localgroup administradores
```

> [!NOTE]
> Remember that I have **Client01** in Spanish, that's why I used `administradores` instead of `administrators`.

✅ It should look like:

```pgsql
Administratores
LAB\dbender
```

![image3](imgs/image3.png)  

---

By opening session in **Client01** as `Alice`, I should not be able to install any software.

![movie1](imgs/movie1.gif)

On the other hand, using **Client01** as `Dani` should not give me any problems to install VLC.

![movie2](imgs/movie2.gif)

Seems that the `Allow Software installation`GPO is working!!

