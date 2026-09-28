# AD Help Desk Lab — Skills Checklist

Every discrete skill needed to build this lab again from nothing, without AI
assistance, in the order you'd hit them.

**How to use it:** mark each line honestly.

- `[ ]` I could not do this unaided
- `[~]` I could do it with documentation and some searching
- `[x]` I could do it cold, and explain *why* it works

Anything still at `[ ]` or `[~]` after the lab is a gap an interviewer can find.

---

## Skills used continuously, not at one step

- [ ] 001. Reading official vendor documentation (Microsoft Learn) instead of a random blog
- [ ] 002. Searching an exact error string or error number rather than describing the symptom
- [ ] 003. Judging source quality — official docs, dated forum posts, Server Fault answers
- [ ] 004. Reading Windows Event Viewer and finding the relevant log for a given failure
- [ ] 005. Changing one variable at a time when troubleshooting
- [ ] 006. Forming a hypothesis before making a change, and stating what result would disprove it
- [ ] 007. Working from the bottom of the stack upward (link → IP → name → service → app)
- [ ] 008. Keeping a running lab journal of what you changed and when
- [ ] 009. Taking a snapshot before any change you can't easily undo
- [ ] 010. Time-boxing a stuck problem and deliberately backing out instead of grinding
- [ ] 011. Verifying a change from the user's side, not just the admin console

---

## Phase 0 — Planning and prerequisites

- [ ] 012. Defining what the lab is for and which job role it supports
- [ ] 013. Assessing host capacity — RAM, CPU cores, free disk — against planned VMs
- [ ] 014. Checking CPU virtualization support and enabling VT-x/AMD-V in BIOS/UEFI
- [ ] 015. Choosing a hypervisor and justifying it (VirtualBox vs Hyper-V vs VMware)
- [ ] 016. Installing VirtualBox and the Extension Pack
- [ ] 017. Knowing where to obtain legitimate Windows Server evaluation media
- [ ] 018. Downloading a Windows 10/11 ISO with the Media Creation Tool
- [ ] 019. Verifying an ISO's checksum before using it
- [ ] 020. Knowing Windows edition differences, especially Pro vs Home for domain join
- [ ] 021. Choosing a host naming convention (AD-DC01, AD-CL01) and applying it consistently
- [ ] 022. Choosing a domain name, and knowing the tradeoffs of `.local` vs a real owned domain
- [ ] 023. Planning an IP scheme: subnet, mask, which addresses are reserved for servers
- [ ] 024. Setting up a folder structure for ISOs, VM disks, notes and screenshots

## Phase 1 — Virtual machines and OS installation

- [ ] 025. Creating a VM and picking the right OS type and version
- [ ] 026. Sizing RAM and vCPU per role, and knowing the host's limits
- [ ] 027. Creating a virtual disk and choosing dynamic vs fixed allocation
- [ ] 028. Attaching an ISO to the virtual optical drive and setting boot order
- [ ] 029. Running Windows Server setup and choosing Desktop Experience vs Server Core
- [ ] 030. Partitioning during a custom install
- [ ] 031. Setting and recording the local Administrator password safely
- [ ] 032. Installing Guest Additions and knowing what they provide
- [ ] 033. Sending Ctrl-Alt-Del into a guest, and knowing what the host key is
- [ ] 034. Adjusting guest display scaling and resolution
- [ ] 035. Renaming a Windows computer and rebooting to apply it
- [ ] 036. Setting the time zone and confirming the clock is accurate
- [ ] 037. Taking a snapshot with a name and description that will make sense in six months
- [ ] 038. Restoring a snapshot, and knowing what state is lost when you do
- [ ] 039. Understanding snapshot chains and their disk cost
- [ ] 040. Installing Windows 10 Pro, skipping the product key, creating a local account offline
- [ ] 041. Identifying the installed Windows edition and build (`winver`)
- [ ] 042. Upgrading Home to Pro with a generic key when domain join is greyed out

## Phase 2 — Networking

- [ ] 043. Explaining an IPv4 address as a network portion plus a host portion
- [ ] 044. Using a subnet mask to decide whether two addresses are on the same network
- [ ] 045. Explaining what a default gateway does, and when you don't need one
- [ ] 046. Explaining what DNS does, and how it differs from routing
- [ ] 047. Comparing VirtualBox network modes: NAT, NAT Network, Bridged, Host-only, Internal
- [ ] 048. Explaining why default NAT isolates VMs from each other
- [ ] 049. Configuring an Internal Network with the same network name on both VMs
- [ ] 050. Running `ipconfig /all` and interpreting every field it prints
- [ ] 051. Recognising an APIPA address (169.254.x.x) and knowing what it proves
- [ ] 052. Explaining DHCP lease mechanics: obtain, expire, renew
- [ ] 053. Explaining why a domain controller must not use a DHCP-assigned address
- [ ] 054. Setting a static IP, mask, gateway and DNS through the GUI
- [ ] 055. Doing the same in PowerShell (`New-NetIPAddress`, `Set-DnsClientServerAddress`)
- [ ] 056. Re-checking configuration after a change instead of assuming it saved
- [ ] 057. Testing with `ping` and distinguishing reply, timeout, and "transmit failed"
- [ ] 058. Recognising when a successful ping proves nothing (a host pinging itself)
- [ ] 059. Separating connectivity failure from name resolution failure
- [ ] 060. Using `nslookup` for forward lookups, and reading its "UnKnown server" noise correctly
- [ ] 061. Explaining loopback (127.0.0.1) versus a host's LAN address
- [ ] 062. Checking Windows Firewall rules for ICMP and File and Printer Sharing
- [ ] 063. Proofreading typed IP addresses digit by digit

## Phase 3 — AD DS installation and promotion

- [ ] 064. Explaining what a directory service is and what problem AD solves
- [ ] 065. Distinguishing forest, tree, domain, OU and container
- [ ] 066. Adding the AD DS role through Server Manager
- [ ] 067. Explaining the difference between installing the role and promoting the server
- [ ] 068. Running the promotion wizard and choosing a new forest
- [ ] 069. Choosing forest and domain functional levels, and knowing what they restrict
- [ ] 070. Setting a DSRM password and explaining what Directory Services Restore Mode is for
- [ ] 071. Explaining the NetBIOS domain name and why it still exists
- [ ] 072. Interpreting the DNS delegation warning during promotion
- [ ] 073. Verifying promotion three ways: roles, Tools menu, domain membership
- [ ] 074. Reading the Directory Service and DNS Server event logs after a failure
- [ ] 075. Explaining what SYSVOL and NETLOGON are and what they hold
- [ ] 076. Confirming the forward lookup zone for the domain exists in DNS
- [ ] 077. Querying SRV records (`nslookup -type=SRV _ldap._tcp.dc._msdcs.<domain>`)
- [ ] 078. Explaining Kerberos's five-minute clock skew tolerance and the PDC emulator's role
- [ ] 079. Knowing which ports domain services use: DNS 53, Kerberos 88, LDAP 389, SMB 445

## Phase 4 — Domain join

- [ ] 080. Pointing a client's DNS at the domain controller and explaining why
- [ ] 081. Confirming the client can resolve the domain name before attempting a join
- [ ] 082. Joining a computer to a domain through System Properties (`sysdm.cpl`)
- [ ] 083. Knowing the PowerShell equivalent (`Add-Computer`)
- [ ] 084. Supplying credentials in the right format (`DOMAIN\user` or UPN) and explaining why they're required
- [ ] 085. Explaining the computer account object that gets created in AD
- [ ] 086. Logging in as a domain user versus a local user (`.\` versus `DOMAIN\`)
- [ ] 087. Verifying the join from the client (Primary DNS Suffix, `whoami`, `systeminfo`)
- [ ] 088. Verifying the join from the server (ADUC, Computers container)
- [ ] 089. Diagnosing a failed join: DNS, clock skew, credentials, duplicate name
- [ ] 090. Recognising a broken trust relationship and knowing how to repair it
- [ ] 091. Moving a computer object into a Workstations OU, and explaining why the default container can't hold a GPO link

## Phase 5 — OUs, users and groups

- [ ] 092. Designing an OU structure that supports policy and delegation, not just tidiness
- [ ] 093. Creating nested OUs and understanding "protect from accidental deletion"
- [ ] 094. Creating a user and knowing the difference between display name, UPN and sAMAccountName
- [ ] 095. Reciting the default domain password policy: length, complexity, history, min and max age
- [ ] 096. Explaining why password history needs minimum password age to be effective
- [ ] 097. Using "user must change password at next logon" and explaining non-repudiation
- [ ] 098. Creating a security group, and choosing type (security vs distribution)
- [ ] 099. Choosing group scope: domain local, global, universal
- [ ] 100. Applying AGDLP and explaining what it prevents
- [ ] 101. Adding and removing members, and verifying membership from both objects
- [ ] 102. Populating department, title and description, and explaining why they aren't cosmetic
- [ ] 103. Moving objects between OUs and predicting the policy consequences
- [ ] 104. Bulk-creating users with PowerShell and a CSV (`Import-Csv`, `New-ADUser`)
- [ ] 105. Querying the directory with `Get-ADUser`, `Get-ADGroupMember`, `Search-ADAccount`
- [ ] 106. Explaining what a SID is and why it makes disable different from delete
- [ ] 107. Delegating control over an OU without granting domain admin

## Phase 6 — Help desk operations

- [ ] 108. Verifying a caller's identity before any account action
- [ ] 109. Resetting a password through ADUC and through `Set-ADAccountPassword`
- [ ] 110. Configuring account lockout policy in the Default Domain Policy
- [ ] 111. Justifying a lockout threshold against denial-of-service risk
- [ ] 112. Triggering and clearing a lockout, and distinguishing locked from disabled
- [ ] 113. Finding the source of repeated lockouts (Event IDs 4740 and 4625, cached credentials)
- [ ] 114. Disabling and re-enabling accounts and verifying both from the client
- [ ] 115. Performing an offboarding sequence in the correct order, and defending that order
- [ ] 116. Performing a department transfer as two separate changes, OU and groups
- [ ] 117. Explaining privilege accumulation and why access reviews exist
- [ ] 118. Provisioning a new hire by role rather than by copying an existing account
- [ ] 119. Mapping a reported error message to a likely cause before touching anything

## Phase 7 — File services and permissions

- [ ] 120. Creating a shared folder structure on a server
- [ ] 121. Publishing an SMB share and setting share-level permissions
- [ ] 122. Removing `Everyone` and granting a group instead
- [ ] 123. Explaining share permissions versus NTFS permissions, and which one wins
- [ ] 124. Reading an NTFS ACL and identifying inherited versus explicit entries
- [ ] 125. Disabling inheritance and converting inherited permissions to explicit
- [ ] 126. Choosing Modify over Full Control, and knowing why SYSTEM and Administrators stay
- [ ] 127. Connecting to a share with `net use` and testing both read and write
- [ ] 128. Running a negative test with an unauthorised account
- [ ] 129. Distinguishing System error 53 from System error 5 and what each rules out
- [ ] 130. Enumerating shares with `net share` on the server and `net view` from the client
- [ ] 131. Using the Effective Access tab to troubleshoot a permissions complaint

## Phase 8 — Group Policy

- [ ] 132. Explaining what Group Policy solves and how a client actually receives it
- [ ] 133. Distinguishing Policies from Preferences
- [ ] 134. Creating a GPO in GPMC and linking it to an OU
- [ ] 135. Explaining processing order: local, site, domain, OU, and link order within a container
- [ ] 136. Using Block Inheritance and Enforced, and predicting the result
- [ ] 137. Distinguishing User Configuration from Computer Configuration
- [ ] 138. Configuring a Drive Maps preference with a valid UNC path
- [ ] 139. Explaining "run in logged-on user's security context" and what breaks without it
- [ ] 140. Knowing that drive maps apply at logon, not on a policy refresh
- [ ] 141. Forcing a refresh with `gpupdate /force` and knowing when a logoff is still needed
- [ ] 142. Reading `gpresult /r` and `gpresult /h` to confirm which GPOs applied
- [ ] 143. Diagnosing a GPO that applies but produces no effect
- [ ] 144. Using security filtering to scope a GPO beyond OU boundaries
- [ ] 145. Knowing that domain password and lockout policy must be linked at the domain root

## Phase 9 — Documentation and packaging

- [ ] 146. Taking clean, legible screenshots that show the evidence and not the whole desktop
- [ ] 147. Naming screenshot files so they're identifiable without opening them
- [ ] 148. Writing a ticket in a consistent structure: request, actions, verification, resolution, prevention
- [ ] 149. Writing up a failure honestly, including the root cause
- [ ] 150. Building a network diagram in draw.io or Excalidraw
- [ ] 151. Writing a README with architecture, service inventory and a change log
- [ ] 152. Writing markdown: headings, tables, code blocks, image links, task lists
- [ ] 153. Initialising or cloning a git repository and understanding the working folder
- [ ] 154. Staging, committing and pushing, and explaining why a file isn't on GitHub yet
- [ ] 155. Writing a `.gitignore` and keeping ISOs and VM disks out of a repository
- [ ] 156. Setting repository description and topics
- [ ] 157. Writing a resume bullet that states what was built and what was verified
- [ ] 158. Explaining the whole project out loud in 45 seconds
- [ ] 159. Answering "what went wrong and how did you fix it" without rehearsing

---

## Scoring

| Marked | Meaning |
|---|---|
| Mostly `[x]` | You can defend this lab in an interview |
| Many `[~]` | You built it but followed instructions; expect follow-up questions to expose that |
| Several `[ ]` | Rebuild that phase from your own notes before claiming it |

The honest test isn't whether the lab works. It's whether you could rebuild it on a
different machine, alone, with only Microsoft's documentation, and explain each
decision while you did it.
