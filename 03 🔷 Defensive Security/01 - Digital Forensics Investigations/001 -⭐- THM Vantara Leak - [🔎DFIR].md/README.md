# The Vantara Leak — DFIR Investigation

A self-directed digital forensics investigation into a simulated compromise of Vantara Financial Group, based on TryHackMe's "The Vantara Leak" room.

## Approach

Unlike a typical guided CTF walkthrough, this investigation was conducted **without referencing the room's built-in questions**. Starting from a raw KAPE triage of the compromised endpoint (`VFG-CTR-W019`), the goal was to build a complete incident timeline from scratch: forming hypotheses, testing them against available artifacts, and revising them as evidence came in, rather than following a pre-defined question path.

## Scope

- **In scope:** Single endpoint (`VFG-CTR-W019`), KAPE-collected artifacts (Windows event logs, NTFS/MFT metadata, registry hives, Prefetch, PowerShell console history)
- **Out of scope:** Rest of the Vantara Financial Group network
- **Limitations:** Compromised user's `NTUSER.DAT` was missing from the triage; no browser, email, or network capture artifacts were available

## Summary of Findings

An attacker dropped a malicious payload (`VPNSetup_v2.1.exe`) onto the endpoint via SMB, then authenticated over RDP as `daniel.avery` to execute it. The payload dropped a masquerading second-stage binary (`svchosts.exe`), established two persistence mechanisms (a scheduled task disguised as an Edge updater, and a backdoor admin account named `helpdesk$`), ran discovery commands, staged a sensitive finance document for exfiltration, and attempted lateral movement into a file server using harvested credentials. Full technical detail, timeline tables, and MITRE ATT&CK mappings are in the attached report.

## Notable Detail: Anti-Forensic Clock Manipulation

Partway through the investigation, the timeline stopped lining up: Prefetch showed the first-stage payload executing *before* the MFT's creation timestamp for the second-stage payload. Rather than write this off as a bad artifact, I cross-referenced Windows Time Service operational logs, which track **tick count** (milliseconds since boot) alongside wall-clock time. Tick count is far harder to spoof than a system timestamp. That comparison revealed the system clock had been rolled back roughly 12 minutes at some point during the attack, an anti-forensic detail that would have gone unnoticed if timestamps had been taken at face value.

## Contents

- `vantara-leak.pdf` — Full investigation report (executive summary, timeline, technical analysis, recommendations, IoCs) || Note: Due to failures in GitHub displaying the PDF file, a long JPG has been provided to easily preview the PDF without downloading anything.
- Investigator notes and working timeline spreadsheet (linked in report appendix, and also underneath)

## Links
https://docs.google.com/document/d/1Lzx70mom2cZdvwT1mzPws1-OPwfbkCrnpnG5TmaEars/edit?usp=sharing - Investigator's Notes \
https://docs.google.com/spreadsheets/d/1lXCx9vvBQaPycv8Q_Qt2IPNul-2shAJdiJyBw84gj10/edit?usp=sharing - Investigator's Constructed Timeline

## Disclaimer

1. This is a fictional scenario from a TryHackMe training room. All entities, company names, and data referenced are simulated and used for educational purposes only.
2. This README was written by Claude and manually verified. The PDF report is still fully my work with little AI influence.
