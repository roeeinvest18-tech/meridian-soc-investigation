---
title: Meridian SOC Investigation Report
---

SECURITY INCIDENT INVESTIGATION REPORT

INTERNAL — INVESTIGATION

| Organisation | Report date | Classification |
| --- | --- | --- |
| Meridian Energy Ltd | 03/09/2026 | Internal — Investigation |



| Role | Name | Sections owned |
| --- | --- | --- |
| Case Lead | Roee | 1, 2, 4, 10 |
| Identity & Access |  | 3, 5a |
| Data & Impact | shahar | 5b, 7 |
| Intel & Context | Roee,shahar | 5c, 8, 9 |



All timestamps in this report are UTC. The winevent source reports in UTC+3 and was converted manually; every converted row is marked (+3 translated).

# 1 · Executive Summary

On the night of 13 August 2026, an attacker signed in to our remote-access service using the account of an employee who left the company in June 2025. The account had never been disabled and was not protected by multi-factor authentication. Over the following eight hours the attacker moved from that first foothold to our file server, our billing server and our backup server.

Three things happened, in this order. Roughly 3.2 gigabytes of business files were opened on the file server. About 1.3 gigabytes were then packaged and uploaded to an external cloud location through a sharing link the attacker created, set so that anyone holding the link can open it. That link has no expiry date. Finally the attacker deleted a backup job and scrambled 9,412 files on the billing server, leaving a ransom note behind.

The business impact is that the billing system and the customer portal have been unavailable to customers since early on 14 August. Fuel movement at the terminals was not affected. We have found no evidence that the attacker reached the operational systems at the terminals, but we also have no monitoring data covering those systems, so we cannot state that as a fact.

The incident is not contained. We have no evidence the attacker's access has been removed, the external sharing link is still open as far as we can tell, and a second possible way in was created during the attack and never accounted for. We also cannot confirm whether a usable backup survived. Our first three requests are the immediate ones listed in section 10.

# 2 · Incident Classification & Severity

Category: ransomware with data exfiltration, commonly described as double extortion. Both elements are present in the evidence: files were staged and uploaded to external storage before encryption began, and a ransom note was written after it.

Severity: CRITICAL. The justification rests on three factors, not on the label.

| Factor | Assessment |
| --- | --- |
| Asset criticality | Four of the organisation's five crown-jewel assets were touched: SRV-FS-01, SRV-BILLING, SRV-BKP-01 and SRV-DC-01. Billing and the customer portal are core business systems. |
| Scope | Two accounts abused, four servers and one workstation involved, 9,412 files encrypted, 840 files read totalling 3.20 GB, 1.32 GB uploaded externally. |
| Impact status | Encryption and the ransom note are CONFIRMED. Exfiltration is CONFIRMED as an upload out of the environment. Recoverability is UNKNOWN, and that unknown is what holds the severity at critical rather than high. |



Severity changed once during the investigation. It was opened as HIGH on the basis of the encryption alone. It was raised to CRITICAL when the backup deletion record was read and found to carry a file count of 0 and a size of 0 GB, which means the outcome of that deletion cannot be established from the data we hold. A ransomware event with a verified backup is a recovery exercise. The same event without one is a business continuity event.

# 3 · Entities

| Entity | Type | Role | Confidence |
| --- | --- | --- | --- |
| d.aviram | Account | חשבון terminated מיוני 2025, ללא MFA, שימש לכניסה, למיפוי, ל-Kerberoasting ולמחיקת הגיבוי | Confirmed |
| y.shaked | Account | חשבון פעיל עם MFA, שימש לתנועה רוחבית, לאיסוף, להעלאה לענן ולהצפנה | Confirmed |
| 194.63.180.117 | IP | מקור ההתחברות ל-VPN, רומניה | Confirmed |
| 194.63.180.204 | IP | מקור ההעלאה ל-upload.aspx, אותו /24 | Suspected |
| WKS-OPS-03 | Host | עמדת הכניסה, ממנה יצאו כל התנועות הרוחביות | Confirmed |
| SRV-FS-01 | Host | crown-jewel, מקור 840 הקבצים שנקראו | Confirmed |
| SRV-BILLING | Host | crown-jewel, השרת שהוצפן | Confirmed |
| SRV-BKP-01 | Host | crown-jewel, השרת שממנו נמחקה עבודת הגיבוי | Confirmed |
| SRV-DC-01 | Host | crown-jewel, יעד בקשות ה-Kerberos | Confirmed |
| dmp64.exe | File | כלי גניבת אישורים שזוהה בידי Defender | Confirmed |
| 212.143.200.14 | IP | נפסל. זו כתובת היציאה של מרידיאן עצמה. מופיעה בכל 702 רשומות הענן לאורך 90 יום ובבדיקת ה-curl השגרתית | Ruled out |
| svc_backup | Account | נפסל. מבצע מחיקת הגיבוי הוא d.aviram, לא חשבון השירות | Ruled out |
| svc_scada | Account | נפסל. קריאות ה-Historian שלו רצות באותה תדירות מדי לילה גם לפני האירוע וגם אחריו | Ruled out |
| p.weiss | Account | נפסל. ללא MFA אך אינו terminated, ואין פעילות חריגה בחלון | Ruled out |
| m.aharon | Account | נפסל. terminated וללא MFA, אך אינו מופיע כלל בחלון האירוע | Ruled out |
| upgradeagent.exe | Process | נפסל. 48 הרצות ב-SYSTEM בין 01:30 ל-01:42, פריסת עדכון מתוזמנת, לא קשור לתקיפה | Ruled out |
| f.nagar מקפריסין | Account | נפסל. שבע התחברויות VPN מקפריסין ביוני, כולן עם totp, נסיעה לגיטימית | Ruled out |



# 4 · Timeline

All times UTC. Rows drawn from winevent were converted from UTC+3 and are marked (+3 translated). Ordering is by real time, not by the string in the log.

| Time (UTC) | Source | Host / Actor | What happened | Ref |
| --- | --- | --- | --- | --- |
| 13/08 22:41 | vpn | d.aviram from 194.63.180.117 (RO) | VPN session established. Result: success. MFA: none. Session length 214 minutes. Account terminated 19/06/2025. | E-01 |
| 13/08 22:52 | winevent (+3) | WKS-OPS-03 | EventCode 4624, logon type 10 (RemoteInteractive) from the VPN pool address 10.99.7.44. | E-02 |
| 13/08 22:58 – 23:00 | winevent (+3) | WKS-OPS-03 / d.aviram | Four 4688 process creations: whoami /groups; net group "Domain Admins" /domain; nltest /dclist:meridian.local; net view /domain. | E-03 |
| 13/08 23:30 | weblog | WEB-PORTAL from 194.63.180.204 | POST to /portal/upload.aspx, status 200, 341 bytes. First and only request to this path in 90 days. | E-04 |
| 14/08 00:14 – 00:18 | winevent (+3) | SRV-DC-01 / d.aviram | 63 x EventCode 4769 against 21 distinct service principals in three identical rounds, 4 to 5 seconds apart. Kerberoast. | E-05 |
| 14/08 00:52 | winevent (+3) | WKS-OPS-03 / d.aviram | EventCode 1116. Defender detection HackTool:Win32/CredDump.A on C:\Users\Public\dmp64.exe. | E-06 |
| 14/08 02:20 | winevent (+3) | SRV-FS-01 / y.shaked | EventCode 4624, logon type 10, source WKS-OPS-03. First appearance of the second account. | E-07 |
| 14/08 03:05 – 03:54 | fileaudit | SRV-FS-01 / y.shaked | 840 read operations totalling 3,199,999,590 bytes (3.20 GB). | E-08 |
| 14/08 04:10 | winevent (+3) | SRV-FS-01 / y.shaked | EventCode 4688, process 7z.exe. Command line not recorded: file servers log process name only. | E-09 |
| 14/08 04:48 | cloudaudit | y.shaked | share_link_create on /drive/Shared/ops-transfer/. Scope anyone_with_link, expiry none. | E-10 |
| 14/08 04:52 – 05:07 | cloudaudit | y.shaked | Four uploads, arch_0814_part1.7z to part4.7z, totalling 1,322,350,361 bytes (1.32 GB). | E-11 |
| 14/08 05:24 | winevent (+3) | SRV-BKP-01 / d.aviram | EventCode 4624, logon type 3, source WKS-OPS-03. | E-12 |
| 14/08 05:30 | backup | SRV-BKP-01 | NIGHTLY-FULL job status DELETED. Actor d.aviram, not the svc_backup service account. files_count 0, size_gb 0.0. | E-13 |
| 14/08 05:58 | winevent (+3) | SRV-BILLING / y.shaked | EventCode 4624, logon type 3, source WKS-OPS-03. | E-14 |
| 14/08 06:12 – 06:38 | fileaudit | SRV-BILLING / y.shaked | 9,412 rename operations, all appending the double extension .mrdn. All on SRV-BILLING; no other server affected. | E-15 |
| 14/08 06:40 | fileaudit | SRV-BILLING / y.shaked | Write of HOW_TO_RESTORE.txt, 1,834 bytes, at the root of the Billing share. | E-16 |
| 14/08 07:04 | weblog | WEB-PORTAL | First 503 returned to an external client. 503s continue to the end of the dataset on 15/08 11:56. | E-17 |
| 14/08 10:00 | weblog | WEB-PORTAL | Scheduled availability check (curl, every six hours from the corporate egress address) returns 503. The 04:00 check the same morning had returned 200. | E-18 |



Two intervals carry the case. Between the Defender alert at 00:52 and the first rename at 06:12 there are 5 hours and 20 minutes in which containment was still possible and no action was taken. Between the first file read at 03:05 and the last upload at 05:07 there are 2 hours and 2 minutes in which the data left the environment.







# 5 · Investigation — What We Checked

## 5a · Identity & Access

[Identity & Access] Owns this block. Include the checks that returned nothing — the search for 4625 events on d.aviram across all 90 days returned zero rows, and that null result is what makes section 6 item 2 defensible.

## The first known intrusion point is a VPN login at **22:41 UTC, 13/08** using the `d.aviram` account — terminated since June 2025, `mfa=none`. How the credentials were obtained is **not established** from available telemetry.

No Available logs for 4625 - only immediate 4624 success.



## 5b · Data & Impact

[Data & Impact] Owns this block. Source the numbers from fileaudit, cloudaudit and backup directly and state the query for each, so section 7 can cite them

## ## From `fileaudit`

## | Metric | Value | Filter | Window (UTC) |

## | Files read | 840 files, 3.20GB | `host=SRV-FS-01, actor=y.shaked, action=read` | 03:05–03:54 |

## | Files renamed | 9,412 | `host=SRV-BILLING, action=rename, new_ext=mrdn.` | 06:12–06:38 |

## | Ransom note write | `HOW_TO_RESTORE.txt`, 1.8KB | `host=SRV-BILLING, action=write, filename=HOW_TO_RESTORE.txt` | 06:40 |



## From `cloudaudit`

| Share link created | folder `ops-transfer`, scope `anyone_with_link`, no expiry | `actor=y.shaked, action=share_link_create, folder=ops-transfer` | 04:48 |

| Archives uploaded | 4 files (`arch_0814_part1`–`part4`), 1.32GB total | `actor=y.shaked, action=upload, filename LIKE arch_0814_part%` | 04:52–05:07 |

| Baseline for the share anomaly | 61 `share_link_create` events in 90 days; only this one pairs `anyone_with_link` + empty expiry vs. 60 `internal` + expiring | `action=share_link_create`, 90-day window | — |



## 5c## From `backup`

## | Deletion log entry | `NIGHTLY-FULL`, status `DELETED`, actor `d.aviram`, 0 files / 0GB | `job=NIGHTLY-FULL, host=SRV-BKP-01` | 05:30 |

## | Bounding successful runs | Same-night 01:35 run succeeded; 15/08 01:36 run also succeeded | same filter, adjacent nights | 01:35 (13→14/08), 01:36 (15/08) |

## **Caveat (must ship with the backup row):** this is a deletion *log entry* reporting 0/0GB — it cannot be cited as proof the backup was actually destroyed, nor that it survives. State it as "logged as deleted, unconfirmed," never "backup deleted." · Intel & Context

| Question | Query / method | Result |
| --- | --- | --- |
| Did any outbound traffic from the compromised workstation reach a command-and-control host? | proxy sourcetype, filtered to src_host=WKS-OPS-03 across the full 90 days; then the full proxy set sorted by time to establish coverage. | Zero rows for WKS-OPS-03 in 90 days. The proxy log itself ends at 13/08 13:26, roughly eight hours before the intrusion began. No outbound visibility exists for any part of this incident. Carried to section 6. |
| Is 212.143.200.14 an attacker-controlled address, as the first-pass notes assumed? | cloudaudit grouped by src_ip; then the same address checked in weblog. | It is the organisation's own egress address. All 702 cloud records across 90 days carry it, including unrelated user activity on 18/05, and it is the source of the scheduled six-hourly availability check. Ruled out as an attacker indicator. This corrected an earlier working assumption. |
| Was anything uploaded to the public web servers? | weblog filtered to method=POST and to URI paths outside the known portal page set. | One result: POST /portal/upload.aspx at 13/08 23:30 from 194.63.180.204, status 200, 341 bytes. The path appears nowhere else in 4,862 requests. with subsequent GET, session 1-11 to it. Consistent with a web shell drop that was either unused or used outside our logging. |
| Is the portal outage a symptom of the attack or an unrelated fault? | weblog status distribution before and after the incident window; scheduled check results isolated. | 503s begin at 14/08 07:04 and are continuous to the end of the data. The scheduled check returned 200 at 04:00 on 14/08 and 503 at 10:00 the same day. Before 14/08 the error profile is ordinary 404s, 403s and occasional 500s. The outage begins after encryption, not before it. |
| Do the two external addresses belong to one operator? | Comparison of 194.63.180.117 (vpn) and 194.63.180.204 (weblog) and of their timing. | Same /24, 49 minutes apart, both first-seen in 90 days. Suggestive but not conclusive. Attribution to a single operator is stated as Suspected, not Confirmed. |
| Does the SCADA historian collector show unusual activity? | fileaudit filtered to actor=svc_scada, compared across the nights of 13, 14 and 15 August. | Between 40 and 61 reads per half hour in the 01:00 to 03:00 window on all three nights, including the night after the attack. Consistent baseline behaviour. Ruled out. |

# 6 · What Could Not Be Determined

Reasons are limited to the four permitted values. Each row states what the reader cannot rely on.

| Item | Reason | What this means for the reader |
| --- | --- | --- |
| Whether a backup copy was actually destroyed | No data source | The deletion record reports 0 files and 0.0 GB, so it cannot be read either way. Do not plan recovery on the assumption that a restore point exists, and do not declare it lost. The scheduled job at 01:35 on 14/08 succeeded and the one at 01:36 on 15/08 succeeded; whether either survived the 05:30 deletion needs to be established on the backup appliance itself. |
| How the credentials for d.aviram were obtained | No data source | There is not a single 4625 failed-logon event for this account in 90 days. Credential theft outside our environment cannot be ruled out, which means rotating this one account may not close the original route in. |
| Whether the web shell at /portal/upload.aspx was ever executed | No data source | One POST with status 200 and no follow-up request. A second, independent access path may still be open on the public web server. Treat WEB-PORTAL as potentially compromised until it is examined on the host. |
| Which specific files left the organisation | No data source | The cloud log records four archive names and their sizes, not their contents. Notification decisions cannot be based on a file list. The 840 files read on SRV-FS-01 are the best available proxy and are an inference, not a record. |
| What outbound traffic occurred during the intrusion | No data source | The proxy log ends eight hours before the first login and WKS-OPS-03 never appears in it at all. No command-and-control channel, no second exfiltration path and no beaconing pattern can be confirmed or excluded. |
| What commands ran on the file and billing servers | No data source | Command-line logging is enabled on workstations only. The 7z.exe execution at 04:10 arrives with no arguments, so what was compressed, from where, and to where is not recorded. |
| The true time of the night shift escalation | No data source | The handover brief states 07:20 without naming a time zone. Depending on the reading, the gap between the first customer-visible failure and escalation is either sixteen minutes or three hours and sixteen minutes. Any response-time metric built on this figure is unsafe. |

# 7 · Impact Assessment

[Data & Impact] Owns this section. Keep CONFIRMED and POTENTIAL in separate paragraphs — mixing them fails the section. Confirmed: 9,412 files encrypted on SRV-BILLING, 840 files / 3.20 GB read on SRV-FS-01, 1.32 GB uploaded in four archives, four crown-jewel assets touched, two accounts abused. Potential: contents of the archives, recoverability, and anything on the OT network.

# **Confirmed.** On `SRV-BILLING`, 9,412 files were encrypted — renamed to the double extension `mrdn.` between 06:12 and 06:38 UTC. On `SRV-FS-01`, 840 files totaling 3.20GB were read by `y.shaked` between 03:05 and 03:54 UTC. 1.32GB was uploaded in four archives (`arch_0814_part1` through `part4`) between 04:52 and 05:07 UTC, via a cloud share set to `anyone_with_link` with no expiry. Four crown-jewel assets were touched: `SRV-FS-01`, `SRV-BILLING`, `SRV-BKP-01`, and `SRV-DC-01`. Two accounts were abused: `d.aviram` and `y.shaked`



**Potential.** The contents of the four uploaded archives are not established — only that 840 files were read beforehand exists as a data point, and mapping the two is an estimate, not a determination. Backup recoverability is unresolved: the `NIGHTLY-FULL` deletion log entry on `SRV-BKP-01` (05:30 UTC) reports 0 files and 0GB, which cannot confirm that anything was actually destroyed or that a restore point survives. Nothing on the OT network can be confirmed or ruled out: `svc_scada`'s `Historian` call pattern is unchanged before and after the window, but no outbound-traffic visibility exists for the incident period at all, so that absence of evidence is not evidence of a clean network, nor of denial of service.





# 8 · MITRE ATT&CK Mapping & Detection Gaps

| Stage | Tactic | Technique | Detected? | By what (Baseline - No SIEM) |
| --- | --- | --- | --- | --- |
| VPN login with a terminated account | Initial Access | T1078.002 Valid Accounts: Domain Accounts | Yes | Nothing. No rule on terminated-account authentication and no rule on MFA-less VPN success. |
| Web shell upload to the public portal | Persistence | T1505.003 Server Software Component: Web Shell | Yes | Nothing. No alerting on writes to webfacing paths. |
| Domain, group and DC enumeration | Discovery | T1087.002, T1018, T1482 | No | Nothing, despite full command-line logging being enabled on the workstation. |
| Kerberoasting | Credential Access | T1558.003 Steal or Forge Kerberos Tickets | No | Nothing. 63 ticket requests to 21 services in four minutes produced no alert. |
| Credential dumping tool executed | Credential Access | T1003 OS Credential Dumping | Yes | Defender, EventCode 1116, at 00:52. The single alert of the intrusion. No response followed. |
| RDP to the file server | Lateral Movement | T1021.001 Remote Services: RDP | No | Nothing. Server-to-server type 10 logon at 02:20 is not baselined. |
| Bulk read of file shares | Collection | T1039 Data from Network Shared Drive | No | Nothing. 840 reads and 3.20 GB in 49 minutes from one account passed without threshold alerting. |
| Archiving with 7-Zip | Collection | T1560.001 Archive via Utility | No | Nothing, and command-line logging is absent on servers so even retrospective analysis is blind. |
| Upload to cloud storage via an open link | Exfiltration | T1567.002 Exfiltration to Cloud Storage | No | Nothing. A share scoped anyone_with_link with no expiry is the highest-signal single event in the dataset. |
| Backup job deletion | Impact | T1490 Inhibit System Recovery | No | Nothing. A backup deleted by a user account rather than svc_backup is trivially detectable and was not detected. |
| Mass encryption and ransom note | Impact | T1486 Data Encrypted for Impact | No | Nothing. 9,412 renames in 26 minutes on a crown-jewel asset produced no alert; the outage was discovered through customer-facing 503s. |



The finding is not the technique list. Eleven stages of this attack were logged and one produced an alert. Three of the missing detections need no new tooling and no new data source: terminated-account authentication, a backup job deleted by a non-service account, and a cloud share created with anyone_with_link and no expiry. Each of those is a single-field condition on data we already collect, and each sits at a point where the attack could still have been stopped. It is worth mentioning, that if we indeed had a functioning SIEM alerts would have most likely shown, and we could’ve contained the attack while it was happening.

# 9 · What This Resembles

This resembles the human-operated double-extortion ransomware pattern that has been documented repeatedly against mid-sized industrial and logistics operators, The comparison is drawn from our own evidence and stated with medium confidence.

What fits.

The published pattern begins with valid credentials on a remote-access service that lacks multi-factor authentication, rather than with an exploit or a phishing payload; our entry is a VPN login with mfa=none at 22:41. It proceeds to native discovery tooling instead of custom implants; our attacker used whoami, net and nltest, all built into the operating system. It uses Kerberoasting for lateral credentials; we logged 63 ticket requests across 21 service principals in four minutes. It stages with a common archiver and exfiltrates to commodity cloud storage before encrypting; we have 7z.exe at 04:10 and four archives uploaded between 04:52 and 05:07. It deletes backups immediately before encryption; ours were deleted at 05:30 and encryption began at 06:12. It works through a single overnight window and finishes before the business day; ours ran eight hours and nineteen minutes end to end. The sequence, the tooling and the compression of the whole operation into one night all match.

What does not fit.

Three elements are absent or unlike the pattern. First, encryption was confined to one server, SRV-BILLING; the documented pattern encrypts broadly across file servers and hypervisors, and SRV-FS-01 was read extensively but never encrypted. Second, we have no evidence of the defence-evasion stage these groups reliably perform: no attempt to disable Defender after it flagged the tool at 00:52, and no shadow-copy deletion in our data. Third, the volume is modest. 1.32 GB is small for a double-extortion set, and the ransom note at 1,834 bytes was written once, in one directory, rather than across every affected share.

Confidence and the honest alternative.

Medium confidence, and the limiting factor is named rather than glossed. We have no outbound traffic data for any part of the intrusion, so we cannot compare infrastructure, tooling downloads or C2 behaviour against public reporting, which is where a family attribution would normally be earned. An equally consistent reading of the same evidence is a less capable operator following a widely circulated playbook, or an intrusion that was interrupted before its final stage rather than one that was executed as designed. The narrow encryption footprint and the untouched Defender installation support that alternative. We are not naming a group.

# 10 · Recommendations

## Immediate — within 24 hours

| # | Recommendation | Ties to finding | Authority |
| --- | --- | --- | --- |
| 1 | Revoke the cloud sharing link on /drive/Shared/ops-transfer/ and disable the y.shaked cloud session. The link is scoped anyone_with_link with no expiry, so exposure continues for as long as it exists. | Section 4, E-10 | Tier 1 |
| 2 | Disable d.aviram and y.shaked, and terminate all active VPN sessions for both. The first is a terminated account that was never disabled; the second was used for every action after 02:20. | Section 4, E-01 and E-07 | Tier 1 |
| 3 | Take SRV-BILLING and SRV-FS-01 off the network without powering them down, and preserve WKS-OPS-03 for forensic imaging. WKS-OPS-03 is the source of every lateral movement in the timeline. | Section 4, E-02 to E-15 | Requires approval |
| 4 | Verify on the backup appliance itself whether a restore point survives, and report the answer as a fact rather than an inference. Section 6 cannot close this from logs. | Section 6, item 1 | Requires approval |
| 5 | Examine WEB-PORTAL on the host for the file dropped at /portal/upload.aspx and remove it. Until this is done the environment must be treated as having a second, unmonitored way in. | Section 5c and section 6, item 3 | Requires approval |



## Short term — within 30 days

| # | Recommendation | Ties to finding | Authority |
| --- | --- | --- | --- |
| 6 | Build a detection for authentication by any account holding a termination_date in identity.csv. This is a single-field join and would have fired at 22:41, before any other action in this incident. | Section 8, stage 1 | Tier 1 |
| 7 | Alert on any backup job whose actor is not svc_backup. The 05:30 deletion was performed by a user account and this condition is already present in the backup log. | Section 8, stage 10 | Tier 1 |
| 8 | Alert on cloud share creation where link_scope is anyone_with_link and link_expiry is empty. One event, 44 minutes before exfiltration completed. | Section 8, stage 9 | Tier 1 |
| 9 | Alert on more than 20 EventCode 4769 requests from a single account within five minutes. Our burst was 63 in four minutes. | Section 8, stage 4 | Requires approval |
| 10 | Enable command-line logging on SRV-FS-01, SRV-BILLING and SRV-BKP-01. The 7z.exe execution is currently unanalysable, and this gap will recur in the next incident. | Section 6, item 6 | Requires approval |
| 11 | Restore proxy log delivery and bring WKS-OPS-03 and its peer workstations into proxy coverage. The log stopped eight hours before the intrusion and nobody noticed. | Section 5c and section 6, item 5 | Requires approval |
| 12 | Define a mandatory response time for Defender detections of credential-theft tooling, with escalation if unacknowledged. The alert at 00:52 was the one chance this organisation had. | Section 4, E-06 | Management |



## Long term

| # | Recommendation | Ties to finding | Authority |
| --- | --- | --- | --- |
| 13 | Enforce MFA on all VPN authentication with no exceptions, and make offboarding disable the account rather than only mark it. d.aviram left in June 2025 and could still sign in fourteen months later. | Section 4, E-01 | Management |
| 14 | Reduce the crown-jewel blast radius from WKS-OPS-03. One medium-criticality workstation reached four of five crown-jewel assets in under four hours with no network control in the way. | Section 4, E-02 to E-14 | Management |
| 15 | Extend logging to the OT network at the terminals and to the five assets currently without EDR, including both HMI terminals. We were unable to state whether operations were affected, which is a reporting failure as much as a security one. | Section 6, item 5 | Management / Legal |
| 16 | Establish who determines notification obligations for exfiltrated customer and invoice data, before the archive contents are known rather than after. This is not a technical decision. | Section 6, item 4 | Management / Legal |

# Appendix A · AI Verification Log

[All roles] Submitted as a separate file as well. Every row needs How verified, Verdict and Error type. Three verified rows are required at the 31/08 gate. Two real errors from this investigation are already available to log: the assumption that 212.143.200.14 was attacker infrastructure (error type: entity), and the reading of the upload.aspx event as 11:30 rather than 23:30 (error type: time zone).

# Appendix B · Queries Used

Queries supporting sections 4, 5c, 8 and 9. Index name as loaded locally.

| # | Section | Query |
| --- | --- | --- |
| Q1 | 4 | index=meridian sourcetype=winevent \| eval _time=_time-10800 \| table _time EventCode host actor logon_type src_host process cmdline object_name \| sort _time |
| Q2 | 4 | index=meridian sourcetype=vpn result=success src_country!="IL" \| table _time user src_ip src_country mfa session_min |
| Q3 | 4 | index=meridian sourcetype=winevent EventCode=4769 actor="d.aviram" \| stats count dc(target_account) as services min(_time) max(_time) |
| Q4 | 4 | index=meridian sourcetype=fileaudit action=rename \| stats count min(_time) max(_time) dc(server) by server |
| Q5 | 4 | index=meridian sourcetype=cloudaudit operation IN (upload, share_link_create) earliest="08/14/2026:04:00:00" \| table _time actor operation target_object link_scope link_expiry bytes_out |
| Q6 | 4 | index=meridian sourcetype=backup status!=SUCCESS \| table _time job_name target status files_count size_gb actor |
| Q7 | 5c | index=meridian sourcetype=proxy src_host="WKS-OPS-03" \| stats count |
| Q8 | 5c | index=meridian sourcetype=proxy \| stats min(_time) as first max(_time) as last count |
| Q9 | 5c | index=meridian sourcetype=cloudaudit \| stats count dc(actor) as actors by src_ip |
| Q10 | 5c | index=meridian sourcetype=weblog method=POST NOT uri_path IN ("/portal/login.aspx","/portal/logout.aspx","/portal/orders.aspx","/portal/invoices.aspx","/portal/account.aspx","/portal/deliveries.aspx") \| table _time src_ip uri_path status bytes_out |
| Q11 | 5c | index=meridian sourcetype=weblog status=503 \| stats min(_time) as first max(_time) as last count |
| Q12 | 5c | index=meridian sourcetype=fileaudit actor="svc_scada" \| timechart span=30m count |
| Q13 | 8 | index=meridian sourcetype=winevent EventCode=1116 \| table _time host actor process object_name |
| Q14 | 8 | index=meridian sourcetype=fileaudit actor="y.shaked" action=read \| stats count sum(bytes_out) as total_bytes by server |
| Q15 | 8 | \| inputlookup identity.csv \| search isnotnull(termination_date) \| table user department termination_date mfa_enrolled |
| Q16 | 9 | \| inputlookup assets.csv \| search criticality="crown-jewel" OR has_edr="no" \| table hostname criticality has_edr owner |

# Appendix C · Evidence Excerpts

| Ref | Excerpt |
| --- | --- |
| E-01 | 2026-08-13 22:41:00, d.aviram, 194.63.180.117, RO, success, none, 214 |
| E-05 | 2026-08-14 03:14:09 (local), 4769, SRV-DC-01, d.aviram, WKS-OPS-03, cifs_svc, CIFS/SRV-FS-01.meridian.local  — one of 63 |
| E-06 | 2026-08-14 03:52:00 (local), 1116, WKS-OPS-03, d.aviram, C:\Users\Public\dmp64.exe, HackTool:Win32/CredDump.A |
| E-10 | 2026-08-14 04:48:00, y.shaked, share_link_create, /drive/Shared/ops-transfer/, anyone_with_link, none |
| E-13 | 2026-08-14 05:30:00, NIGHTLY-FULL, SRV-BKP-01, DELETED, 0, 0.0, d.aviram |
| E-15 | 2026-08-14 06:12:00, y.shaked, \\SRV-BILLING\Billing\2024\Q3\INV-377285.pdf.mrdn, rename, 0, WKS-OPS-03, SRV-BILLING  — one of 9,412 |
| E-16 | 2026-08-14 06:40:00, y.shaked, \\SRV-BILLING\Billing\HOW_TO_RESTORE.txt, write, 1834, WKS-OPS-03, SRV-BILLING |

