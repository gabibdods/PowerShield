# Windows 11 Security Hardening

### Description

- This project implements a rigorous Windows 11 hardening procedure, combining PowerShell security configuration, script execution restrictions, and code signing enforcement
- It is designed to mitigate common attack vectors while maintaining flexibility for developers and advanced users

---

## Problem Statement

- Secure the Windows 11 operating system by disabling legacy scripting engines, enforcing language mode constraints, and enabling script validation through code signing, all while ensuring the system remains usable by developers and power users

---

## Project Goals

### Harden the Windows Environment

- Apply advanced policies and restrictions to the system's scripting capabilities to enforce constrained execution and reduce attack surfaces

### Preserve Developer Agility

- Allow legitimate developers and users to continue writing and executing custom PowerShell scripts using signed code while blocking unauthorized execution paths

---

## Tools, Materials & Resources

### Tools

- PowerShell (Admin Mode): for configuring execution policies, generating certificates, and signing scripts
- PolicyPlus: to enable group policy editing on Windows 11 Home and apply execution policy restrictions

### Materials

- PowerShell script: provided to demonstrate signing and lockdown validation

### Resources

- PowerShell GitHub: download latest PowerShell builds and ADMX templates
- PolicyPlus GitHub: alternative group policy editor for non-Pro editions

---

## Design Decision

### Disable Legacy PowerShell Engines

- PowerShell 2.0 is deprecated and vulnerable; disabling it removes a common backdoor exploited by attackers

### Enforce Constrained Language Mode

- Setting `__PSLockDownPolicy=4` forces PowerShell into ConstrainedLanguage mode, significantly reducing script-based attack capabilities

### Require Script Signing

- All local scripts must be signed using a self-issued code-signing certificate; execution policies such as `AllSigned` or `Restricted` are enforced via PolicyPlus

---

## Features

### System-Level Lockdown

- Implements environmental restrictions via system variables and group policies to prevent full scripting capabilities

### Code-Signing Infrastructure

- Generates a local certificate, installs it in the trusted root store, and applies it to custom scripts using `Set-AuthenticodeSignature`

### Compatibility with Developers

- While the system enforces hardened settings, developers can still run signed scripts, preserving local automation workflows

---

## Procedure

### Applying Security Patches for Windows 11

1. Go to Turn Windows features on or off and disable Windows PowerShell 2.0. After a reboot, PowerShell 2.0 should be unavailable, for the better.

2. Open Edit the system environment variables, click Environment Variables, add a new system variable named `__PSLockDownPolicy` with value `4`. Verify by running in PowerShell:
```plaintext
$ExecutionContext.SessionState.LanguageMode
```
It should return ConstrainedLanguage instead of FullLanguage.

3. Download [PolicyPlus](https://github.com/Fleex255/PolicyPlus). Launch it, navigate to Windows Components > Windows PowerShell, set Turn on Script Execution to Disabled, then save policies in the File tab. If you exit without saving the policies, change won't be applied!

4. Download the latest PowerShell 7 from [PowerShell](https://github.com/PowerShell/PowerShell) GitHub. Unzip the download, copy `PowerShellCoreExecutionPolicy.admx` to `C:/Windows/PolicyDefinitions` and `PowerShellCoreExecutionPolicy.adml` to `C:/Windows/PolicyDefinitions/en-US` and all the other language folders on the same directory level.

5. In PolicyPlus, go to PowerShell Core and disable Turn on Script Execution, and save.

### Creating a Code Signing Certificate

1. Open PowerShell with Administrator privileges.

2. Run:
```plaintext
$CertificateName = "PSCS"
```

3. Run:
```plaintext
$Certificate = New-SelfSignedCertificate -CertStoreLocation Cert:\CurrentUser\My -Subject "CN=$CertificateName" -KeySpec Signature -Type CodeSigningCert
```

4. Run:
```plaintext
$CertificatePath = [Environment]::GetFolderPath("MyDocuments")+"\$CertificateName.cer"
```

5. Run:
```plaintext
Export-Certificate -Cert $Certificate -FilePath $CertificatePath
```

6. Import to Trusted Root:
```plaintext
Get-Item $CertificatePath | Import-Certificate -CertStoreLocation "Cert:\LocalMachine\Root"
```

7. Display Thumbprint:
```plaintext
Write-host "Certificate Thumbprint: " $Certificate.Thumbprint
```

8. Save the Certificate to Environment:
```plaintext
[System.Environment]::SetEnvironmentVariable("CertificateThumbprint", $Certificate.Thumbprint, "User")
```

### Signing a Local PowerShell Script

1. Get the Certificate:
```plaintext
$Cert = Get-ChildItem -Path Cert:\CurrentUser\My | Where-Object {$_.Thumbprint -eq $env:CertificateThumbprint}
```

2. Sign the script:
```plaintext
Set-AuthenticodeSignature -FilePath "<script_path>" -Certificate $Cert
```
The output should indicate Valid signature.

---

## Functional Overview

- The user disables deprecated features, applies group policy restrictions, and generates a certificate for local script signing
- This results in a Windows system that only permits execution of verified scripts, mitigating script-based threats

---

## Challenges & Solutions

### Ensuring Compatibility with Windows 11 Home

- Use PolicyPlus to apply the required policy restrictions

### Certificate Management

- Generate a self-signed certificate and add it to the Trusted Root Certification Authorities store

---

## Lessons Learned

### Security Requires Proactive Planning

- Locking down script execution demands a strong understanding of PowerShell behavior, certificate trust chains, and administrative policy enforcement

### Usability and Security Must Coexist

- The balance between restricting harmful scripts and allowing legitimate development is achievable with the right configuration tools

---

## Future Enhancements

- Integrate a GUI-based installer for easier configuration
- Add auto-detection scripts for unsigned scripts and prompt signing
- Extend compatibility to cover Windows Server configurations
