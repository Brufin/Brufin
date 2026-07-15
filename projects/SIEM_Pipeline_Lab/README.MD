# SIEM Pipeline Lab

## Project Overview

A hands-on SIEM and incident response lab designed to simulate a small SOC environment using Windows telemetry, centralized log analysis, and security case management. This project combines Sysmon, Wazuh, TheHive, Cassandra, Elasticsearch, Azure, VirtualBox, Shuffle, and VirusTotal to demonstrate practical blue-team and SOC Tier 1 skills.

## Project Goal

The goal of this project is to understand the core structure of SOC environments: how telemetry flows through the system, how EDR platforms process data by decoding events and applying rules that generate alerts, how alerts are enriched with open-source threat intelligence and automation, and how these tools work together to help a SOC analyst triage incidents.

## Repository Structure

- `README.md`: Main project overview, purpose, architecture, and key takeaways.
- `ARCHITECTURE.md`: High-level view of the lab design and data flow.
- `DEPLOYMENT.md`: Setup steps for the endpoint, SIEM server, and TheHive stack.
- `TROUBLESHOOTING.md`: Issues encountered during the build and how they were resolved.
- `DETECTION_SCENARIOS.md`: Example alerts, test cases, and detection logic.
- `INTEGRATION.md`: How Wazuh, Shuffle, VirusTotal, and TheHive work together.
- `SCREENSHOTS/`: Images used throughout the documentation to show proof of work.
- `diagrams/`: Architecture diagrams and any visual workflow references.

## Lab Environment

### Infrastructure / virtualization

- **VirtualBox:** The virtualization platform used to run the lab’s Windows endpoint in an isolated environment.
- **Azure Virtual Machines:** The cloud infrastructure used to host the Linux security servers.
- **Ubuntu 24.04:** The operating system running on the Azure-based servers.

### Endpoint

- **Windows 11:** The monitored endpoint operating system used to simulate user activity and generate security events.

### Telemetry collection

- **Sysmon:** The endpoint monitoring tool installed on Windows to capture detailed process, network, and file activity.

### SIEM / detection

- **Wazuh:** The SIEM and security monitoring platform that collects telemetry, applies rules, and generates alerts.

### Incident response

- **TheHive:** The incident response platform used to turn alerts into cases and support investigation workflows.

### Backend services

- **Cassandra:** The database used to store TheHive’s data.
- **Elasticsearch:** The indexing and search engine used by TheHive for fast querying.

### Automation

- **Shuffle:** The automation platform used to route alerts and trigger workflows.

### Threat intelligence

- **VirusTotal:** The threat intelligence service used to enrich hashes and add context to suspicious files.

## Why These Tools

I chose these tools primarily because they are open source and free, which makes the lab more accessible, flexible, and realistic for long-term learning. I used Azure for the server infrastructure because it is the cloud platform I am most familiar with, which allowed me to focus more on building and understanding the SIEM workflow rather than spending extra time learning a new environment.

## Architecture Summary

This diagram provides a very high-level view of the architecture and shows the main telemetry, alerting, automation, and case-management components and how they connect.

+-------------------+ +-------------------+ +-------------------+ +-------------------+
| Windows Endpoint | ---> | Wazuh Manager | ---> | Shuffle | ---> | TheHive |
| Telemetry | | Alerting | | Automation | | Case Management |
+-------------------+ +-------------------+ +-------------------+ +-------------------+

## What I Learned

- SIEM deployment and basic administration.
- Endpoint telemetry collection with Sysmon.
- Windows and Linux system troubleshooting.
- Cloud VM provisioning and configuration.
- Firewall and network access debugging.
- Log analysis and alert validation.
- Custom detection rule creation.
- Incident response workflow understanding.
- Security alert enrichment and automation.

## SOC Relevance

This project reflects the kind of workflow a SOC analyst works with every day: monitoring endpoint activity, identifying suspicious behavior, validating alerts, and gathering enough context to decide whether an event should be escalated. It also shows the value of automation and enrichment in reducing noise and helping analysts triage incidents faster and with more confidence.
