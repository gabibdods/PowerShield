<h1>Windows 11 Security Hardening</h1>

<h2>Problem Statement</h2>
<p>
  Apply strict security patches to Windows 11 and configure the system to run scripts under stringent restrictions to harden the operating system environment against invaders and hackers, while preserving user and developer flexibility.
</p>

<h2>Goal</h2>
<ul>
  <li>Enhance Windows 11 security posture to the highest practical level.</li>
  <li>Maintain equivalent operational freedom for legitimate users and programmers.</li>
</ul>

<h2>Tools</h2>
<ul>
  <li><strong>PowerShell</strong> with elevated (Administrator) permissions.</li>
  <li><strong>Code signing script</strong> for authenticating and restricting script execution (used in Parts 2 and 3 of the tutorial).</li>
  <li><strong>PolicyPlus</strong> to modify the local group policies, since it is not natively available for Windows 11 Home edition.</li>
</ul>

<h2>Tutorial</h2>

<h3>Applying Security Patches for Windows 11</h3>
<ol>
  <li>
    Go to <strong>Turn Windows features on or off</strong> and disable <strong>Windows PowerShell 2.0</strong>.  
    After a reboot, PowerShell 2.0 should be unavailable, for the better.
  </li>
  <li>
    Open <strong>Edit the system environment variables</strong>, click <strong>Environment Variables</strong>, add a new system variable named <code>__PSLockDownPolicy</code> with value <code>4</code>.  
    Verify by running in PowerShell:
    <pre><code>$ExecutionContext.SessionState.LanguageMode</code></pre>
    It should return <code>ConstrainedLanguage</code> instead of <code>FullLanguage</code>.
  </li>
  <li>
    Download <a href="https://github.com/Fleex255/PolicyPlus">PolicyPlus</a>. Launch it, navigate to <strong>Windows Components &gt; Windows PowerShell</strong>, set <strong>Turn on Script Execution</strong> to <strong>Disabled</strong>, then save policies in the <strong>File</strong> tab. If you exit without saving the policies, change won't be applied!
  </li>
  <li>
    Download the latest PowerShell 7 from <a href="https://github.com/PowerShell/PowerShell">PowerShell GitHub</a>. Unzip the download, copy <code>PowerShellCoreExecutionPolicy.admx</code> to <code>C:/Windows/PolicyDefinitions</code> and <code>PowerShellCoreExecutionPolicy.adml</code> to <code>C:/Windows/PolicyDefinitions/en-US</code> and all the other language folders on the same directory level.
  </li>
  <li>
    In PolicyPlus, go to <strong>PowerShell Core &gt; Windows PowerShell</strong>, disable <strong>Turn on Script Execution</strong>, and save.
  </li>
</ol>

<h3>Creating a Code Signing Certificate</h3>
<ol>
  <li>Open PowerShell with Administrator privileges.</li>
  <li>
    Run:
    <pre><code>$CertificateName = "PSCS"</code></pre>
  </li>
  <li>
    Run:
    <pre><code>$Certificate = New-SelfSignedCertificate -CertStoreLocation Cert:\CurrentUser\My -Subject "CN=$CertificateName" -KeySpec Signature -Type CodeSigningCert</code></pre>
  </li>
  <li>
    Run:
    <pre><code>$CertificatePath = [Environment]::GetFolderPath("MyDocuments")+"\$CertificateName.cer"</code></pre>
  </li>
  <li>
    Run:
    <pre><code>Export-Certificate -Cert $Certificate -FilePath $CertificatePath</code></pre>
  </li>
  <li>
    Import to Trusted Root:
    <pre><code>Get-Item $CertificatePath | Import-Certificate -CertStoreLocation "Cert:\LocalMachine\Root"</code></pre>
  </li>
  <li>
    Display thumbprint:
    <pre><code>Write-host "Certificate Thumbprint:" $Certificate.Thumbprint</code></pre>
  </li>
</ol>

<h3>Signing a Local PowerShell Script</h3>
<ol>
  <li>
    Set variables:
    <pre><code>$CertificateThumbprint = "&lt;CertificateThumbprint&gt;"
$ScriptPath = "&lt;ScriptPath&gt;"</code></pre>
  </li>
  <li>
    Retrieve the certificate:
    <pre><code>$CodeSignCert = Get-ChildItem -Path Cert:\CurrentUser\My | Where-Object {$_.Thumbprint -eq $CertificateThumbprint}</code></pre>
  </li>
  <li>
    Sign the script:
    <pre><code>$Set-AuthenticodeSignature -FilePath $ScriptPath -Certificate $CodeSignCert</code></pre>
  </li>
  <li>Confirm status: The output should indicate <strong>Valid</strong> signature.</li>
</ol>

<h2>Design Choices</h2>
<ul>
  <li>Disable default settings that do not meet advanced security standards (e.g., default execution policies, telemetry services, unnecessary network protocols).</li>
  <li>Enforce strict execution policies (<code>AllSigned</code> or <code>Restricted</code>) in PowerShell.</li>
  <li>Enable and configure Windows Defender Antivirus and Microsoft Defender Firewall with customized rules.</li>
  <li>Apply the latest Windows 11 cumulative updates and security patches.</li>
</ul>

<h2>Challenges</h2>
<ul>
  <li>Researching and verifying the latest vulnerabilities and security best practices for Windows 11.</li>
  <li>Balancing stringent security configurations with usability requirements.</li>
  <li>Ensuring that signed scripts are recognized and permitted without hindering legitimate automation.</li>
</ul>

<h2>Solution</h2>
<p>
  Leverage community forums, Microsoft documentation, and security-focused blogs to:
</p>
<ol>
  <li>Identify critical patches and configuration recommendations.</li>
  <li>Develop and test PowerShell scripts for automated hardening and code signing.</li>
  <li>Share findings and scripts back to the community for peer review and improvement.</li>
</ol>

<h2>Lessons Learned</h2>
<ul>
  <li>Achieving maximal security on Windows 11 requires deep knowledge of cyber-security concepts.</li>
  <li>Users and administrators must stay informed about emerging vulnerabilities and patch releases.</li>
  <li>Community collaboration is essential for maintaining a hardened Windows 11 environment without sacrificing functionality.</li>
</ul>
