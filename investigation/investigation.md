# Security Event Investigation: Failed Interactive Logon

## 1. Executive Summary

This investigation examines a failed interactive logon involving `LAB\labuser` on CLIENT01, a Windows endpoint joined to the Active Directory domain `lab.practice.local`.

Windows recorded the event as Security Event ID `4625`. Wazuh detected the event and generated an alert under rule `60122`, allowing the authentication failure to be reviewed through the SIEM dashboard.

The event's status and substatus are consistent with an incorrect password. The recorded source address, `127.0.0.1`, is a loopback address and does not identify a remote originating host.

**Finding:** The investigation confirmed a failed interactive logon consistent with an incorrect password. Wazuh successfully surfaced the event for analysis. Based on the available evidence, the event is classified as a failed authentication attempt rather than a security incident.

## 2. Lab Environment and Monitoring Validation

The lab uses Active Directory for domain services and Wazuh to monitor security events collected from a domain-joined Windows endpoint.

| Component | Role | Internal IP |
|---|---|---|
| DC01 | Windows Server domain controller and DNS | `192.168.50.10` |
| CLIENT01 | Domain-joined Windows endpoint and Wazuh agent | `192.168.50.20` |
| SIEM01 | Ubuntu-based Wazuh server | `192.168.50.30` |

**Active Directory domain:** `lab.practice.local`

### Wazuh agent status

![Wazuh agent status](screenshots/Screenshot%202026-10-24%20181134_agent_list.png)

The Wazuh agent list shows CLIENT01 as active, confirming that the endpoint was connected to the monitoring environment when the status was captured.

### Domain membership

![Windows domain membership verification](screenshots/Screenshot%202026-10-24%20181801_domain_info.png)

The endpoint's system information confirms that CLIENT01 belongs to `lab.practice.local`. This establishes the domain context for the authentication event examined in this report.

### Network and DNS configuration

![Client network and DNS configuration](screenshots/Screenshot%202026-10-24%20181916_network_info.png)

The network configuration shows the endpoint's internal address as `192.168.50.20`, with `192.168.50.10` configured as DNS for the internal lab network. These details provide context for the endpoint's role in the environment.

The investigation focuses on the event collected from CLIENT01. It does not claim that Wazuh was collecting events directly from DC01.

## 3. Event Identification and Analysis

The Windows Security log recorded a failed logon involving `LAB\labuser`.

| Field | Observed value |
|---|---|
| Event timestamp | `2026-10-24 18:01:20` local time |
| Windows event ID | `4625` |
| Computer | `CLIENT01.lab.practice.local` |
| Target account | `LAB\labuser` |
| Logon type | `2` — Interactive |
| Failure reason | Unknown user name or bad password |
| Status | `0xC000006D` |
| Substatus | `0xC000006A` |
| Source address | `127.0.0.1` |
| Source port | `0` |
| Workstation name | `CLIENT01` |
| Caller process | `C:\Windows\System32\svchost.exe` |

### Interpretation of the event fields

- **Event ID `4625`:** Indicates that a logon attempt failed.
- **Logon type `2`:** Identifies the attempt as an interactive logon.
- **Status `0xC000006D`:** Indicates a logon failure.
- **Substatus `0xC000006A`:** Is consistent with an incorrect password.
- **Source address `127.0.0.1`:** Is the loopback address. It does not identify a remote source host.
- **Caller process:** `svchost.exe` is recorded in the event, but this field alone does not establish which application or person initiated the attempt.

Taken together, the fields support the conclusion that the interactive authentication attempt failed, with an incorrect password being the likely explanation. The event alone does not establish the user's intent or prove malicious activity.

## 4. Wazuh Detection and Alert Review

### Authentication-failure results

![Wazuh authentication-failure results](screenshots/Screenshot%202026-10-24%20182420_authentication_failures.png)

The Wazuh Threat Hunting dashboard displayed two authentication failures, corresponding to the two intentional failed logon attempts performed during testing.

### Detailed alert evidence

![Wazuh alert event details](screenshots/Screenshot%202026-10-24%20182650_detailed_alert_info_1.png)

![Additional Wazuh alert event details](screenshots/Screenshot%202026-10-24%20182729_detailed_alert_info_2.png)

The detailed alert fields provide the information used to examine the event, including the target account, Windows event ID, logon type, source address, and endpoint identity.

The relevant Wazuh alert fields are:

| Field                     | Observed value                                 |
| ------------------------- | ---------------------------------------------- |
| Rule ID                   | `60122`                                        |
| Rule description          | `Logon Failure - Unknown user or bad password` |
| Alert level               | `5`                                            |
| Agent name                | `CLIENT01`                                     |
| Agent ID                  | `001`                                          |
| Agent IP                  | `192.168.50.20`                                |
| Manager                   | `SIEM01`                                       |
| Windows event ID          | `4625`                                         |
| Alert index               | `wazuh-alerts-4.x-2026.10.24`                  |
| Wazuh alert timestamp     | `2026-10-24T22:01:24.323Z` (UTC)               |
| Windows event system time | `2026-10-24T22:01:20.4027613Z` (UTC)           |

The Windows event system time and Wazuh alert timestamp are approximately four seconds apart. This indicates that Wazuh surfaced the event shortly after the recorded Windows event time.

The alert rule classified the activity as a logon failure involving an unknown username or incorrect password. Reviewing the underlying event fields provides additional context beyond the rule description and helps avoid relying on the alert label alone.

## 5. Findings
### Findings supported by the evidence

1. CLIENT01 recorded a failed interactive logon involving `LAB\labuser`.
2. The event was available in Wazuh and generated an alert under rule `60122`.
3. The status and substatus are consistent with an incorrect password.
4. The recorded source address is `127.0.0.1`, not an identified remote source IP.
5. The recorded event and alert timestamps indicate that Wazuh surfaced the event within approximately four seconds.

These limitations matter because a failed logon is a security-relevant event, but it is not automatically evidence of an attack.

## 6. Recommended SOC Follow-up

In a production environment, the next steps would be to:

1. **Review surrounding authentication activity.** Check whether the account has other failed or successful logons around the event time.
2. **Correlate with domain-controller logs.** Review relevant Active Directory security events for corroborating authentication activity involving the account.
3. **Validate expected activity.** Determine whether the logon attempt aligns with legitimate user activity or an expected test.
4. **Investigate further if warranted.** Examine additional endpoint or authentication telemetry if repeated failures, unusual activity, or other indicators raise concern.
5. **Determine disposition based on corroboration.** Escalate if additional evidence supports suspicious behavior; otherwise, document the finding according to the organization's procedures.

These are recommended next steps. They are not represented as actions completed during this lab investigation.

## 7. Conclusion

This investigation demonstrated the use of Wazuh to identify and examine a Windows authentication failure. CLIENT01 generated Security Event ID `4625`, and Wazuh surfaced the event under rule `60122`. Examination of the event fields supports the conclusion that an interactive logon failed in a manner consistent with an incorrect password.

The source address was the loopback address `127.0.0.1`, and the available evidence did not identify a remote originating host or establish malicious intent. The appropriate disposition is therefore to document the event as a failed authentication attempt, with further correlation recommended if this occurred in a production environment.

The investigation demonstrates a realistic SIEM workflow: validating endpoint monitoring, locating an alert, examining its underlying fields, interpreting the evidence, and reaching a conclusion that stays within what the logs support.
