# Malicious Document Analysis Summary

## Document Information
- **Filename**: image_steg_malicious.docm
- **File Type**: Microsoft Word 2007+ Document with Macros (.docm)
- **Creator**: 3ef827ad4c04b6b04f4ea5dd46447c2e
- **Last Modified By**: TigerHunter
- **Created Date**: 2022-09-25 15:58:00 UTC
- **Modified Date**: 2022-09-25 21:56:00 UTC
- **Revision**: 4

## Executive Summary
This document contains malicious VBA macros designed to execute automatically when opened. The malware performs system reconnaissance, exfiltrates sensitive information, and modifies Windows registry settings to weaken Microsoft Office security protections.

## Malicious Behavior

### 1. Auto-Execution
The malware uses the `Document_Open()` event to automatically execute when the document is opened, requiring no user interaction beyond opening the file.

### 2. Information Gathering
The macros collect the following system information:
- **IP Address**: Retrieves local IP addresses from network adapters (excluding VMware adapters)
- **Hostname**: Obtains the computer name
- **OS Version**: Collects Windows operating system version information

### 3. Data Exfiltration
The collected information is transmitted to a remote Command & Control (C2) server:
- **C2 URL**: `http://word2022.c1.biz//index.php` (note: double slashes are in the original malware code)
- **Method**: HTTP GET request with parameters
- **Data Sent**: os=[OS Version]&name=[Hostname]&ip=[IP Address]
- **Technique**: Uses Microsoft.XMLHTTP object for communication

### 4. Security Settings Manipulation
The malware modifies Windows Registry to disable security features in Microsoft Word:

| Registry Key | Value | Impact |
|--------------|-------|--------|
| `AccessVBOM` | 1 | Enables access to VBA Object Model, allowing macros to modify other macros |
| `VBAWarnings` | 1 | Disables macro security warnings |
| `DisableAttachmentsInPV` | 1 | Disables Protected View for email attachments |
| `DisableUnsafeLocationsInPV` | 1 | Disables Protected View for unsafe locations |
| `DisableInternetFilesInPV` | 1 | Disables Protected View for files from the internet |

These registry modifications are written to: `HKCU\Software\Microsoft\Office\[Version]\Word\Security\`

## VBA Macro Code Structure

### Key Functions:

1. **Document_Open()**: Entry point - executes when document is opened
2. **GetIp()**: Retrieves local IP addresses via WMI
3. **GetHostName()**: Retrieves computer name using WinNTSystemInfo
4. **OsVersion()**: Retrieves Windows version via WMI
5. **HS86S0DEJ()**: Constructs and sends HTTP request to C2 server
6. **FDK346SSD()**: Modifies registry to disable security features

## Indicators of Compromise (IOCs)

### Network Indicators:
- **URL**: http://word2022.c1.biz//index.php (double slashes are in the original malware)
- **Domain**: word2022.c1.biz
- **Protocol**: HTTP (unencrypted)

### File Indicators:
- **Filename**: image_steg_malicious.docm
- **File Hash**: (MD5/SHA256 hashing recommended for deployment)

### Registry Modifications:
- Multiple registry keys under `HKCU\Software\Microsoft\Office\[Version]\Word\Security\`

### Behavioral Indicators:
- Document with no visible content
- Auto-execution on document open
- WMI queries for system information
- HTTP connections to suspicious domains
- Registry modifications to security settings

## Threat Classification

### MITRE ATT&CK Framework Mapping:
- **T1566.001**: Phishing - Spearphishing Attachment
- **T1204.002**: User Execution - Malicious File
- **T1059.005**: Command and Scripting Interpreter - Visual Basic
- **T1082**: System Information Discovery
- **T1016**: System Network Configuration Discovery
- **T1071.001**: Application Layer Protocol - Web Protocols
- **T1112**: Modify Registry
- **T1562.001**: Impair Defenses - Disable or Modify Tools

## Risk Assessment

**Severity**: HIGH

### Risks:
1. **Data Exfiltration**: Sensitive system information transmitted to attacker
2. **Persistence Preparation**: Security settings modifications enable future attacks
3. **Reconnaissance**: System profiling for targeted attacks
4. **Security Degradation**: Disabled protections leave system vulnerable

## Recommendations

### Immediate Actions:
1. **Do not open** this document on production systems
2. **Isolate** any systems that have opened this document
3. **Block** the domain word2022.c1.biz at network level
4. **Scan** affected systems for additional malware
5. **Reset** Microsoft Office security settings to defaults

### Registry Remediation:
Delete or reset the following registry values to default:
```
HKCU\Software\Microsoft\Office\[Version]\Word\Security\AccessVBOM (should be 0)
HKCU\Software\Microsoft\Office\[Version]\Word\Security\VBAWarnings (should be 2 or 3)
HKCU\Software\Microsoft\Office\[Version]\Word\Security\ProtectedView\* (should be 0)
```

### Long-term Prevention:
1. Implement application whitelisting
2. Disable macros from untrusted sources
3. Use Microsoft Office Protected View for internet files
4. Deploy email filtering to block suspicious attachments
5. Conduct security awareness training on phishing threats
6. Monitor network traffic for connections to known malicious domains
7. Implement endpoint detection and response (EDR) solutions

## Note on Steganography
Despite the filename suggesting image steganography, this document contains **no embedded images** or visual steganography. The filename "image_steg_malicious" appears to be a misdirection or previous version artifact. The malicious behavior is entirely contained within the VBA macros.

## Conclusion
This is a well-crafted malicious document designed for initial reconnaissance and security weakening. While it doesn't deploy additional payloads directly, it prepares the system for future compromise by:
1. Profiling the victim system
2. Reporting back to attackers
3. Weakening security defenses

The lack of visible content and immediate obvious malicious behavior makes this document particularly dangerous for untrained users.
