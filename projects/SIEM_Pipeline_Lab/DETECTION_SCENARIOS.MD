### Suspicious PowerShell Execution

#### Objective

PowerShell is frequently abused by attackers because it is a legitimate Windows administration tool capable of executing scripts, downloading payloads, and running code directly in memory. This scenario validates that Wazuh can detect suspicious PowerShell command-line arguments commonly associated with post-exploitation and defense evasion techniques.

#### Detection Logic

A custom Wazuh rule was created in `local_rules.xml` to monitor Sysmon Event ID 1 (Process Creation). The rule inspects PowerShell command-line arguments for keywords commonly observed during malicious activity.

```XML
<rule id="100003" level="10">
  <if_group>sysmon_event1</if_group>
  <field name="win.eventdata.image" type="pcre2">(?i)\\powershell\.exe|\\pwsh\.exe</field>
  <field name="win.eventdata.commandLine" type="pcre2">(?i)(-enc|-encodedcommand|iex|invoke-expression|downloadstring|webclient|new-object\s+net\.webclient)</field>
  <description>Suspicious PowerShell execution detected.</description>
  <mitre>
    <id>T1059.001</id>
  </mitre>
</rule>
```

#### Why This Rule?

The rule looks for command-line indicators commonly used by attackers, including:

- `-EncodedCommand` to obfuscate PowerShell execution
- `Invoke-Expression (IEX)` for dynamic code execution
- `DownloadString()` and `WebClient` for payload retrieval
- `New-Object Net.WebClient` for download cradle techniques

These behaviors are regularly observed in malware, red team tooling, and penetration testing frameworks.

#### Test Procedure

An elevated PowerShell session was opened on the Windows 11 endpoint and a command containing one of the monitored patterns was executed.

Sysmon generated a Process Creation event, which was forwarded to Wazuh and matched against the custom rule.

#### Alert Analysis

During triage, the following fields were verified:

- Process Image
- Parent Process
- Full Command Line
- User Account
- Hostname
- Event Timestamp
- Rule Level (10)

The command line contained one of the monitored PowerShell indicators, causing the event to match the detection rule.

#### MITRE ATT&CK

| Technique | Description |
| --------- | ----------- |
| T1059.001 | PowerShell  |

#### Analyst Assessment

This alert represents suspicious PowerShell execution consistent with attacker tradecraft. In a production SOC, the next investigative steps would include:

- Reviewing the complete command line
- Identifying the parent process
- Determining whether the script was administrator-initiated
- Searching for additional PowerShell activity from the same endpoint
- Checking for subsequent network connections or child processes

Since this activity was intentionally generated during testing, the alert was classified as a **true positive**.

---

## Scenario 2

### Mimikatz Execution

#### Objective

Credential dumping is a common post-exploitation technique used to obtain passwords, NTLM hashes, and Kerberos tickets from memory. This scenario validates that Wazuh detects execution of Mimikatz using Sysmon process creation events.

#### Detection Logic

A custom rule was created to monitor Sysmon Event ID 1 and inspect the `OriginalFileName` metadata embedded in PE files.

```XML
<rule id="100002" level="10">
    <if_group>sysmon_event1</if_group>
    <field name="win.eventdata.originalFileName" type="pcre2">(?i)mimikatz\.exe</field>
    <description>Mimikatz usage detected.</description>
    <mitre>
        <id>T1003</id>
    </mitre>
</rule>
```

#### Why This Rule?

Attackers frequently rename Mimikatz to evade simple filename-based detections. The `OriginalFileName` field, however, often retains the value embedded during compilation, making it a more resilient detection method.

#### Test Procedure

Mimikatz was executed on the Windows 11 endpoint.

Sysmon generated a Process Creation event that included the PE metadata.

Wazuh successfully matched the `OriginalFileName` field and generated an alert.

#### Alert Analysis

The following information was reviewed:

- Executable path
- Original File Name
- Parent Process
- Executing User
- Hostname
- Timestamp

The alert confirmed that the executed binary contained the original filename `mimikatz.exe`, triggering the custom rule.

#### MITRE ATT&CK

| Technique | Description           |
| --------- | --------------------- |
| T1003     | OS Credential Dumping |

#### Analyst Assessment

Credential dumping tools represent high-confidence malicious activity.

A Tier 1 analyst would escalate this alert immediately while collecting additional context, including:

- Evidence of LSASS access
- Recent privileged logons
- Additional credential access attempts
- Lateral movement indicators
- Network connections initiated by the process

This activity was intentionally generated during testing and was confirmed as a **true positive**.

---

## Scenario 3

### Registry Run Key Persistence

#### Objective

Windows Registry Run Keys provide a common persistence mechanism by automatically launching programs during user logon. This scenario validates Wazuh's ability to detect modifications to these registry locations.

#### Detection Logic

A custom rule monitors Sysmon Event ID 13 (Registry Value Set) for changes made to the Windows Run key.

```XML
<rule id="100004" level="10">
  <if_group>sysmon_event13</if_group>
  <field name="win.eventdata.targetObject" type="pcre2">(?i)\\software\\microsoft\\windows\\currentversion\\run\\</field>
  <description>Potential registry run key persistence detected.</description>
  <mitre>
    <id>T1547.001</id>
  </mitre>
</rule>
```

#### Why This Rule?

Attackers commonly establish persistence by creating registry Run Keys that automatically execute malware whenever a user logs into Windows.

Monitoring these registry locations provides visibility into one of the most frequently abused persistence mechanisms.

#### Test Procedure

A benign startup entry was created using PowerShell:

```
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "WazuhTest" -Value "notepad.exe" -PropertyType String -Force
```

Sysmon recorded the registry modification, and Wazuh generated an alert based on the custom detection rule.

#### Alert Analysis

The investigation focused on:

- Modified registry key
- Registry value name
- Registry value data
- Executing process
- User Account
- Timestamp

The alert showed that a new value had been written to the Run key, indicating the creation of an automatic startup entry.

#### MITRE ATT&CK

| Technique | Description                        |
| --------- | ---------------------------------- |
| T1547.001 | Registry Run Keys / Startup Folder |

#### Analyst Assessment

Registry Run Key modifications warrant investigation because they are a common persistence mechanism.

A Tier 1 analyst would validate:

- Whether the startup entry is expected
- The reputation of the executable
- File location and digital signature
- Recent activity on the endpoint
- Whether similar persistence exists elsewhere in the environment

Because the registry modification was intentionally created during testing, the alert was verified as a **true positive**.
