# Active Directory Threat Detection Lab

## Project Overview

This project demonstrates the use of **[Wazuh SIEM](https://wazuh.com)** to monitor Windows security events and investigate a failed logon in my Active Directory lab environment.

The investigation follows a standard SOC analyst workflow: validate the monitoring setup, investigate security events, examine its fields, interpret the available evidence, and document the event along with the evidence.

## Objectives

- Demonstrate practical familiarity with the Wazuh SIEM dashboard and alert investigation.
- Identify and examine a Windows failed logon event.
- Correlate Windows Security Event ID `4625` with the corresponding Wazuh alert.
- Interpret authentication-related fields and gather evidence from the alerts.
- Document findings in the investigation report.

## Lab Architecture

The lab contains three primary systems:

| System | Role | Internal IP |
|---|---|---|
| DC01 | Windows Server domain controller and DNS | `192.168.50.10` |
| CLIENT01 | Domain-joined Windows client and Wazuh agent | `192.168.50.20` |
| SIEM01 | Ubuntu-based Wazuh server | `192.168.50.30` |

**Active Directory domain:** `lab.practice.local`

The Wazuh agent on CLIENT01 forwards endpoint security events to the Wazuh server for monitoring and investigation.

## Investigation Summary

A failed interactive logon involving the account `LAB\labuser` was examined on CLIENT01.

- **Windows event:** Security Event ID `4625` — An account failed to log on.
- **Wazuh rule:** `60122` — Logon Failure - Unknown user or bad password.
- **Wazuh alert severity:** Level `5`.
- **Logon type:** `2` — Interactive logon.
- **Failure status:** `0xC000006D`.
- **Failure substatus:** `0xC000006A`, consistent with an incorrect password.
- **Reported source address:** `127.0.0.1` (loopback).

The event was visible in Wazuh, and its fields provided useful context for assessing the failed authentication. The loopback address indicates that the event does not identify a remote source IP; therefore, this indicates this is not a remote attack but rather a local authentication failure.

## Key Evidence

### Wazuh Detection

![Wazuh authentication-failure results](screenshots/Screenshot%202026-10-24%20182420_authentication_failures.png)

Wazuh surfaced Windows authentication failures for investigation.

### Alert Investigation

![Wazuh alert details](screenshots/Screenshot%202026-10-24%20182650_detailed_alert_info_1.png)

![Wazuh alert details](screenshots/Screenshot%202026-10-24%20182729_detailed_alert_info_2.png)

The alert details provided the event ID, target account, logon type, and source address used to assess the failed logon.

## Conclusion

This project demonstrates an end-to-end example of client endpoint monitoring and investigation using Wazuh. I verified the endpoint's monitoring status, analyzed a failed logon alert, and examined its event fields to determine what the evidence suggests.

The available evidence supports classifying the event as a failed interactive logon consistent with an incorrect password. It does **not** establish a remote attacker's identity or prove malicious intent. In a SOC environment, this event would be documented and assessed alongside relevant authentication activity and other available endpoint or domain-controller logs before determining whether escalation is warranted.

For the technical details, evidence interpretation, and investigation limitations, see [investigation/investigation.md](investigation/investigation.md).

## Technologies

- Wazuh SIEM
- Windows Security Event Logging
- Active Directory
- Windows Server
- Windows client
- Ubuntu Server
- Oracle VirtualBox


