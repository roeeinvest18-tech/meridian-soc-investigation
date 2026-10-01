---
title: Meridian SOC Investigation Report
---

SECURITY INCIDENT INVESTIGATION REPORT

EDUCATIONAL CASE STUDY — SIMULATED INCIDENT

| Organisation | Report date | Classification |
| --- | --- | --- |
| Meridian Energy Ltd (fictional) | 03/09/2026 | Educational case study |


| Role | Name | Sections owned |
| --- | --- | --- |
| Case Lead | Roee Nir | 1, 2, 4, 10 |
| Identity & Access | Not recorded in published role table | 3, 5a |
| Data & Impact | Shahar | 5b, 7 |
| Intel & Context | Roee Nir, Shahar | 5c, 8, 9 |


This is a retrospective training investigation. Full source logs and execution results are not published; seven selected excerpts appear in Appendix B. Numerical results below are retained from the original investigation, not re-computed during documentation review. Proposed response actions are not reported as completed.

[Analyst handover](HANDOVER.md) · [Repository overview](../README.md)

All timestamps in this report are UTC. The winevent source reports in UTC+3 and was converted manually; every converted row is marked (+3 translated).

# 1 · Executive Summary

The reported incident begins on 13 August 2026 at 22:41 UTC with a successful VPN login by `d.aviram`, an account marked terminated since June 2025, with `mfa=none`. Subsequent records describe discovery commands, suspicious Kerberos ticket requests, a Defender credential-dumping-tool detection, and logons to sensitive servers.

The investigation reports 840 file-read operations on `SRV-FS-01` totalling 3.20 GB, followed by four cloud archive uploads totalling 1.32 GB. A sharing link was created with `anyone_with_link` scope and no expiry. A backup job was logged with status `DELETED`, zero files and zero GB; actual backup destruction and recoverability are unresolved. On `SRV-BILLING`, 9,412 rename operations added a `.mrdn` extension and a ransom-note file was written. This is consistent with ransomware activity; the supplied rename excerpts do not independently confirm encryption of file contents or unique-file counts.

The customer portal began returning HTTP 503 at 07:04 UTC on 14 August. This establishes observed service unavailability in the reported log window, not its root cause. OT impact is unknown because the available telemetry cannot establish whether terminal operations were affected.

Containment status is unknown. No published response record establishes that access was revoked, the sharing link was removed, or systems were isolated. Escalate the scenario for approved identity, endpoint, cloud, web-host, and recovery validation.

# 2 · Incident Classification & Severity

**Assessment:** suspected ransomware with observed cloud data uploads. The sequence is consistent with data theft followed by ransomware impact; archive contents, file-content encryption, and any extortion demand beyond the named ransom-note artifact are not independently validated here.

**Training severity assessment: CRITICAL**, based on sensitive-asset involvement, observed portal unavailability, ransomware indicators, and unresolved recoverability. This is an analyst assessment, not a verified production ticket severity or an organisation-wide severity rule.

| Factor | Assessment |
| --- | --- |
| Asset criticality | The investigation identifies SRV-FS-01, SRV-BILLING, SRV-BKP-01, and SRV-DC-01 as crown-jewel assets. |
| Reported scope | Two accounts, four servers, and one workstation; 840 read operations (3.20 GB), four archive uploads (1.32 GB), and 9,412 rename operations. |
| Observed impact indicators | File-extension changes, a ransom-note write, and HTTP 503 responses. Rename operations alone do not establish unique encrypted-file counts. |
| Recovery and response | Backup survival and containment are unknown. Verify them before planning recovery or closing the incident. |

The original narrative described a HIGH-to-CRITICAL revision. Without a published case-status history, this report does not treat that sequence as an independently verified operational event.

# 3 · Entities

| Entity | Type | Role | Confidence |
| --- | --- | --- | --- |
| d.aviram | Account | חשבון terminated מיוני 2025, ללא MFA, שימש לכניסה, למיפוי, לפעילות Kerberos חשודה ולרשומת גיבוי בסטטוס DELETED | Confirmed |
| y.shaked | Account | חשבון פעיל עם MFA, שימש לתנועה רוחבית, לאיסוף, להעלאה לענן ולשינויי שמות קבצים | Confirmed |
| 194.63.180.117 | IP | מקור ההתחברות ל-VPN, רומניה | Confirmed |
| 194.63.180.204 | IP | מקור ההעלאה ל-upload.aspx, אותו /24 | Suspected |
| WKS-OPS-03 | Host | עמדת הכניסה, ממנה יצאו כל התנועות הרוחביות | Confirmed |
| SRV-FS-01 | Host | crown-jewel, מקור 840 פעולות הקריאה | Confirmed |
| SRV-BILLING | Host | crown-jewel, השרת עם שינויי סיומת ופתק כופר | Confirmed |
| SRV-BKP-01 | Host | crown-jewel, השרת עם רשומת גיבוי בסטטוס DELETED | Confirmed |
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
| 14/08 00:14 – 00:18 | winevent (+3) | SRV-DC-01 / d.aviram | 63 x EventCode 4769 against 21 distinct service principals in three identical rounds, 4 to 5 seconds apart. Pattern consistent with Kerberoasting; offline cracking is not established. | E-05 |
| 14/08 00:52 | winevent (+3) | WKS-OPS-03 / d.aviram | EventCode 1116. Defender detection HackTool:Win32/CredDump.A on C:\Users\Public\dmp64.exe. | E-06 |
| 14/08 02:20 | winevent (+3) | SRV-FS-01 / y.shaked | EventCode 4624, logon type 10, source WKS-OPS-03. First appearance of the second account. | E-07 |
| 14/08 03:05 – 03:54 | fileaudit | SRV-FS-01 / y.shaked | 840 read operations totalling 3,199,999,590 bytes (3.20 GB). | E-08 |
| 14/08 04:10 | winevent (+3) | SRV-FS-01 / y.shaked | EventCode 4688, process 7z.exe. Command line not recorded: file servers log process name only. | E-09 |
| 14/08 04:48 | cloudaudit | y.shaked | share_link_create on /drive/Shared/ops-transfer/. Scope anyone_with_link, expiry none. | E-10 |
| 14/08 04:52 – 05:07 | cloudaudit | y.shaked | Four uploads, arch_0814_part1.7z to part4.7z, totalling 1,322,350,361 bytes (1.32 GB). | E-11 |
| 14/08 05:24 | winevent (+3) | SRV-BKP-01 / d.aviram | EventCode 4624, logon type 3, source WKS-OPS-03. | E-12 |
| 14/08 05:30 | backup | SRV-BKP-01 | NIGHTLY-FULL job status DELETED. Actor d.aviram, not the svc_backup service account. files_count 0, size_gb 0.0. | E-13 |
| 14/08 05:58 | winevent (+3) | SRV-BILLING / y.shaked | EventCode 4624, logon type 3, source WKS-OPS-03. | E-14 |
| 14/08 06:12 – 06:38 | fileaudit | SRV-BILLING / y.shaked | 9,412 rename operations, all appending the double extension .mrdn. All reported matching rename operations were on SRV-BILLING; broader impact is not excluded. | E-15 |
| 14/08 06:40 | fileaudit | SRV-BILLING / y.shaked | Write of HOW_TO_RESTORE.txt, 1,834 bytes, at the root of the Billing share. | E-16 |
| 14/08 07:04 | weblog | WEB-PORTAL | First 503 returned to an external client. 503s continue to the end of the dataset on 15/08 11:56. | E-17 |
| 14/08 10:00 | weblog | WEB-PORTAL | Scheduled availability check (curl, every six hours from the corporate egress address) returns 503. The 04:00 check the same morning had returned 200. | E-18 |


The interval between the Defender detection (00:52) and the first reported rename (06:12) is 5 hours and 20 minutes. This does not establish whether containment was attempted. File reads are reported between 03:05 and 03:54, and uploads between 04:52 and 05:07; the full interval is 2 hours and 2 minutes, not the upload duration. The first HTTP 503 at 07:04 occurred 52 minutes after the first rename.


# 5 · Investigation — What We Checked

## 5a · Identity & Access

The first known intrusion point in the reported chronology is the VPN login at **22:41 UTC, 13/08** using `d.aviram`, marked terminated with `mfa=none`. The investigation reports zero matching 4625 failed-logon events for this account across 90 days. This absence does not establish how credentials were obtained or rule out activity outside available coverage.

## 5b · Data & Impact

These are reported operation counts, not independently verified counts of unique files.

| Source | Metric | Reported value | Window (UTC) | Query reference |
| --- | --- | --- | --- | --- |
| fileaudit | Read operations on SRV-FS-01 by y.shaked | 840 operations; 3,199,999,590 bytes | 03:05–03:54 | Q14, with explicit incident window |
| fileaudit | Rename operations on SRV-BILLING | 9,412 operations adding .mrdn | 06:12–06:38 | Q4, with target-path review |
| fileaudit | Ransom-note write | HOW_TO_RESTORE.txt; 1,834 bytes | 06:40 | Timeline E-16; Appendix B excerpt |
| cloudaudit | Public-link creation | ops-transfer; anyone_with_link; no expiry | 04:48 | Q5 |
| cloudaudit | Archive uploads | Four archives; 1,322,350,361 bytes | 04:52–05:07 | Q5; aggregate result not published |
| cloudaudit | Share baseline | 61 share-link events over 90 days; reported unique scope/expiry combination | 90-day baseline | Full baseline output not published |
| backup | Deletion status | NIGHTLY-FULL; DELETED; d.aviram; zero files / zero GB | 05:30 | Q6; E-13 |
| backup | Successful-job status | Successful runs at 14/08 01:35 and 15/08 01:36 | Adjacent nights | Job status does not prove surviving restore points |

The backup status cannot establish either data destruction or recoverability. Archive contents and a mapping from read operations to uploaded files are unknown.

## 5c · Intel & Context

| Question | Query / method | Result |
| --- | --- | --- |
| Did any outbound traffic from the compromised workstation reach a command-and-control host? | proxy sourcetype, filtered to src_host=WKS-OPS-03 across the full 90 days; then the full proxy set sorted by time to establish coverage. | Zero rows for WKS-OPS-03 in 90 days. The proxy log itself ends at 13/08 13:26, roughly eight hours before the intrusion began. No outbound visibility exists for any part of this incident. Carried to section 6. |
| Is 212.143.200.14 an attacker-controlled address, as the first-pass notes assumed? | cloudaudit grouped by src_ip; then the same address checked in weblog. | It is the organisation's own egress address. All 702 cloud records across 90 days carry it, including unrelated user activity on 18/05, and it is the source of the scheduled six-hourly availability check. Ruled out as an attacker indicator. This corrected an earlier working assumption. |
| Was anything uploaded to the public web servers? | weblog filtered to method=POST and to URI paths outside the known portal page set. | One result: POST /portal/upload.aspx at 13/08 23:30 from 194.63.180.204, status 200, 341 bytes. The path appears nowhere else in 4,862 requests. The original notes conflict on whether a subsequent GET occurred. The POST alone does not establish uploaded file contents, web shell deployment, or execution; verify the request sequence and host artifacts. |
| Is the portal outage a symptom of the attack or an unrelated fault? | weblog status distribution before and after the incident window; scheduled check results isolated. | 503s begin at 14/08 07:04 and are continuous to the end of the data. The scheduled check returned 200 at 04:00 on 14/08 and 503 at 10:00 the same day. Before 14/08 the error profile is ordinary 404s, 403s and occasional 500s. The outage follows the rename activity. Temporal ordering does not establish the root cause. |
| Do the two external addresses belong to one operator? | Comparison of 194.63.180.117 (vpn) and 194.63.180.204 (weblog) and of their timing. | Same /24, 49 minutes apart, both first-seen in 90 days. Suggestive but not conclusive. Attribution to a single operator is stated as Suspected, not Confirmed. |
| Does the SCADA historian collector show unusual activity? | fileaudit filtered to actor=svc_scada, compared across the nights of 13, 14 and 15 August. | Between 40 and 61 reads per half hour in the 01:00 to 03:00 window on all three nights, including the night after the attack. Consistent baseline behaviour. Ruled out. |

# 6 · What Could Not Be Determined

Each row distinguishes an evidence gap from a confirmed finding.

| Item | Reason | What this means for the reader |
| --- | --- | --- |
| Whether a backup copy was actually destroyed | No data source | The deletion record reports 0 files and 0.0 GB, so it cannot be read either way. Do not plan recovery on the assumption that a restore point exists, and do not declare it lost. The scheduled job at 01:35 on 14/08 succeeded and the one at 01:36 on 15/08 succeeded; whether either survived the 05:30 deletion needs to be established on the backup appliance itself. |
| How the credentials for d.aviram were obtained | No data source | There is not a single 4625 failed-logon event for this account in 90 days. Credential theft outside our environment cannot be ruled out, which means rotating this one account may not close the original route in. |
| Whether /portal/upload.aspx involved web shell deployment or execution | No data source | One suspicious POST is reported. The notes conflict on follow-up requests, and neither file contents nor execution artifacts are published. Investigate WEB-PORTAL without presenting a web shell as confirmed. |
| Which specific files left the organisation | No data source | The cloud log records four archive names and their sizes, not their contents. Notification decisions cannot be based on a file list. The 840 read operations on SRV-FS-01 provide context, but are not a verified unique-file list or archive-content inventory. |
| What outbound traffic occurred during the intrusion | No data source | The proxy log ends eight hours before the first login and WKS-OPS-03 never appears in it at all. No command-and-control channel, no second exfiltration path and no beaconing pattern can be confirmed or excluded. |
| What commands ran on the file and billing servers | No data source | Command-line logging is enabled on workstations only. The 7z.exe execution at 04:10 arrives with no arguments, so what was compressed, from where, and to where is not recorded. |
| The true time of the night shift escalation | No data source | The handover brief states 07:20 without naming a time zone. If 07:20 is UTC, it is sixteen minutes after the first 503 at 07:04 UTC. If it is UTC+3, it is 04:20 UTC, before that outage. Verify the source time zone and event ordering; do not derive a response-time metric. |

# 7 · Impact Assessment

**Reported observations:** 840 read operations totalling 3.20 GB on `SRV-FS-01`; four archive uploads totalling 1.32 GB; 9,412 rename operations on `SRV-BILLING`; a `HOW_TO_RESTORE.txt` write; and HTTP 503 responses from the portal. Two accounts and four crown-jewel assets are identified in the investigation.

**Assessment:** the sequence is consistent with data collection, cloud transfer, and ransomware impact. The portal outage occurs after the rename activity, but the available records do not prove a causal link.

**Unresolved:** unique affected-file counts, encryption of contents, archive contents, actual backup destruction, restore-point survival, containment, and OT impact. The unchanged `svc_scada` pattern is not evidence that the entire OT environment is unaffected.

# 8 · MITRE ATT&CK Mapping & Detection Opportunities

ATT&CK mappings describe behavior and hypotheses, not independent proof of every attack stage. A retrospective observation is distinct from a real-time alert.

| Reported behavior | Technique / hypothesis | Evidence status | Alert visibility in supplied material |
| --- | --- | --- | --- |
| Terminated-account VPN success | T1078.002 Valid Accounts: Domain Accounts | Login and account status reported | Alert generation and handling unknown |
| Suspicious POST to upload.aspx | T1505.003 Web Shell, provisional | Deployment and execution unconfirmed | Unknown |
| Domain/group discovery commands | T1087.002, T1018, T1482 | Commands reported | Unknown |
| Burst of 4769 requests | T1558.003 Kerberoasting, suspected | 63 requests to 21 targets reported; cracking unproven | Unknown |
| Defender credential-dumping-tool detection | T1003 OS Credential Dumping, suspected | Event 1116 excerpt; successful dumping unproven | Defender detection present; response unknown |
| RDP logon to file server | T1021.001 Remote Desktop Protocol | Logon type 10 from workstation reported | Unknown |
| Bulk file-share reads | T1039 Data from Network Shared Drive | Read operations reported; unique files unknown | Unknown |
| 7z.exe execution | T1560.001 Archive via Utility, suspected | Process reported; arguments and contents absent | Unknown |
| Cloud archive uploads | T1567.002 Exfiltration to Cloud Storage | Uploads reported; content inventory unknown | Unknown |
| Backup DELETED status | T1490 Inhibit System Recovery, suspected | Status excerpt; actual destruction unconfirmed | Unknown |
| Rename burst and ransom-note write | T1486 Data Encrypted for Impact, suspected | Ransomware indicators; content encryption unverified | Unknown |

Candidate detections include terminated-account authentication, backup deletion-status events by unexpected actors, and public cloud shares without expiry. These require tested field logic, account context, baselines, and approved deployment. The published material does not establish which SIEM rules existed, whether additional alerts fired, or whether operators responded.

# 9 · What This Resembles

The sequence is consistent with a human-operated intrusion involving valid-account access, discovery, credential-access indicators, lateral logons, collection, cloud uploads, and ransomware indicators.

This is a behavioral assessment of the scenario, not attribution to a threat group or proof of a specific ransomware family. Kerberos ticket requests do not establish offline cracking; the Defender detection does not establish successful credential theft; rename operations do not independently prove encrypted contents.

Proxy coverage is missing during the incident, server command-line arguments are incomplete, and archive contents are unavailable. These gaps limit conclusions about C2, secondary exfiltration paths, compression inputs, and actor intent. No external campaign comparison is claimed without a cited source.

# 10 · Recommendations

These are proposed actions for a simulated incident, not actions performed. Tier 1 may document, enrich, and escalate; account/session changes, link revocation, isolation, forensic work, and detection deployment follow approved playbooks and role-specific authorisation.

## Immediate — within 24 hours

| # | Recommendation | Ties to finding | Authority |
| --- | --- | --- | --- |
| 1 | Revoke the cloud sharing link on /drive/Shared/ops-transfer/ and disable the y.shaked cloud session. The link is scoped anyone_with_link with no expiry, so exposure continues for as long as it exists. | Section 4, E-10 | Approved playbook / authorised responder |
| 2 | Disable d.aviram and y.shaked, and terminate all active VPN sessions for both. The first is marked terminated but authenticated; the second appears in lateral-logon, read, upload, and rename activity. | Section 4, E-01 and E-07 | Approved playbook / authorised responder |
| 3 | Take SRV-BILLING and SRV-FS-01 off the network without powering them down, and preserve WKS-OPS-03 for forensic imaging. WKS-OPS-03 is the source of every lateral movement in the timeline. | Section 4, E-02 to E-15 | Requires approval |
| 4 | Verify on the backup appliance itself whether a restore point survives, and report the answer as a fact rather than an inference. Section 6 cannot close this from logs. | Section 6, item 1 | Requires approval |
| 5 | Preserve and examine WEB-PORTAL artifacts associated with /portal/upload.aspx. If malicious content is confirmed, remediate through the approved IR process; deployment and execution are not yet established. | Section 5c and section 6, item 3 | Requires approval |


## Short term — within 30 days

| # | Recommendation | Ties to finding | Authority |
| --- | --- | --- | --- |
| 6 | Build a detection for authentication by any account holding a termination_date in identity.csv. Correlate authentication with the identity lookup and validate account status, timestamps, and test results before claiming detection coverage. | Section 8, stage 1 | Approved playbook / authorised responder |
| 7 | Alert on any backup job whose actor is not svc_backup. The 05:30 DELETED-status event records a user account. Validate the expected actor list and backup-event semantics. | Section 8, stage 10 | Approved playbook / authorised responder |
| 8 | Alert on cloud share creation where link_scope is anyone_with_link and link_expiry is empty. The recorded link creation precedes the last upload by 19 minutes. Test how the source encodes missing or no-expiry values. | Section 8, stage 9 | Approved playbook / authorised responder |
| 9 | Alert on more than 20 EventCode 4769 requests from a single account within five minutes. Our burst was 63 in four minutes. | Section 8, stage 4 | Requires approval |
| 10 | Enable command-line logging on SRV-FS-01, SRV-BILLING and SRV-BKP-01. The 7z.exe execution is currently unanalysable, and this gap will recur in the next incident. | Section 6, item 6 | Requires approval |
| 11 | Restore proxy log delivery and bring WKS-OPS-03 and its peer workstations into proxy coverage. The published coverage ends roughly eight hours before the intrusion; monitoring-team awareness is not established. | Section 5c and section 6, item 5 | Requires approval |
| 12 | Define a mandatory response time for Defender detections of credential-theft tooling, with escalation if unacknowledged. The 00:52 detection is one documented opportunity for triage; acknowledgement and response are unknown. | Section 4, E-06 | Management |


## Long term

| # | Recommendation | Ties to finding | Authority |
| --- | --- | --- | --- |
| 13 | Enforce MFA on all VPN authentication with no exceptions, and make offboarding disable the account rather than only mark it. d.aviram left in June 2025 and could still sign in fourteen months later. | Section 4, E-01 | Management |
| 14 | Reduce the crown-jewel blast radius from WKS-OPS-03. Reported activity links one workstation to multiple sensitive assets. Validate the network paths, permissions, and existing controls before designing segmentation. | Section 4, E-02 to E-14 | Management |
| 15 | Extend logging to the OT network at the terminals and to the five assets currently without EDR, including both HMI terminals. We were unable to state whether operations were affected, which is a reporting failure as much as a security one. | Section 6, item 5 | Management / Legal |
| 16 | Establish who determines notification obligations for exfiltrated customer and invoice data, before the archive contents are known rather than after. This is not a technical decision. | Section 6, item 4 | Management / Legal |

# Documentation Validation Notes

The original working notes contained an incorrect attacker-IP assumption and time-zone interpretation. The report retains the analysis that identifies 212.143.200.14 as corporate egress. A separate completed AI-verification log is not published.

A later documentation review corrected inconsistent operation/file counts, unsupported response claims, draft instructions, and the distinction between suspicious behavior and proven effects. This review did not re-run the source dataset.

# Appendix A · Recorded Queries

Recorded training searches. Execution results are not published here. Before reuse, verify the index, field names and meanings, lookup schema, incident time bounds, and time-zone configuration. These are investigation searches, not deployed detection rules.

**Time handling:** `eval _time=_time-10800` reflects the original manual correction. Apply it only if the parsed epoch is actually shifted by three hours. A correctly parsed UTC+3 source already has the correct epoch; subtracting again would corrupt the chronology. UI display time zone and absolute search bounds must also be verified.

**Counting:** Q3 counts distinct `target_account` values, not distinct SPNs unless the field semantics establish that relationship. Q4 and Q14 count operations, not unique files. Q5 lists events without a final upload sum. Results depend on the selected range.

**Syntax correction:** Q15 now uses `where isnotnull(termination_date)`; its output still requires verification against the actual lookup.

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
| Q15 | 8 | \| inputlookup identity.csv \| where isnotnull(termination_date) \| table user department termination_date mfa_enrolled |
| Q16 | 9 | \| inputlookup assets.csv \| search criticality="crown-jewel" OR has_edr="no" \| table hostname criticality has_edr owner |

# Appendix B · Evidence Excerpts

Selected excerpts already present in the original report. They are not screenshots or a complete export and do not independently reproduce aggregate counts. Other timeline events have no published raw excerpt.

| Ref | Excerpt |
| --- | --- |
| E-01 | 2026-08-13 22:41:00, d.aviram, 194.63.180.117, RO, success, none, 214 |
| E-05 | 2026-08-14 03:14:09 (local), 4769, SRV-DC-01, d.aviram, WKS-OPS-03, cifs_svc, CIFS/SRV-FS-01.meridian.local  — one of 63 |
| E-06 | 2026-08-14 03:52:00 (local), 1116, WKS-OPS-03, d.aviram, C:\Users\Public\dmp64.exe, HackTool:Win32/CredDump.A |
| E-10 | 2026-08-14 04:48:00, y.shaked, share_link_create, /drive/Shared/ops-transfer/, anyone_with_link, none |
| E-13 | 2026-08-14 05:30:00, NIGHTLY-FULL, SRV-BKP-01, DELETED, 0, 0.0, d.aviram |
| E-15 | 2026-08-14 06:12:00, y.shaked, \\SRV-BILLING\Billing\2024\Q3\INV-377285.pdf.mrdn, rename, 0, WKS-OPS-03, SRV-BILLING  — one of 9,412 |
| E-16 | 2026-08-14 06:40:00, y.shaked, \\SRV-BILLING\Billing\HOW_TO_RESTORE.txt, write, 1834, WKS-OPS-03, SRV-BILLING |

