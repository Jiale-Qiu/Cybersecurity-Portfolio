<a href="./">
  <img src="./Images/homelab_hero_banner.png" alt="Cybersecurity Portfolio" width="100%">
</a>

<a href="./">
  <img src="../misc (ignore)/images/03_glitch_divider_v5_1@2x.png" alt="divider" width="100%">
</a>

<a href="./">
  <img src="./Images/03_heading_2_overview.png" alt="overview" width="100%">
</a>

A hands-on security homelab built on physical hardware and virtual machines, where I practice my defensive skills by viewing attacks from both sides: attacking a deliberately vulnerable web server (OWASP Juice Shop) and examining it in the SIEM. Each entry documents what I built or tested, what went wrong, and how I fixed it.

<a href="./">
  <img src="../misc (ignore)/images/03_glitch_divider_v5_1@2x.png" alt="divider" width="100%">
</a>

<a href="./">
  <img src="./Images/03_heading_2_insidethelab.png" alt="overview" width="100%">
</a>

- **Network:** Extended star topology with an OPNsense router and managed switch in a Router-on-a-Stick design. Contains three segmented VLANs (DMZ, Security, simulated internet) using 802.1Q tagging, each with their own firewall rules validated with Nmap.
- **Monitoring:** Runs the Elastic Stack and SIEM (Elasticsearch, Kibana, Fleet) with Elastic Agents. Sysmon and Elastic EDR is installed for greater monitoring capability. Hosts an offline package registry, as the lab endpoints do not have internet access.
- **Detection testing:** MITRE ATT&CK techniques simulated against the Juice Shop endpoint to find and fix gaps in SIEM visibility.
- More to come

<a href="./">
  <img src="../misc (ignore)/images/03_glitch_divider_v5_1@2x.png" alt="divider" width="100%">
</a>

<a href="./">
  <img src="./Images/03_heading_2_tableofcontents.png" alt="overview" width="100%">
</a>

> Note: The count starts at 4 due to three archived entries before Summer 2026

| Entry | Description |
|-------|-------------|
| [4: Network Revamp](<04 - Network Revamp.md>) | Rebuilt a flat network into an RoaS setup with segmented VLANs with firewall rules |
| [5: Elastic SIEM Installation](<05 - Elastic SIEM Installation.md>) | Deployed the SIEM and agents on an isolated network |
| [6: Improving Endpoint Visibility](<06 - Improving Endpoint Visibility.md>) | Simulated ATT&CK techniques and fixed visibility gaps |