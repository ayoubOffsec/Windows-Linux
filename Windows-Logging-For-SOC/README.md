# Windows Logging for SOC

## Overview

Windows Logging is the process of recording system activities, user actions, application errors, and security-related events.

Windows logs help system administrators and SOC analysts monitor system behavior, investigate incidents, and detect suspicious activities.

This document provides a practical reference for understanding and analyzing Windows Event Logs.

---

## Table of Contents

- [1. Windows Event Viewer](#1-windows-event-viewer)
- [2. Windows Log Categories](#2-windows-log-categories)
- [3. Important Windows Event IDs](#3-important-windows-event-ids)
- [4. Understanding an Event](#4-understanding-an-event)
- [5. Common Security Investigations](#5-common-security-investigations)
- [6. PowerShell Log Analysis](#6-powershell-log-analysis)
- [7. Basic Investigation Workflow](#7-basic-investigation-workflow)
- [8. Important Analysis Principles](#8-important-analysis-principles)
- [9. Quick Reference](#9-quick-reference)

---

## 1. Windows Event Viewer

**Event Viewer** is a built-in Windows utility used to view, filter, and analyze system events.

### How to Open Event Viewer

1. Press `Win + R`.
2. Type `eventvwr.msc`.
3. Press `Enter`.

---

## 2. Windows Log Categories

Windows organizes its events into several categories.

| Log Category | Description |
|---|---|
| Application | Records application events and errors. |
| System | Records Windows system, driver, and service events. |
| Security | Records security-related activities, including logons and account changes. |
| Setup | Records Windows installation and setup events. |
| Forwarded Events | Stores events forwarded from other computers. |

**Important:** Security logs are particularly useful when investigating authentication and account-related activities.

---

## 3. Important Windows Event IDs

Event IDs identify specific activities recorded by Windows.

### 3.1 Authentication and Logon Events

| Event ID | Description |
|---|---|
| `4624` | Successful logon |
| `4625` | Failed logon |
| `4634` | Account logged off |
| `4647` | User initiated logoff |
| `4648` | Logon using explicit credentials |
| `4672` | Special privileges assigned to a new logon |

### 3.2 Account Management Events

| Event ID | Description |
|---|---|
| `4720` | User account created |
| `4722` | User account enabled |
| `4723` | Attempt to change an account password |
| `4724` | Attempt to reset an account password |
| `4726` | User account deleted |
| `4732` | Member added to a security-enabled local group |

### 3.3 Process and System Activity Events

| Event ID | Description |
|---|---|
| `4688` | New process created |
| `4689` | Process terminated |
| `4697` | Service installed |
| `7045` | New service installed (System log) |

> **Note:** Event availability depends on Windows audit policies and system configuration. Some events require additional auditing to be enabled.

---

## 4. Understanding an Event

When analyzing a Windows event, examine the following fields:

| Field | Description |
|---|---|
| Event ID | Identifies the recorded activity. |
| Time Created | Indicates when the event occurred. |
| Level | Information, Warning, or Error. |
| Source | Identifies the component that generated the event. |
| User / Account | Identifies the account associated with the activity. |
| Computer | Identifies the machine that generated the event. |
| Details | Provides additional event information. |

Always inspect both the **General** and **Details** tabs to understand the recorded activity.

---

## 5. Common Security Investigations

### 5.1 Detecting Failed Login Attempts

**Event ID:** `4625`

Investigation steps:

1. Open `Windows Logs → Security`.
2. Filter events using Event ID `4625`.
3. Identify the targeted account.
4. Examine the source IP address when available.
5. Look for repeated failures within a short period.

Repeated failed login attempts may indicate brute-force activity or unauthorized access attempts.

### 5.2 Investigating Successful Logins

**Event ID:** `4624`

Check the following:

- Account name.
- Login timestamp.
- Logon Type.
- Source Network Address, when available.

Unexpected login times, unfamiliar accounts, or unusual logon types may require further investigation.

### 5.3 Investigating New User Accounts

**Event ID:** `4720`

Check:

- Which account was created.
- When the account was created.
- Which account performed the creation.
- Whether the action was authorized.

### 5.4 Investigating Suspicious Processes

**Event ID:** `4688`

Check:

- Process name.
- Executable path.
- Process creator.
- Parent process, when available.
- Command-line arguments, if auditing is configured to record them.

Unexpected processes or unusual execution paths may indicate suspicious activity.

### 5.5 Investigating New Services

**Event ID:** `7045`

Check:

- Service name.
- Service executable path.
- Service start type.
- Whether the installation was authorized.

Unexpected service installations may indicate persistence or unauthorized system modifications.

---

## 6. PowerShell Log Analysis

PowerShell provides `Get-WinEvent` for retrieving and filtering Windows event logs.

### 6.1 View Recent System Events

```powershell
Get-WinEvent -LogName System -MaxEvents 20
```

### 6.2 View Recent Security Events

```powershell
Get-WinEvent -LogName Security -MaxEvents 20
```

### 6.3 Search for Failed Logins

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
} -MaxEvents 50
```

### 6.4 Search for Successful Logins

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
} -MaxEvents 50
```

### 6.5 Search for Process Creation Events

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4688
} -MaxEvents 50
```

> **Note:** Accessing Security logs may require administrator privileges. Process creation events and command-line details depend on the enabled audit policies.

---

## 7. Basic Investigation Workflow

A simple workflow for investigating Windows activity:

1. **Identify:** Determine what activity needs investigation.
2. **Locate:** Open the appropriate Windows event log.
3. **Filter:** Search by Event ID and relevant time range.
4. **Inspect:** Examine event details and associated accounts.
5. **Correlate:** Compare related events to build a timeline.
6. **Evaluate:** Determine whether the activity is normal or suspicious.
7. **Document:** Record findings and relevant timestamps.

---

## 8. Important Analysis Principles

- A single event does not necessarily indicate malicious activity.
- Always investigate the context surrounding an event.
- Correlate multiple events to understand what happened.
- Check timestamps and associated user accounts.
- Investigate unusual processes, services, and login activities.
- Consider legitimate administrative actions before classifying an event as suspicious.
- Missing logs do not necessarily mean an activity never occurred.

---

## 9. Quick Reference

| Activity | Event ID | Log |
|---|---|---|
| Successful login | `4624` | Security |
| Failed login | `4625` | Security |
| Special privileges assigned | `4672` | Security |
| New process created | `4688` | Security |
| User account created | `4720` | Security |
| User added to local security group | `4732` | Security |
| New service installed | `7045` | System |

---

## Conclusion

Windows Event Logs provide valuable information about system activity, authentication, account management, and process execution.

Understanding how to read and correlate these events helps identify suspicious behavior, investigate security incidents, and reconstruct activities performed on a Windows machine.

This knowledge is useful for Windows administration, troubleshooting, and basic security monitoring.
