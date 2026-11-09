# JML-Identity-Lifecycle-Lab-MS-Entra-ID-
Overview
In this lab I simulated the full identity lifecycle of employees in Microsoft Entra ID: onboarding a new hire, processing an internal promotion, and offboarding a departing employee. This process is known as Joiner–Mover–Leaver (JML), and it's one of the core responsibilities of Identity and Access Management (IAM).

I built a small fictional organization, Medical Health Clinic IT, and walked three users through each stage, verifying every change and documenting it with screenshots and audit-log evidence.

| Stage | Scenario |
|---------|-------------|
| Joiner | Fred Taylor is hired as a Sales Analyst |
| Mover | Jesse Junior is promoted from Service Desk Analyst to Security Analyst |
| Leaver | Mike Jones, a Sales Analyst, retires |

Skills demonstrated
+ User provisioning and attribute management (job title, department, manager,)
+ Security group design and group-based access control
+ Least privilege and preventing privilege creep
+ Microsoft Entra admin role assignment and removal
+ Secure offboarding: removing access, blocking sign-in, revoking sessions, soft-delete
+ Auditing and evidence collection with Entra ID audit and sign-in logs
+ Troubleshooting an unexpected platform behavior during offboarding


1. Building the Environment
   
Before running any lifecycle events, I needed an organization to work in: departments with defined access, and existing employees.

Security groups
Rather than assigning access to people one at a time, I used group-based access control: access is granted to a group, and people are added to or removed from that group as their role changes. This keeps JML manageable and auditable.
I created four security groups using a consistent naming convention (SEC- for security group, Dept- for department-based):

| Group | Purpose | 
|---------|-------------|
| SEC-AllStaff-Users | All active employees |
| SEC-Dept-Sales | Sales POS systems and Revenue |
| SEC-Dept-IT-ServiceDesk | Ticketing and support tools |
| SEC-Dept-Security | Security tooling and logs |

All groups used Assigned Membership, meaning an Admin controls who is in them.

<img width="1027" height="606" alt="Screenshot 2026-11-04 222505" src="https://github.com/user-attachments/assets/79a960df-d6d9-4990-b88d-c5a63c22f0f0" />

Existing employees

I created four existing employees, each with a job title, department, and manager. Managers matter because in real organizations they approve access requests and are involved in deprovisioning.


| Name | Job title | Department | Manager
|---------|-------------|---------|-------------
| Mike Jones | Sales Manager | Sales | -------
| Sam White | Director of IT | IT | ------- 
| Jesse Junior | Service Desk Analyst | IT Service Desk | John Reyes
| Fred Taylor | Sales Analyst | Sales | Mike Jones 

I added everyone to SEC-AllStaff and to their department's group.

<img width="580" height="270" alt="Screenshot 2026-11-05 192523" src="https://github.com/user-attachments/assets/ee1c6a4f-a0cf-47b9-b90d-f7e52a35ec95" />

Admin role

Help desk staff need to reset users' passwords, so I assigned Jesse the Helpdesk Administrator role, even though his role is Service Desk, these roles are similar in real IT environment. 

<img width="501" height="313" alt="Screenshot 2026-11-05 195808" src="https://github.com/user-attachments/assets/b7f6978b-bc60-4d0e-9d43-a8dab052b43d" />

Group membership controls access to resources, while Entra admin roles control what someone can do to the directory itself. Because admin roles are high-risk, they need to be removed as soon as someone no longer needs them, which became important in the Mover stage.

2. Joiner — Fred Taylor

Scenario: HR notified IT that Fred Taylor was hired as a Sales Analyst reporting to Mike Jones.

Account creation


I created Fred's account with his job title, department, and manager, and used an auto-generated temporary password. IT should never know a user's real password; the temporary password is handed off securely, and Entra forces the user to set their own at first sign-in.

<img width="520" height="597" alt="Screenshot 2026-11-05 200840" src="https://github.com/user-attachments/assets/997fb7e3-4327-4a8a-b568-1df0ca24413d" />

Least-privilege access

I added Fred to only SEC-AllStaff-Users and SEC-Sales-Users. He received no Help Desk or Security access and no admin roles. This is least privilege: giving people only the access their role requires.

<img width="944" height="270" alt="Screenshot 2026-11-05 201756" src="https://github.com/user-attachments/assets/d8748e9e-04f6-4cf5-a02a-a37d77b4be9c" />

Day-one verification
I signed in as Fred in a private browser window to confirm the account worked, completed the forced password change, and confirmed He could reach his account page. A joiner process isn't complete until the new hire can actually work on day one.

<img width="592" height="321" alt="Screenshot 2026-11-05 202913" src="https://github.com/user-attachments/assets/8c6724a2-2593-4598-84d9-a145ee97c4ca" />

3. Mover — Jesse Junior
   
Scenario: Jesse Junior was promoted from Help Desk Analyst/SD to Security Analyst. Avoiding privilege creep

<img width="765" height="315" alt="Screenshot 2026-11-05 205404" src="https://github.com/user-attachments/assets/4da04da8-4355-4690-b923-6d947779a397" />

The most common mistake in a mover process is only adding access for the new role. Over time, someone who moves through several teams accumulates access from all of them. This is known as privilege creep, and it means a compromised account exposes everything the person ever had. A correct mover process both adds new access and removes old access.

Before
I documented Jesse’s access before making any changes, to create an audit trail.

<img width="839" height="322" alt="Screenshot 2026-11-05 204728" src="https://github.com/user-attachments/assets/1ab90597-4e32-41d4-b4b2-0edbe457c25a" />

Changes made 

Updated attributes: Changed Job title to Security Analyst and Department to Security. These attributes are the source of truth for directories, org charts, and automated access rules. Added new access: added Jesse to SEC-Dept-Security and assigned the Security Administrator role. Removed old access: removed Jesse from SEC-Dept-IT-HelpDesk and removed the Helpdesk Administrator role.

<img width="645" height="370" alt="Screenshot 2026-11-05 205201" src="https://github.com/user-attachments/assets/f9508218-a30c-47dd-9a4c-ecafcadbc677" />

Jesse kept SEC-AllStaff and the new Security access and lost both the Help Desk group and the Helpdesk Administrator role.

4. Leaver — Mike Jones Scenario: Mike Jones retired, and HR asked IT to remove access on the last day. To test the process realistically, I first signed in as Mike in a private browser window and left the session open, simulating an employee who is still logged in at their desk. Lockout Blocked sign-in by disabling the account, which stops any new sign-ins. Revoked sessions. Blocking sign-in doesn't remove someone who is already signed in, because their apps hold tokens that stay valid for a while. If the system times out and the session needs to be renewed or, revoking sessions invalidates those tokens and forces re-authentication, which then fails because the account is blocked.


<img width="661" height="258" alt="Screenshot 2026-11-05 214114" src="https://github.com/user-attachments/assets/85195456-6377-42c9-9435-db64c6486393" />

Verifying the lockout
I opened a new private window and tried to sign in as Mike with the password he had been using. The sign-in failed with an error. This confirmed the control actually worked, not just that the setting was changed.

<img width="630" height="210" alt="Screenshot 2026-11-05 214043" src="https://github.com/user-attachments/assets/7915ca7e-2ed1-44bc-9a6a-fea2fa1a8eb5" />

Removing access
With the account locked out, I removed Mike from SEC-Dept-Sales and SEC-AllStaff and confirmed no admin roles were assigned. Removing access after the lockout ensures there's no window where the user can still access.

<img width="708" height="466" alt="Screenshot 2026-11-05 214337" src="https://github.com/user-attachments/assets/9a3ba34a-3f9d-4197-804c-7da223c4c041" />

I then updated Mike's job title to Former Employee - Sales Analyst and department to Terminated to record the departure. Account was created in Active Directory, can’t change in Entra.  I will have to wait to Entra Connect resyncs to completely remove the account. Entra usually resynch every 10 mins intervals. You could also push a powershell command to manually force sync. 

Open PowerShell as an Administrator on your Entra Connect server and execute  command:
PowerShell
Start-ADSyncSyncCycle -PolicyType Delta


In production, accounts are usually kept disabled for a retention period so the manager can recover files and email. For this lab I deleted the account to demonstrate Entra ID's soft delete: deleted users stay recoverable for 30 days before permanent deletion, which protects against offboarding the wrong person.

5. Audit and Evidence Every change in this lab was recorded in Entra ID's logs. In real environments, this is how IT proves to auditors that a process was followed, and how incidents get investigated.
Audit log
The audit log captured each lifecycle event: user creation, group membership changes, role assignments and removals, attribute updates, and the deletion.

<img width="1846" height="750" alt="Screenshot 2026-11-05 221827" src="https://github.com/user-attachments/assets/ed354c8f-1ee5-4ae4-9419-da214af51543" />

6. Key Findings
Order of offboarding steps matters in Entra ID. I planned to block sign-in first and reset the password afterward, but Entra ID won't reset the password on a disabled account. The account was still fully locked out, but the finding changed how I'd write an offboarding runbook: reset the password first, then immediately block sign-in, then revoke sessions, all back to back.
The mover stage is where access risk hides. Adding access for a new role is the easy part. Removing the old group and the old admin role is what prevents privilege creep.
Verification is part of the process. Testing Fred's first sign-in and attempting to sign in as Mike after offboarding confirmed that each control actually worked.




<img width="452" height="681" alt="Screenshot 2026-11-05 222000" src="https://github.com/user-attachments/assets/7813cdc0-6b26-4640-a52c-c1ff304285e9" />


7.	How This Works in the Real Company
I performed this lab manually to show each underlying step. In a real organization, most of this is automated:

| Manual step in this lab |How it's automated in production |
|---------|-------------|
|Creating the user by hand | HR-driven provisioning: the HR system (e.g., Workday, Saleforce, ICMS) is the source of truth and automatically creates the account in Entra ID. On-prem environments sync from Active Directory using Entra Connect. |
| Adding users to department groups | Dynamic groups (Entra ID P1): a rule like user.department -eq "Sales" adds and removes members automatically when the Department attribute changes. Jesse would have been moved between groups just by updating his department. |
| Mover access review | Access reviews (Entra ID Governance): managers periodically confirm whether their team still needs access. |
| Leaver checklist | Lifecycle Workflows (Entra ID Governance): pre-built joiner, mover, and leaver tasks that run automatically based on hire and leave dates. |
| Permanent admin roles | Privileged Identity Management (PIM) (Entra ID P2): admins get roles just-in-time, for a limited period and with approval, instead of permanently. |
| Leaver's mailbox and files | In Microsoft 365, the mailbox is often converted to a shared mailbox, and OneDrive access is handed to the manager before the license is removed. |
| Keeping logs |Logs are streamed to a SIEM (e.g., Microsoft Sentinel, Splunk) for long-term retention and alerting. |




























