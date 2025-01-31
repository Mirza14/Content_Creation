# Investigation Scenario

- Rob, a SOC Engineer, implemented a protective firewall, patched vulnerabilities, and accessed endpoints to patch for security vulnerabilities. As he worked tirelessly, he left "breadcrumbs," small traces of his activity.
- Unaware of Rob's good intentions, the SOC team spotted anomalies: Logs showing admin access, escalation of privileges, patched systems behaving differently, and security tools triggering alerts. The SOC team misinterpreted the system modifications as a sign of an insider threat or rogue attacker and decided to launch an investigation using the Atomic Red Team framework.

## Learning Objectives

- Learn how to identify malicious techniques using the MITRE ATT&CK Framework.
- Learn how to use Atomic Red Team tests to conduct attack simulations.
- Understand how to create alerting and detection rules from the attack tests.

*Skills:*

- MITRE ATT&CK
- Unified Kill Chain
- Alert Creation/Detection Rule - Sigma
- Phishing Analysis
- Sysmon

### Detection Gaps

- Every blue teamer's dream is to be abe to detect every attack or step that in an attack kill chain. However, no matter what, there are always gaps in their detection.
- Gaps are usually for one of two main reasons: *Security is a cat-and-mouse game* - As we detect more, the threat actors and red teamers will find new sneaky ways to thwart our detection. We then need to study these novel techniques and update our signature and alert rules to detect these new techniques.
- *The line between anomalous and expected behaviour is often very fine and sometimes even has significant overlap* - For example, let's say we are a company based in the US. We expect to see almost all of our logins come from IP addresses in the US. One day, we get a login event from an IP in the EU, which would be an anomaly. However, it could also be our CEO travelling for business. This is an example where normal and malicious behaviour intertwine, making it hard to create accurate detection rules that would not have too much noise.

#### Cyber Attacks and the Kill Chain

- As a Cyber Defender professional, it would be be our dream to prevent all attacks at the start of the kill chain. So even just when threat actors start their reconnaissance, we already stop them dead in their tracks. But, as discussed before, this is not possible. The goal then shifts slightly. If we are unable to fully detect and prevent a threat actor at any one phase in the kill chain, the goal becomes to perform detections across the entire kill chain in such a way that even if there are detection gaps in a single phase, the gap is covered in a later phase. The goal is, therefore, to ensure we can detect the threat actor before the very last phase of goal execution.
![Check the image for your reference](/Blogs/Images/image38.png)

##### MITRE ATT&CK Framework

- A popular framework in the Cyber Security space for understanding the several tactics and techniques that threat actors/adversaries perform through the cyber kill chain in the MITRE framework. The framework is a collection of tactics, techniques, and procedures that have been seen to be implemented by real threat actors. ([MITRE ATT&CK](https://attack.mitre.org/))

###### Atomic Red Team

- The Atomic Red Team library is a collection of red team test cases that are mapped to the MITRE ATT&CK framework. The library consists of simple test cases that can be executed by any blue team to test for detection gaps and help close them down. The library also supports automation, where the techniques can be automatically executed. However, it is also possible to execute them manually.

###### Attack Simulation using Atomic Test

- SOC suspects that the supposed attacker used the MITRE ATT&CK technique ([T1566.001 Spearphishing](https://attack.mitre.org/techniques/T1566/001/)) with an attachment. Let's recreate the attack emulation performed by the supposed attacker and then look for the artefacts created.
- 1st Step: Open up a PowerShell prompt as administrator and follow along with us. Let's start by having a quick peek at the help page. Enter the command `Get-Help Invoke-Atomictest`. You should see the output below:
![Check the image for your reference](/Blogs/Images/image39.png)

| Parameter | Explanation | Example Use |
| --------- | ----- | ------- |
| -Atomic Technique | This defines what technique you want to emulate. You can use the complete technique name or the "TXXXX" value. This flag can be omitted. | Invoke-AtomicTest -AtomicTechnique T1566.001  |
| -ShowDetails | Shows the details of each test included in the Atomic. |  Invoke-AtomicTest T1566.001 -ShowDetails   |
| -ShowDetailsBrief | Shows the title of each test included in the Atomic. |  Invoke-AtomicTest T1566.001 -ShowDetailsBrief  |
|  -CheckPrereqs | Provides a check if all necessary components are present for testing |  Invoke-AtomicTest T1566.001 -CheckPrereqs|
| -TestNames  | Sets the tests you want to execute using the complete Atomic Test Name. |  Invoke-AtomicTest T1566.001 -TestNames "Download Macro-Enabled Phishing Attachment" |
| -TestGuids  | Sets the tests you want to execute using the unique test identifier. |  Invoke-AtomicTest T1566.001 -TestGuids 114ccff9-ae6d-4547-9ead-4cd69f687306 |
| -TestNumbers  | Sets the tests you want to execute using the test number. The scope is limited to the Atomic Technique. |  Invoke-AtomicTest T1566.001 -TestNumbers 2,3 |
| -Cleanup  | Run the cleanup commands that were configured to revert your machine state to normal. |  Invoke-AtomicTest T1566.001 -TestNumbers 2 -Cleanup |

- 2nd Step: Run the command.
![Check the image for your reference](/Blogs/Images/image40.png)
- Let's examine what type of information is provided in a test. We will use the test we want to run as an example.

| Key | Value | Description |
| --------- | ----- | ------- |
| Technique | Phishing: Spearphishing Attachment T1566.001 | The full name of the MITRE ATT&CK technique that will be tested |
| Atomic Test Name | Download Macro-Enabled Phishing Attachment | A descriptive name of the type of test that will be executed |
| Atomic Test Number | 1 | A number is assigned to the test; we can use this in the command to specify which test we want to run. |
| Atomic Test GUID | 114ccff9-ae6d-4547-9ead-4cd69f687306 | A unique ID is assigned to this test; we can use this in the command to specify which test we want to run. |
| Description | This atomic test downloads a macro-enabled document from the Atomic Red Team GitHub repository, simulating an end-user clicking a phishing link to download the file. The file "PhishingAttachment.xlsm" is downloaded to the %temp% | Provides a detailed explanation of what the test will do. |
| Attack commands | *Executor:* powershell *ElevationRequired:* False *Command:* $url = ‘http://localhost/PhishingAttachment.xlsm’ Invoke-WebRequest -Uri $url -OutFile $env:TEMP.xlsm | This provides an overview of all the commands run during the test, including the executor of those commands and the required privileges. It also helps us determine where to look for artefacts in Windows Event Viewer. |
| Cleanup commands | Command: Remove-Item $env:TEMP.xlsm -ErrorAction Ignore | An overview of the commands executed to revert the machine back to its original state. |
| Dependencies | There are no dependencies required. | An overview of all required resources that must be present on the testing machine in order to execute the test |

- 3rd Step: Before running the emulation, we should ensure that all required resources are in place to conduct it successfully.
![Check the image for your reference](/Blogs/Images/image41.png)

- 4th Step: Run the Atomic Test to emulate the attack. **Phishing: Spearphishing Attachment T1566.001 Emulated**
![Check the image for your reference](/Blogs/Images/image42.png)
- Based on the output, we can determine that the test was successfully executed. We can now analyse the logs in the Windows Event Viewer to find Indicators of Attack and Compromise.

- 5th Step: Clear the Sysmon logs and cleanup the Atomic test, so that we can look for log entries that point us to this emulated attack. `Windows Event Viewer => Applications and Services => Microsoft => Windows => Sysmon => Operational`
![Check the image for your reference](/Blogs/Images/image43.png)

###### Detecting the Atomic Test

- Now that we have cleaned up the files and the sysmon logs, let us run the emulation again by issuing the command
![Check the image for your reference](/Blogs/Images/image42.png)
- *Sysmon Log Analysis:* Found the **File Creation Event** with the process ID *1652*
![Check the image for your reference](/Blogs/Images/image44.png)
- *Sysmon Log Analysis:* Found the **Process Creation Event** with the parent process ID of *1652*
![Check the image for your reference](/Blogs/Images/image45.png)

###### Sigma Alerting on Atomic Test T1566.001 Spearphishing Attachment

- Check the `PowerShell_Invoke_WebRequest.yml` file under the *TryHackMe* directory.

###### Artefacts from the Log Analysis

1. Command Line: `"powershell.exe" & {$url = 'http://localhost/PhishingAttachment.xlsm' Invoke-WebRequest -Uri $url -OutFile $env:TEMP\PhishingAttachment.xlsm}"`
2. `Invoke-WebRequest:` It is not common for this command to run from a script behind the scenes.
3. `$url = 'http://localhost/PhishingAttachment.xlsm'`: Attackers often use a specific malicious domain to host their payloads. Including the malicious URL in the Sigma rule could help us detect that specific URL.
4. `PhishingAttachment.xlsm:` This is the malicious payload downloaded and saved on our system. We can include its name in the Sigma rule as well.

###### Investigation Challenege

1. What was the flag found in the .txt file that is found in the same directory as the PhishingAttachment.xslm artefact?
![Check the image for your reference](/Blogs/Images/image46.png)

2. What ATT&CK technique ID would be our point of interest?
`T1059` - ([Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059/))

3. What ATT&CK subtechnique ID focuses on the Windows Command Shell?
`T1059.003` - ([Command and Scripting Interpreter: Windows Command Shell](https://attack.mitre.org/techniques/T1059/003/))

4. What is the name of the Atomic Test to be simulated?
![Check the image for your reference](/Blogs/Images/image47.png)

5. What is the name of the file used in the test?
![Check the image for your reference](/Blogs/Images/image48.png)

6. What is the flag found from this Atomic Test?
![Check the image for your reference](/Blogs/Images/image49.png)
