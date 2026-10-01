# Analyst Handover — Meridian Training Scenario

**Case:** Meridian Energy Ltd (fictional capstone)  
**Analyst role:** Roee Nir, Case Lead; team investigation  
**Window:** 13 August 2026 22:41 UTC through the reported activity on 14 August  
**Disposition:** Escalate for IR and recovery validation  
**Training severity assessment:** Critical, based on sensitive assets, ransomware indicators, service unavailability, and unresolved recovery  
**Containment status:** Unknown; no published record establishes completed response actions

## Summary

A terminated account authenticated through VPN without MFA. The reported sequence includes discovery commands, suspicious Kerberos requests, a Defender credential-dumping-tool detection, lateral logons, bulk reads, public cloud sharing and archive uploads, then a rename burst and ransom-note write. Portal HTTP 503 responses began at 07:04 UTC.

## Key Evidence

| Time (UTC) | Finding | Reference |
| --- | --- | --- |
| 13/08 22:41 | d.aviram VPN success; mfa=none; marked terminated | E-01 |
| 14/08 00:14–00:18 | 63 Kerberos ticket requests to 21 reported targets | E-05 |
| 14/08 00:52 | Defender HackTool:Win32/CredDump.A detection | E-06 |
| 14/08 03:05–03:54 | 840 read operations; 3.20 GB | E-08 |
| 14/08 04:48 | anyone_with_link share; no expiry | E-10 |
| 14/08 04:52–05:07 | Four archive uploads; 1.32 GB | E-11 |
| 14/08 05:30 | Backup job DELETED status; zero files / zero GB | E-13 |
| 14/08 06:12–06:40 | 9,412 rename operations followed by ransom-note write | E-15, E-16 |

Counts are retained from the original report, not re-computed. Selected excerpts cover only some events.

## Entities to Validate

Accounts: d.aviram, y.shaked. Hosts: WKS-OPS-03, SRV-DC-01, SRV-FS-01, SRV-BKP-01, SRV-BILLING, WEB-PORTAL. External addresses: 194.63.180.117 and 194.63.180.204; shared /24 does not establish common ownership. Corporate egress 212.143.200.14 was ruled out as an attacker indicator.

## Unknowns

Credential origin; successful ticket cracking or credential dumping; archive contents; unique-file counts and file-content encryption; actual backup destruction and restore-point survival; web shell deployment/execution; incident-period proxy traffic; OT impact; completed containment and alert-handling history.

## Requested Next Actions

1. Validate current identity/session and public-link exposure; revoke access under the approved response workflow.
2. Request approved host isolation and evidence preservation where warranted by current activity.
3. Verify usable restore points on the backup system.
4. Investigate WEB-PORTAL artifacts without assuming the suspicious POST proves a web shell.
5. Retrieve source logs, query outputs, response records, and missing telemetry; verify time-zone parsing before correlating events.

These are recommendations, not completed actions.

[Full report, query caveats, and evidence excerpts](REPORT.md) · [Interactive timeline](https://roeeinvest18-tech.github.io/meridian-soc-investigation/timeline/)
