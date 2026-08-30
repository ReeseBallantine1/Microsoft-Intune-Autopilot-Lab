# Windows 11 Endpoint Management with Microsoft Intune and Windows Autopilot

A hands-on Microsoft Intune and Windows Autopilot lab demonstrating end-to-end Windows 11 provisioning, cloud identity, endpoint security, compliance, device-side validation, and troubleshooting.

> **Key outcome:** Successfully provisioned a Windows 11 endpoint through Windows Autopilot, joined it to Microsoft Entra ID, enrolled it into Microsoft Intune, deployed security controls including Microsoft Defender, BitLocker and Windows LAPS, and validated the resulting configuration directly from Windows.
>
> **Troubleshooting highlight:** Diagnosed an earlier Autopilot / Intune Management Extension provisioning failure using PowerShell, Event Viewer, MDM diagnostics, Windows services and registry analysis. A Windows Installer restriction (`DisableMSI = 2`) was identified as a likely blocker, the Intune configuration was corrected, and the endpoint was successfully reprovisioned.

**Core Technologies:** Microsoft Intune · Microsoft Entra ID · Windows Autopilot · Windows 11 · PowerShell · Microsoft Defender · BitLocker · Windows LAPS · Hyper-V

---

## Overview

This project simulates a modern cloud-managed Windows endpoint lifecycle: registering a device with Autopilot, provisioning it through OOBE, joining it to Microsoft Entra ID, enrolling it into Intune, applying security and configuration policies, evaluating compliance, validating controls directly on the endpoint, and troubleshooting a failed deployment through root-cause analysis and remediation.

The managed endpoint is a Windows 11 virtual machine running in Hyper-V. The project was created as a practical lab and portfolio environment rather than a production deployment.

## Skills Demonstrated

- Microsoft Intune administration and endpoint management
- Microsoft Entra ID device join and identity validation
- Windows Autopilot registration and user-driven deployment
- Enrollment Status Page configuration
- Intune Management Extension validation
- Settings Catalog and security policy deployment
- Windows compliance policy configuration and remediation
- Microsoft Defender Antivirus and Firewall management
- Attack Surface Reduction rule deployment
- BitLocker encryption and recovery key escrow
- Windows LAPS configuration and password escrow
- TPM and Secure Boot validation
- PowerShell-based endpoint validation
- Event Viewer and MDM diagnostics
- Policy troubleshooting and root-cause analysis

---

## Lab Architecture

```text
Microsoft Entra ID
        |
        v
Microsoft Intune
        |
        +-- Windows Autopilot
        +-- Configuration Profiles
        +-- Compliance Policies
        +-- Defender / Firewall / ASR
        +-- BitLocker
        +-- Windows LAPS
        |
        v
Windows 11 Hyper-V Endpoint
        |
        +-- Microsoft Entra Joined
        +-- Intune Enrolled
        +-- Intune Management Extension
        +-- Device-side validation with PowerShell
```

## End-to-End Workflow

**Hardware Hash Collection → Autopilot Registration → Profile Assignment → OOBE Sign-In → Enrollment Status Page → Microsoft Entra Join → Intune Enrollment → Security Policy Deployment → Compliance Evaluation → Remediation → Validation**

---

## 1. Windows Autopilot Registration

The Windows 11 virtual machine was registered with Windows Autopilot by collecting its hardware hash and importing the resulting CSV into Microsoft Intune.

The workflow included hardware hash collection, Autopilot registration, creation of a user-driven Microsoft Entra joined deployment profile, profile assignment, and confirmation that the profile status was **Assigned**.

![Autopilot user-driven deployment profile](images/autopilot/Autopilot-User-Driven-Profile.png)

## 2. Enrollment Status Page and Provisioning

The Enrollment Status Page was configured to display provisioning progress and block device use while required configuration was being applied.

![Enrollment Status Page configuration](images/autopilot/Autopilot-ESP-Configuration.png)

The successful deployment completed the critical provisioning stages:

- **Device preparation — Completed**
- **Device setup — Completed**
- Device continued successfully to the Windows desktop

![Successful Autopilot ESP provisioning](images/autopilot/Autopilot-ESP-Success-BT-LAPTOP-02.png)

## 3. Microsoft Entra Join and Intune Enrollment

After Autopilot provisioning, the endpoint successfully joined Microsoft Entra ID and enrolled into Microsoft Intune.

Device-side validation with `dsregcmd /status` confirmed the Microsoft Entra join state and provided endpoint-side evidence rather than relying only on portal status.

![Microsoft Entra device join validation](images/validation/Entra-Device-Join-dsregcmd-Redacted.png)

## 4. Intune Management Extension Validation

The Microsoft Intune Management Extension was validated after the successful rebuild. The Windows work/school management page showed centrally applied policy categories including ADMX / Windows Explorer, BitLocker, Defender, Device Lock, Security, and System.

The Intune Management Extension application status reported **EnforcementCompleted**.

![Intune Management Extension EnforcementCompleted](images/validation/Intune-Management-IME-EnforcementCompleted-Redacted.png)

This was an important validation point because the earlier failed endpoint had not established the expected IME state.

---

## 5. Endpoint Security and Configuration

### Device Configuration

A Windows 11 configuration profile was created with Device Lock controls including password enforcement, failed-attempt limits, inactivity lock, and minimum password length requirements.

### Microsoft Defender Firewall

Firewall policy was deployed and validated with PowerShell. Domain, Private, and Public firewall profiles reported as enabled.

![Microsoft Defender Firewall PowerShell validation](images/security/Firewall-PowerShell-Validation.png)

### Attack Surface Reduction

Attack Surface Reduction rules were deployed through Intune and verified from PowerShell by reviewing configured ASR rule IDs and actions.

![Attack Surface Reduction PowerShell validation](images/security/ASR-PowerShell-Validation.png)

### BitLocker

BitLocker was deployed and validated from the endpoint, with recovery key escrow demonstrated in the management environment.

![BitLocker fully encrypted endpoint validation](images/bitlocker/BitLocker-Fully-Encrypted.png)

![BitLocker recovery key escrow](images/bitlocker/BitLocker-Recovery-Key-Escrow.png)

> **Security note:** Recovery passwords are not published in this repository.

### Windows LAPS

Windows LAPS was configured through Intune to demonstrate managed local administrator password rotation and escrow.

![Windows LAPS password escrow with password masked](images/laps/LAPS-Password-Escrow-Masked.png)

> **Security note:** LAPS passwords are masked or redacted in all public evidence.

---

## 6. Compliance and Remediation

A Windows compliance policy evaluated endpoint security requirements including Firewall, Antivirus, BitLocker, Microsoft Defender Antimalware, real-time protection, Secure Boot, Defender security intelligence currency, and TPM.

![Intune security compliance all settings compliant](images/compliance/Intune-Security-Compliance-All-Settings-Compliant.png)

During validation, Microsoft Defender security intelligence was initially not current. Rather than weakening the compliance requirement, the issue was remediated using the Intune device action **Update Windows Defender security intelligence**.

The remote action completed successfully, the endpoint checked in again, and the device returned to **Compliant** status.

![Defender security intelligence remediation completed and device compliant](images/compliance/Intune-Defender-Intelligence-Remediation-Compliant.png)

---

## 7. Device-Side Validation

Portal status alone was not treated as sufficient evidence of successful deployment. Controls were also validated directly from Windows using:

- `dsregcmd /status`
- PowerShell
- TPM and Secure Boot checks
- BitLocker status
- Defender and Firewall cmdlets
- Work/school management information
- Event Viewer
- MDM diagnostic logs

This provided evidence that configured policies had actually reached and affected the endpoint.

---

## 8. Troubleshooting Case Study: Autopilot / IME Failure

The most valuable troubleshooting exercise in the project came from an earlier Windows Autopilot deployment failure.

### Symptoms

The original endpoint repeatedly failed during the Autopilot Enrollment Status Page while preparing the device for mobile management.

![Autopilot deployment failure](images/troubleshooting/Autopilot-Deployment-Failure-Redacted.png)

The expected Intune Management Extension installation state was absent from the endpoint.

![Intune Management Extension program files absent](images/troubleshooting/IME-Program-Files-Absent.png)

![Intune Management Extension service absent](images/troubleshooting/IME-Service-Absent.png)

MDM diagnostic logs provided further evidence through Event ID 404 and ADMXInstall failures.

![MDM Event ID 404 ADMXInstall errors](images/troubleshooting/MDM-Event-404-ADMXInstall.png)

### Investigation

Registry inspection identified:

```text
HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
DisableMSI = 2
```

The Intune Windows security configuration contained Windows Installer restrictions equivalent to disabling Windows Installer **Always**. This restriction was identified as a likely blocker interfering with the expected Intune Management Extension deployment state.

### Remediation

The Windows Installer restriction was removed from the Intune security configuration while retaining the intended SmartScreen security setting.

A clean Windows 11 VM was then created to eliminate stale local policy state. Before enrollment, the rebuilt endpoint was checked and the previous `DisableMSI` registry value was no longer present.

The new device was registered with Windows Autopilot and provisioned again.

### Result

The rebuilt endpoint successfully:

- Completed Autopilot device preparation and device setup
- Reached the Windows desktop
- Joined Microsoft Entra ID
- Enrolled into Microsoft Intune
- Established the Intune Management Extension
- Reported **EnforcementCompleted**
- Applied security and configuration policies
- Reached **Compliant** status

**Troubleshooting summary:**

**Deployment Failure → Sidecar / IME Investigation → Missing IME State → Event 404 / ADMX Errors → Windows Installer Restriction Identified → Policy Corrected → Clean Rebuild → Successful Autopilot Provisioning → IME Functional → Device Compliant**

---

## Evidence Structure

```text
images/
├── autopilot/
├── security/
├── compliance/
├── bitlocker/
├── laps/
├── validation/
└── troubleshooting/
```

Selected screenshots are used rather than publishing every captured image, keeping the repository readable while providing evidence for each major stage.

## Project Status

- [x] Windows 11 Hyper-V endpoint deployed
- [x] Microsoft Entra ID integration and join validation
- [x] Microsoft Intune enrollment
- [x] Windows Autopilot registration and user-driven deployment
- [x] Enrollment Status Page configuration and successful provisioning
- [x] Intune Management Extension validation
- [x] Windows configuration and endpoint security policies
- [x] Microsoft Defender Firewall and Attack Surface Reduction
- [x] BitLocker encryption and recovery key escrow
- [x] Windows LAPS configuration and password escrow
- [x] Windows compliance policy and remediation
- [x] PowerShell-based device validation
- [x] Autopilot / IME troubleshooting and remediation
- [x] Final compliant device state

---

## Security and Privacy

Screenshots are reviewed before publication. Sensitive or unnecessary environment-specific data is masked or redacted where appropriate, including user email addresses / UPNs, tenant identifiers, unnecessary device identifiers, serial numbers, BitLocker recovery passwords, Windows LAPS passwords, authentication tokens, secrets, and credentials.

Technical evidence such as policy names, event IDs, compliance results, PowerShell output, configuration state, and troubleshooting details is retained where it does not expose sensitive information.

## Key Takeaways

This project demonstrates an end-to-end cloud endpoint lifecycle rather than only the creation of Intune policies. It combines provisioning, identity, endpoint security, compliance, remediation, endpoint-side validation, and structured troubleshooting.

The strongest technical outcome was diagnosing a failed Autopilot deployment, tracing the problem through endpoint state and MDM diagnostics, identifying a Windows Installer restriction as a likely blocker, correcting the configuration, rebuilding the endpoint, and validating successful provisioning and compliance afterward.

The project demonstrates practical skills relevant to endpoint support, desktop engineering, infrastructure support, systems administration, and modern workplace roles.

---

## Disclaimer

This project was created in a lab environment for educational and portfolio purposes. It does not represent a production Microsoft Intune deployment. Configuration decisions were made to demonstrate endpoint management, security, compliance, provisioning, and troubleshooting concepts in a controlled environment.
