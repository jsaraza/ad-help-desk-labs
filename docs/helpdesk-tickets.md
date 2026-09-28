# Help Desk Ticket Log — adlab.local

Lab environment: Windows Server 2022 domain controller (AD-DC01), Windows 10 Pro
client (AD-CL01), domain `adlab.local`.

Each ticket records the request, the actions taken, and — most importantly — how the
result was verified from the end user's side rather than from the admin console.
Scenarios are fictional; the technical work, the failures and the fixes are real.

---

## TICKET-001 — Password Reset Request

**Priority:** Medium
**Category:** Account Management / Authentication
**Reported by:** John Smith (jsmith), IT Department
**Date:** 2026-08-15

**Request**
User reported being unable to log in after returning from vacation. Stated he could
not recall his current password. No lockout present on the account.

**Verification performed**
Confirmed caller identity before reset (employee ID + department match against the
directory record).

**Actions taken**
1. Located user object `jsmith` in `Departments > IT` OU (ADUC)
2. Right-click > Reset Password
3. Set a temporary password
4. Enabled "User must change password at next logon"

**Verification**
Logged into AD-CL01 as `adlab\jsmith` using the temporary password. The system
immediately prompted for a password change, confirming the reset applied and the
change-at-logon flag was enforced. The user set a new password and reached the
desktop.

![Forced password change at next logon](../screenshots/ticket-001-forced-password-change.png)

**Issue encountered during resolution**
The attempt to set a new password was rejected with "does not meet the length,
complexity, or history requirements."

**Diagnosis**
The proposed password matched a value previously used on the account. The Default
Domain Policy enforces a password history of 24, preventing reuse of recent
passwords.

**Resolution**
Advised the user to select a password not previously used. The new password was
accepted and login succeeded.

**Notes / prevention**
A temporary password with forced change at next logon means the administrator never
knows the user's working credential, which preserves non-repudiation. Caller identity
was verified before the reset; unverified reset requests are one of the most common
social engineering routes into a help desk.

Password history (24) works together with minimum password age (1 day). History alone
would let a user cycle through 24 changes in minutes and return to their preferred
password; the minimum age is what makes that impractical.

---

## TICKET-002 — Account Lockout

**Priority:** High
**Category:** Account Management / Authentication
**Reported by:** Michael Lee (mlee), HR Department
**Date:** 2026-08-15

**Request**
User reported inability to log in, receiving a message that the account was locked.
Stated he had mistyped his password several times after returning from leave.

**Diagnosis**
Account `mlee` showed a locked status in ADUC. The lockout was triggered by exceeding
the domain lockout threshold (3 invalid attempts within a 30-minute window) defined in
the Default Domain Policy.

**Verification performed**
Confirmed caller identity before unlocking (employee ID + manager name verified
against the directory record).

**Actions taken**
1. Located `mlee` in `Departments > HR` OU (ADUC)
2. Opened Properties > Account tab
3. Selected "Unlock account"
4. Confirmed no password reset was required — the user knew their credentials

**Verification**
User authenticated successfully to AD-CL01 with their existing password.

**Notes / prevention**
This lockout was legitimate user error, not a false lockout. Repeated lockouts on a
single account warrant investigation of cached credentials on secondary devices,
mapped drives with saved credentials, or services running under the user's context —
all of these retry automatically and re-lock the account within minutes.

Lockouts across multiple accounts in a short window may indicate password spraying and
should be escalated for review of the security event log (Event ID 4740 for lockouts,
4625 for failed logons).

---

## TICKET-003 — Temporary Account Suspension (Extended Leave)

**Priority:** Medium
**Category:** Account Management / Access Control
**Reported by:** HR Department, on behalf of Thu Nguyen (tnguyen)
**Date:** 2026-08-15

**Request**
HR requested temporary suspension of domain access for an employee commencing
extended leave. The account was to be retained and re-enabled on return.

**Actions taken**
1. Located user object `tnguyen` in `Departments > HR` OU (ADUC)
2. Right-click > Disable Account
3. Group memberships left intact — access to be restored on return
4. Added description: `Disabled 2026-08-15 - extended leave - ticket #1003`

**Verification**
Attempted login as `adlab\tnguyen` on AD-CL01 with the correct password.
Authentication was rejected with "Your account has been disabled."

![Disabled account cannot log on](../screenshots/ticket-003-disabled-account-logon-blocked.png)

**Notes / prevention**
Disabling rather than deleting preserves the account SID, so file permissions, mailbox
ownership and audit log attribution remain intact for the duration of the leave.
Group memberships were deliberately retained because this is a temporary suspension;
for a termination they would be stripped as well.

Note the distinction from a lockout. A disabled account is an administrative action
and produces a different error message than a lockout, which is automatic and
temporary. The wording the user reports narrows the diagnosis before anything is
opened.

---

## TICKET-004 — Account Reactivation (Return from Leave)

**Priority:** Medium
**Category:** Account Management / Access Control
**Reported by:** HR Department, on behalf of Thu Nguyen (tnguyen)
**Date:** 2026-08-29

**Request**
HR confirmed the employee had returned from extended leave and requested restoration
of domain access (ref: TICKET-003).

**Actions taken**
1. Located `tnguyen` in `Departments > HR` OU (ADUC)
2. Right-click > **Enable Account**
3. Confirmed group memberships intact — no re-provisioning required
4. Updated the description to reflect the reactivation date

**Verification**
User authenticated successfully to AD-CL01 and reached the desktop. Access to HR
resources confirmed restored.

**Notes**
Because the account was disabled rather than deleted, restoration was a single action
with no re-provisioning of group memberships, file permissions or profile data. This
is the operational argument for disable-over-delete: reversal costs one click instead
of a rebuild.

---

## TICKET-005 — Employee Termination: Access Revocation

**Priority:** High
**Category:** Account Management / Offboarding
**Reported by:** HR Department
**Date:** 2026-08-29

**Request**
HR notified IT that Sarah Chen (schen, Sales) was terminated effective immediately and
requested full revocation of domain access.

**Actions taken, in sequence**
1. Disabled account `schen` — immediate revocation of authentication
2. Removed from security group `Sales-SharedDrive-Access` — revoked share access
3. Set description: `Terminated 2026-08-29 - ticket #1042 - access revoked, retained for audit`
4. Moved the user object to the `Disabled Users` OU

**Verification**
Login attempt as `adlab\schen` on AD-CL01 was rejected. Removal from
`Sales-SharedDrive-Access` confirmed by group membership review.

![Terminated user isolated in the Disabled Users OU](../screenshots/ticket-005-terminated-user-in-disabled-ou.png)

**Notes / prevention**
The sequence is deliberate. Disabling first stops access in a single action; in a
termination for cause, every additional minute of access is risk. Group removal
follows as defence in depth — if the account is later re-enabled in error, the prior
access does not return with it, and group membership lists stay accurate for access
reviews.

The account was retained rather than deleted. The SID is referenced by file
permissions, mailbox ownership and audit log entries; deleting the account orphans
those references, and recreating an account with the same name generates a new SID
that restores nothing. A 90-day retention period before deletion review is
appropriate, subject to legal hold.

**Out of scope for this lab, but part of production offboarding**
Disable VPN and MFA tokens, revoke SSO sessions, convert the mailbox to shared,
reassign file ownership, collect hardware, and notify the manager for data handover.

---

## TICKET-006 — Internal Transfer: Sales to HR

**Priority:** Medium
**Category:** Account Management / Access Control
**Reported by:** HR Department, on behalf of Riya Patel (rpatel)
**Date:** 2026-08-29

**Request**
HR confirmed a transfer from Sales to HR effective 2026-09-01 and requested an update
to access rights and the directory record.

**Actions taken**
1. Moved the user object from `Departments > Sales` to `Departments > HR`
2. Removed from `Sales-SharedDrive-Access`
3. Added to `HR-SharedDrive-Access`
4. Updated the Department attribute on the Organization tab

**Verification**
Confirmed `rpatel` appears in the HR OU and in the members list of
`HR-SharedDrive-Access`, and no longer appears in `Sales-SharedDrive-Access`.

**Notes / prevention**
A transfer requires two independent changes: OU placement, which governs Group Policy
and delegation, and group membership, which governs resource access. Moving the OU
alone leaves the user holding their previous department's access indefinitely.

Failing to revoke old access is one of the most common findings in access reviews.
Across multiple transfers it produces privilege accumulation: long-tenured employees
who have collected access to every department they have passed through. Those accounts
are attractive targets, because compromising one yields broad access without
triggering any privilege escalation alert.

Mitigation in production: periodic access reviews, and automated group assignment
driven by the HR system rather than manual ticket-by-ticket updates.

---

## TICKET-007 — New Hire Account Provisioning

**Priority:** Medium
**Category:** Account Management / Onboarding
**Reported by:** IT Manager
**Date:** 2026-09-01

**Request**
New hire Ben Wilson starting 2026-09-01 as IT Support Technician. Domain account
required with standard IT department access.

**Actions taken**
1. Created user object `bwilson` in `Departments > IT` OU
2. Set a temporary password with "must change at next logon" enabled
3. Added to `IT-Admins-Test` per role requirements
4. Populated Department, Job Title and Description attributes

**Verification**
Logged into AD-CL01 as `adlab\bwilson`. The system prompted for a password change on
first logon as configured, the user reached the desktop, and group membership was
confirmed in ADUC.

**Notes / prevention**
Access was granted by role rather than by copying an existing user's account. Copying
is faster but inherits the source account's accumulated permissions, which propagates
privilege creep into every new hire and quietly violates least privilege.

Directory attributes were populated at creation. They are not cosmetic: they drive
automated group assignment in mature environments, let auditors map access to role,
and allow the next administrator to identify the account without context.

**Out of scope for this lab, but part of production onboarding**
Mailbox provisioning, MFA enrolment, VPN access, hardware assignment and imaging,
software licensing, badge access, and manager notification on completion.

---

## TICKET-008 — Post-Change Verification Sweep

**Priority:** Low
**Category:** Quality Assurance / Change Verification
**Date:** 2026-09-01

**Purpose**
Verify that all account changes from TICKET-001 through TICKET-007 produce the
expected result from the end user's perspective, not only in the administrative
console.

**Tests performed**

| Account | Expected | Result |
|---|---|---|
| jsmith | Login succeeds (post-reset) | Pass |
| mlee | Login succeeds (unlocked) | Pass |
| tnguyen | Login succeeds (re-enabled) | Pass |
| schen | Login denied (disabled) | Pass |
| rpatel | Login succeeds (transferred) | Pass |
| bwilson | Forced password change | Pass |

**Directory state confirmed**
- OU membership matches expected placement for all users
- Group membership matches role assignments
- The terminated account is isolated in the `Disabled Users` OU
- The `AD-CL01` computer object is located in the `Workstations` OU

**Resolution**
All changes verified. No discrepancies between intended and actual state.

**Notes**
Console state and user experience are not the same thing. An account can appear
correctly configured in ADUC while the user still cannot authenticate, due to
replication delay, cached credentials, or policy that has not yet applied. Verifying
from the client is what closes a ticket; verifying from the console only confirms that
the change was submitted.

---

## TICKET-009 — Department Share Access Control

**Priority:** Medium
**Category:** Access Control / File Services
**Date:** 2026-09-05

**Request**
The Sales department required a network share accessible only to Sales staff.

**Actions taken**
1. Created `C:\Shares\SalesShare` on AD-DC01
2. Published it as SMB share `SalesShare`; removed the default `Everyone` entry and
   granted Full Control to `Sales-SharedDrive-Access`
3. Disabled NTFS inheritance (converting inherited entries to explicit), removed
   `Users`, granted Modify to `Sales-SharedDrive-Access`, and retained `SYSTEM` and
   `Administrators`

**Verification — positive test**
`agarcia`, a member of `Sales-SharedDrive-Access`, connected successfully and created
a file, confirming Modify rights.

![Authorised user writing to the share](../screenshots/ticket-009-salesshare-write-access-agarcia.png)

**Verification — negative test**
`mlee` (HR, not a member) was refused with "You do not have permission to access
\\AD-DC01\SalesShare."

![Access denied for a non-member](../screenshots/ticket-009-salesshare-access-denied-mlee.png)

**Issue encountered during setup**
Initial access attempts returned System error 53 (network name not found).
`ping AD-DC01` succeeded, so connectivity and name resolution were intact.
`net view \\AD-DC01` listed a share named `Shares` and no `SalesShare` — the parent
directory had been published instead of the intended subdirectory. The parent share
was removed and the correct folder published.

**Notes**
Error 53 and error 5 separate two different fault domains. Error 53 (path not found)
points at share existence, name resolution or SMB availability; error 5 (access
denied) confirms the path resolved and the failure is permissions. Checking which one
appears narrows troubleshooting before any ACL is touched.

Share permissions were set permissively and access was controlled at the NTFS layer.
This is the standard pattern, because the most restrictive of the two applies, and
maintaining two overlapping restriction sets produces access nobody can explain six
months later.

---

## TICKET-010 — Automated Drive Mapping via Group Policy

**Priority:** Medium
**Category:** Group Policy / Access Provisioning
**Date:** 2026-09-19

**Request**
Map the Sales share as drive S: automatically for Sales staff at logon, without
per-machine configuration.

**Actions taken**
1. Created GPO "Map Sales Drive" and linked it to the `Departments > Sales` OU
2. Configured User Configuration > Preferences > Windows Settings > Drive Maps:
   `\\AD-DC01\SalesShare` → S:, action Update, Reconnect enabled
3. Enabled "Run in logged-on user's security context"

**Issue encountered**
The GPO applied — confirmed via `gpresult /r` — but the S: drive never appeared.

**Diagnosis**
Because `gpresult` showed *Map Sales Drive* under Applied Group Policy Objects, OU
scoping, linking and policy delivery were all ruled out, isolating the fault to the
mapping definition itself. The Location field held a malformed path rather than a
valid UNC path.

**Resolution**
Corrected the path to `\\AD-DC01\SalesShare`, ran `gpupdate /force`, and logged off
and back on. Drive maps apply at logon, not on a policy refresh.

**Verification**
Positive — `agarcia` (Sales OU) received S: automatically at logon.
Negative — `mlee` (HR OU) received no S: drive, confirming the GPO is scoped to the
Sales OU only.

**Notes**
A drive map preference requires a valid UNC path, and a malformed one fails silently,
because there is no interactive session at logon to display an error.
"Run in logged-on user's security context" matters: without it the connection is made
under the computer account, NTFS evaluates the wrong identity, and the drive fails to
appear even though every permission is correct.
