## Architecture Overview

This lab uses a layered architecture that separates endpoint telemetry, security analysis, automation, and incident response. Sysmon on the Windows endpoint generates detailed activity data, the Wazuh agent forwards it to the Wazuh server for analysis, and the alerting output can then be enriched through automation and organized into investigative cases. This provides a high-level view of how the individual systems work together in a SOC-style workflow.

## Major Components

- **Windows 11 Host:** The monitored endpoint that generates telemetry and test events.
- **Wazuh Manager:** The central detection and alerting layer that processes endpoint activity.
- **Shuffle:** The automation and enrichment layer that helps route alerts and add context.
- **TheHive:** The incident response and case management layer where alerts can be tracked and investigated.

## Data Flow

Telemetry begins on the Windows 11 host, where Sysmon extends native Windows logging by capturing richer endpoint activity such as process creation, network connections, and other security-relevant events. That data is collected by the Wazuh agent and forwarded to the Wazuh manager for analysis, where the analysis engine decodes the events, applies rules, and generates alerts when suspicious behavior is detected.

### Endpoint Telemetry

The first stage of the pipeline is the Windows endpoint. Windows already produces telemetry through its built-in event logging, but Sysmon adds deeper visibility. While native Windows telemetry offers broad system health data, Sysmon fills critical gaps by recording process creation with full command lines, network connections attributed to specific processes, and file creation time changes often used by attackers to obfuscate their tracks. This makes the endpoint data far more valuable before it ever reaches the SIEM.

### Wazuh Processing

The Wazuh agent on the Windows host forwards collected telemetry to the Wazuh manager over port 1514/TCP, which is the default communication channel for agent-to-manager traffic. If the agent is being enrolled automatically, port 1515/TCP is used for registration. Once the data reaches the manager, the Wazuh analysis engine decodes the logs, normalizes the fields, and applies built-in or custom rules to identify suspicious activity and generate alerts. Those alerts are then stored and made available for review in the dashboard and downstream workflows.

+---------------+ +----------------+ +-------------------+ +----------+
| Wazuh Agent | ---> | Wazuh Manager | ---> | Analysis Engine | ---> | Alerts |
| (Endpoint) | | (Receives Data)| | (Decoders/Rules) | | Output |
+---------------+ +----------------+ +-------------------+ +----------+

### Alert Enrichment

Once an alert is generated in the detection pipeline, it can be sent into Shuffle through a webhook to begin enrichment and automation. From there, relevant fields such as a file hash can be extracted and checked against threat intelligence sources like VirusTotal to add context to the event. This helps transform a raw alert into something more actionable by giving the analyst additional information for triage and investigation.

+----------+ +----------------+ +--------------+ +-------------+
| Webhook | --> | HASH-Extraction| --> | VirusTotal | --> | TheHive |
| Trigger | | Parse Hash | | Enrichment | | Case Mgmt |
+----------+ +----------------+ +--------------+ +-------------+

### Incident Handling

Once an alert has been enriched and reviewed, it can be escalated into an incident investigation in TheHive. From there, the alert is turned into a case where observables, comments, and supporting evidence can be tracked as part of the response process. This gives the workflow a clear endpoint for SOC-style triage and investigation.
