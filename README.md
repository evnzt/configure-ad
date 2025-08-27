<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>

This guide walks you through deploying an ***on‑premises‑style* Active Directory** environment using **Azure Virtual Machines**.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Provision **DC‑1** (Windows Server) and **Client‑1** (Windows 10) in Azure.
- Configure static addressing on DC-1 (NIC)
- Ensure **network connectivity** between Client‑1 and DC‑1.
- Install **Active Directory Domain Services** and promote DC‑1 to a new forest.
- Create **OUs** and **user accounts** (admin + standard).
- Point **Client‑1 DNS** to DC‑1, **join domain** (e.g., `mydomain.com`).
- Enable **Remote Desktop** for non‑administrator domain users on Client‑1.
- (Optional) **Bulk‑create** test users and sign in with one of them.

<h2>Deployment and Configuration Steps</h2>

__Step 1__: Deploy Azure Resources

*Create two VMs in the same **Resource Group** and **Virtual Network**.*

1. **Create DC‑1** (Windows Server 2022)
2. **Create Client‑1** (Windows 10)
3. Ensure **both VMs** are in the **same VNet/subnet**
  
<img width="1040" height="313" alt="vms" src="https://github.com/user-attachments/assets/f514570e-ff72-474f-b07c-67520bb8eed9" />

**Notes & Tips:**

* Choose the **region** closest to you.
* On **Azure Free** subscription, VM sizes vary by region/zone—pick supported sizes and place both VMs in the **same availability zone**.
* **DC‑1** should be **Windows Server**; **Client‑1** is **Windows 10**.
* A size like **2 vCPU / 16 GiB** is sufficient for this lab.
* **Record** the admin **username/password** you set during VM creation.
* If the VNet created with DC‑1 doesn’t appear when creating Client‑1, wait a minute and retry; it should show under **Virtual network**.

---

__Step 2__: Set DC‑1 NIC to Static IP

*Give DC‑1 a **static private IP** so DNS and domain services remain stable.*

1. Open **DC‑1 → Networking → Network settings → IP configurations**.
2. Select **ipconfig1** and change **Allocation** to **Static**.
3. **Save** the change.

---

__Step 3__: Verify Client ↔ DC Connectivity

*Confirm Client‑1 can reach DC‑1.*

<img width="407" height="494" alt="428757709-0d87934c-ecbf-4062-8e22-dd29b5c00add" src="https://github.com/user-attachments/assets/2e860c01-b9cc-45f0-aa53-7ee040c0e0be" />

1. **Remote Desktop Protocol (RDP)** into **Client‑1** with the credentials created at deployment.
2. From the **Azure portal**, copy **DC‑1’s private IP** (Networking tab).
<img width="1457" height="718" alt="Priv IP" src="https://github.com/user-attachments/assets/b8b7be63-6c01-415d-9f87-c6f6b39d9ed1" />

3. On Client‑1, open **Command Prompt** and start a continuous ping:

   ```cmd
   ping -t <DC1_PRIVATE_IP>
   ```

   It may time out initially due to DC‑1 firewall.
4. **RDP** into **DC‑1** and open **Windows Defender Firewall with Advanced Security** → **Inbound Rules**.
5. Sort by **Protocol**, find **ICMPv4** rules, and **Enable** the echo request rules.
<img width="916" height="342" alt="428757709-0d87934c-ecbf-4062-8e22-dd29b5c00add" src="https://github.com/user-attachments/assets/c4044075-8d6b-47b5-a48e-46091f698aa9" />


6. Return to **Client‑1** and verify **Replies** are now received.

<img width="759" height="555" alt="428757709-0d87934c-ecbf-4062-8e22-dd29b5c00add" src="https://github.com/user-attachments/assets/8ff79311-a599-42f6-acd0-a581e1067e2a" />

---

__Step 4__: Install AD DS & Promote DC‑1

*Install **Active Directory Domain Services** and create a **new forest/domain**.*

1. On **DC‑1**, open **Server Manager** → **Add Roles and Features**.
2. In **Server Roles**, select **Active Directory Domain Services**; proceed to **Install**.
<img width="759" height="555" alt="333888925-e1bbdf7a-e940-424a-8b01-9bdd07d10175" src="https://github.com/user-attachments/assets/dc77b33a-fb5f-46dc-b471-d88977aba8ac" />

3. After installation, click the **yellow flag** in Server Manager → **Promote this server to a domain controller**.
<img width="759" height="555" alt="68747470733a2f2f692e696d6775722e636f6d2f59343558427a4c2e706e67" src="https://github.com/user-attachments/assets/022d78e1-2405-4c2c-a8d3-f8a56e35eda4" />

4. Choose **Add a new forest** and specify a domain name (e.g., `mydomain.com`).
5. Set a **DSRM password** (note it down for lab purposes).
6. Complete the **Prerequisites Check** and **Install**.
<img width="759" height="555" alt="333889334-84311ab0-bc5c-4f03-97b9-6d8b84cc2594" src="https://github.com/user-attachments/assets/890abf71-f264-4a6f-882c-0c9d0b8633b7" />

7. **Reboot** if not automatically restarted.
8. Reconnect to **DC‑1** (your sign‑in name may change to the domain format).

---

__Step 5__: Create Organizational Units (OUs) and Admin/Standard Users

*Organize your directory and create administrative and standard accounts.*

1. On **DC‑1**, open **Tools → Active Directory Users and Computers (ADUC)**.
2. Create some **Organizational Units** (OUs), e.g., `_EMPLOYEES` and `_ADMINS` (use a prefix like `_` to make lab OUs easy to find).
<img width="750" height="524" alt="333889334-84311ab0-bc5c-4f03-97b9-6d8b84cc2594" src="https://github.com/user-attachments/assets/95659680-a907-4bce-8495-a4a099f7da2a" />

3. Create your **admin account** in the `_ADMINS` OU. *Consider*:
   * Unchecking **User must change password at next logon** (for lab simplicity).
   * Checking **Password never expires** (lab only).
4. Make the new admin a **member of** `Domain Admins` (ADUC → user **Properties** → **Member Of** → **Add** → type `Domain Admins`).
5. Sign out and **sign in** as `MYDOMAIN\your_admin` (or `your_admin@mydomain.com`).

---

__Step 6__: Join Client‑1 to the Domain

*Configure DNS and join **Client‑1** to the **MYDOMAIN** domain.*

**Pre‑Req:** Point **Client‑1 DNS** to **DC‑1’s private IP**.

1. In the **Azure portal**, note **DC‑1’s private IP** (Networking → private IP).
2. On **Client‑1 → Networking → DNS servers**, switch to **Custom** and enter **DC‑1’s private IP**.
<img width="1010" height="383" alt="333889334-84311ab0-bc5c-4f03-97b9-6d8b84cc2594" src="https://github.com/user-attachments/assets/9a28b826-d565-424b-8b7e-e941ce467a51" />

3. **Save**, wait for update, then **Restart** Client‑1 from the portal to flush DNS.
4. After reboot, **RDP** into Client‑1.
5. Right‑click **Start → System → Rename this PC (advanced)** → **Change**.
6. Select **Domain**, enter `mydomain.com` (or your chosen domain), and **OK**.
<img width="409" height="464" alt="333894143-9bc26a02-a162-4580-89fd-e8a1251dc9a7" src="https://github.com/user-attachments/assets/2ad37ea2-8bf1-4479-8b11-9663b22e336a" />

7. When prompted, use **domain credentials** (your domain admin from step 5).
8. Reboot when prompted; then sign in using **domain credentials**.

---

__Step 7__: Enable RDP for Non‑Admins on Client‑1

*Allow all **Domain Users** to log in via **Remote Desktop**.*

<img width="483" height="666" alt="333894143-9bc26a02-a162-4580-89fd-e8a1251dc9a7" src="https://github.com/user-attachments/assets/dd2bdb20-c3d4-4ec3-8b91-971b03cc3cc1" />

1. On **Client‑1**: **System → Remote Desktop → Select users… → Add…**
2. Type `Domain Users`, click **Check Names**, then **OK**.
<img width="455" height="249" alt="68747470733a2f2f692e696d6775722e636f6d2f613875594738582e706e67" src="https://github.com/user-attachments/assets/421c763b-72ab-44f5-832d-9577c1c4561e" />
  
3. All members of **Domain Users** can now RDP to Client‑1.
4. On **DC‑1**, open **ADUC → Users → Domain Users → Members** to see group membership.

---

__Step 8__: *Optional*: Bulk‑Create Users via PowerShell

*Populate `_EMPLOYEES` OU with many test users, then sign in as one from Client‑1.*

1. On **DC‑1**, open **PowerShell ISE as Administrator**.
2. Use the community script (credit below) to generate users:
> [https://github.com/joshmadakor1/AD\_PS/blob/master/Generate-Names-Create-Users.ps1](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1)
3. **Copy the raw code** and paste into a new ISE script file.
4. (Recommended) **Reduce** the number of accounts (e.g., from 10k to **1k**) for a faster lab run.
5. **Run** the script (**F5**) and watch accounts being created in `_EMPLOYEES`.
<img width="795" height="900" alt="333894143-9bc26a02-a162-4580-89fd-e8a1251dc9a7" src="https://github.com/user-attachments/assets/6398bf3b-3383-42a5-ae14-30c3f560d5fe" />

6. In **ADUC**, pick a random user → **Properties → Account** to view the **logon name**.
7. **Sign out** of Client‑1 and **RDP** back in using the chosen user and the script’s default password (usually `Password1`).
8. If an account becomes **locked**, on DC‑1 you can **Unlock**, **Reset password**, or **Disable** via **ADUC**.

<img width="405" height="490" alt="xul" src="https://github.com/user-attachments/assets/a5fb89a0-0cdd-4b9a-8923-774a14f0e6b1" />
<img width="928" height="709" alt="xul start" src="https://github.com/user-attachments/assets/1a70a659-00da-4e98-919f-4d03c6bf9918" />
<br />
<br />

**Credit:** Script by **Josh Madakor**.

---

Congratulations! Hopefully, the installation has been completed without any errors.

---

## Troubleshooting

* **Ping fails from Client‑1:** Ensure ICMPv4 inbound rules enabled on DC‑1; confirm both VMs share the **same VNet**.
* **Domain join fails:** Verify Client‑1 **DNS** points to **DC‑1’s private IP**. Reboot Client‑1 after DNS change.
* **Slow logons or name resolution issues:** Check **DNS** and **time sync**; ensure DC‑1 has a **static IP**.
* **Cannot RDP as standard user:** Confirm **Domain Users** is added to **Remote Desktop Users** on Client‑1.

---

## Cleanup

* Delete the **Resource Group(s)** and **VM(s)** in Azure once done to avoid charges.

<h2>Closing Thoughts</h2>

Building an on-premises Active Directory lab in Azure is a practical peek into real IT operations. I covered identity, networking, security, and automation in a hands-on exercise that maps directly to day-to-day responsibilities across IT roles.
</p>
<br />
