# Roee Nir | Meridian SOC Investigation

A simulated team SOC capstone covering account misuse, suspicious Kerberos activity, lateral movement, cloud uploads, and ransomware indicators. The published work focuses on multi-source correlation, incident scoping, UTC timeline reconstruction, evidence limitations, and escalation recommendations.

## My Contribution

I served as Case Lead, with responsibility for the executive summary, incident classification and severity assessment, incident timeline, and recommendations. I also contributed to contextual analysis, MITRE ATT&CK mapping, and detection-gap review.

## Review the Work

| Artifact | What to review |
| --- | --- |
| [Analyst handover](report/HANDOVER.md) | Short case summary, key evidence, uncertainties, and requested next actions |
| [Investigation report](report/REPORT.md) | Timeline, entities, findings, recommendations, recorded SPL, and evidence excerpts |
| [Interactive timeline](https://roeeinvest18-tech.github.io/meridian-soc-investigation/timeline/) | Event sequence, source references, queries, and available report excerpts |
| [Static timeline](assets/meridian-timeline-vertical.svg) | Overview of the reported incident sequence |

## Evidence and Validation

Seven selected evidence excerpts are included in the report. Full source logs, query-output exports, screenshots, and presentation decks are not published in this repository. The timeline displays only existing report excerpts; it does not invent screenshots or missing log rows.

The report's queries were recorded for a training environment. Field semantics, time-zone handling, and output counts require source-dataset validation before reuse. Observed operations are distinguished from unique-file counts, encryption confirmation, and unverified response actions.

## Related Work

[SOC Tier 1 Portfolio](https://github.com/roeeinvest18-tech/soc-tier1-portfolio) — focused packet-analysis and SPL logic-review cases.

## Disclaimer

This is a fictional educational scenario, not a live customer incident or production SOC employment. Entities, accounts, hosts, and events are simulated. Response actions are recommendations; no live containment or remediation is claimed.
