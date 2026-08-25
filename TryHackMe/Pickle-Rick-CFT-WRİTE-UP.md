# Pickle Rick — CTF Write-Up

## 1. Introduction

This write-up documents the penetration testing and exploitation process performed against the **Pickle Rick** machine in the TryHackMe platform.

The objective of this assessment was to identify vulnerabilities within the target environment, understand how the identified weaknesses could be exploited, and ultimately obtain the flags provided by the challenge.

The assessment was conducted from an attacker's perspective, following a structured penetration testing methodology. The process included **reconnaissance, enumeration, vulnerability identification, exploitation, privilege escalation, and post-exploitation activities**.

Throughout the assessment, relevant findings are documented with technical explanations and supporting evidence, including screenshots and command outputs. The purpose of this write-up is not only to demonstrate how the machine was compromised, but also to explain **why each step was performed, what was discovered, and how the discovered vulnerabilities contributed to the overall attack path**.

> **Scope:** This assessment was performed exclusively against the authorized TryHackMe CTF environment.

## 2. Reconnaissance

The first stage of the assessment was reconnaissance. The objective was to identify the target's exposed network services, determine the technologies in use, and identify potential attack surfaces for further enumeration.

### 2.1 Nmap Scan

Nmap was used to perform service and version detection against the target.

The following command was executed:

```bash
nmap 10.113.142.240 -sV -sC -Pn
```
<img width="1266" height="496" alt="nmap çıktısı" src="https://github.com/user-attachments/assets/0260d2fb-9fb6-450b-8565-ac66d72a0e24" />

## 3. Web Enumeration

The Nmap scan identified an HTTP service running on port 80. Since the SSH service on port 22 was exposed but no valid credentials or other information related to SSH access had been identified at this stage, the web application was selected as the primary attack surface for further enumeration.

I accessed the web application through the browser to inspect its publicly available content and identify any information that could assist with further enumeration or exploitation.

<img width="1916" height="762" alt="cft" src="https://github.com/user-attachments/assets/a512cdbd-2ea2-4c96-b983-8708a675fd66" />


The application displayed a page titled **"Help Morty!"** and contained a message requesting assistance with finding three secret ingredients required to complete a pickle-reverse potion.

The page also indicated that the user needed to log in to the target system, but the password was not known at this stage.

From an assessment perspective, the web application became the primary focus because it was directly accessible over HTTP and potentially contained information that could lead to further access to the underlying system.

## 4. Source Code Inspection

After reviewing the web application's visible content, I inspected the page's HTML source code to identify any information that was not directly displayed to the user.

During the source code inspection, a username was discovered within the page source.

<img width="1905" height="756" alt="username" src="https://github.com/user-attachments/assets/54e38c2e-46b9-4e54-a811-c307b5ce0393" />


The discovered username was recorded for further investigation, as it could potentially be useful during later stages of the assessment, particularly if an authentication mechanism or another service requiring user credentials was identified.

At this stage, the username alone did not provide direct access to the system. However, it represented potentially useful information disclosed by the application and was therefore retained as part of the enumeration process.

## 5. Directory and File Enumeration

To expand the attack surface identified during the initial web enumeration, directory and file enumeration was performed against the web server using Gobuster.

The purpose of this step was to identify potentially interesting files and directories that were not directly linked from the main page:

<img width="647" height="348" alt="image_750x_6487466bbe481" src="https://github.com/user-attachments/assets/77a534f3-eefb-4f55-88fa-2bf253dd9c9f" />-
```bash
gobuster dir -u http://10.10.146.14/ -w usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -X php
```
## 6. Authentication Testing

Following the discovery of ^login.php^ endpoint, the authentication mechanism was investigated to determine whether valid credentials could be identified.

<img width="1886" height="828" alt="login sayfasi" src="https://github.com/user-attachments/assets/2956a9c9-ed35-4263-97f0-f6fcffd3b11a" />


### 6.1 Initial Credential Testing

The login functionality was first tested using common/default credentials to determine whether the application was using weak or default authentication credentials.

The attempted credentials were unsuccessful, indicating that the application was not immediately accessible through the tested default credentials.

### 6.2 Previously Discovered Username

During the earlier source code inspection, a username had been identified within the application's HTML source code.

The previously discovered username was therefore tested against the login functionality.

Although the username appeared to be valid, a corresponding password was still required to authenticate successfully.

This established an important piece of information for the next stage of the assessment:

- **Username:** Discovered during source code inspection
- **Password:** Unknown
- **Authentication:** Not yet bypassed

At this point, the assessment required further enumeration to identify information that could potentially reveal or assist in obtaining the missing password.

## 7. Information Disclosure via `robots.txt`

During the previous enumeration phase, the `robots.txt` file was identified as an accessible resource. The file was therefore inspected for potentially interesting paths or information.

<img width="1908" height="863" alt="robotstxt" src="https://github.com/user-attachments/assets/3f9d0d40-b203-4c42-b561-6de6a5f19250" />

The file contained a seemingly random string that did not correspond to an obvious directory or filename.

At this point, the string was documented as potentially sensitive information and considered as a possible credential candidate.

### 7.1 Credential Validation

Since a valid username had already been identified during the source code inspection, the discovered string was tested as the password for that account.

This demonstrated that sensitive authentication information had been exposed through a publicly accessible `robots.txt` file.

Following successful authentication, an authenticated user panel became available. A comment within the panel provided additional information that could be useful for the next stage of the assessment.

<img width="1675" height="731" alt="commend line" src="https://github.com/user-attachments/assets/70cc7e9c-68cf-40cd-a536-8117440690e1" />


### 7.2 Finding — Sensitive Information Disclosure

**Description:**
Sensitive information that could be used as an authentication credential was exposed through a publicly accessible `robots.txt` file.

**Impact:**
An unauthenticated attacker capable of discovering this information could potentially authenticate to the application without legitimately obtaining the user's password.

**Recommendation:**

- Do not store passwords, tokens, or other authentication secrets in publicly accessible files.
- Treat `robots.txt` as a navigation mechanism rather than a security control.
- Review publicly accessible resources for sensitive information disclosure.
- Credentials should be stored securely and should never be exposed in application resources or source files.

## 8 Initial Command Execution
After obtaining authenticated access to the application, I identified an input field that appeared to process system commands.
As an initial test, I submitted a basic operating system command through the available input field.
<img width="1890" height="773" alt="ls" src="https://github.com/user-attachments/assets/397135be-85bf-4c92-ad79-ce3e9f68ccca" />
The command was successfully executed and the response returned a list of files located on the underlying server.
This confirmed that the application was capable of processing user-controlled input as operating system commands.

### 8.1 File Access Testing
Following the initial command execution, I attempted to read the contents of the discovered files.
<img width="1664" height="561" alt="no" src="https://github.com/user-attachments/assets/a5018441-eff5-4e93-83c9-1c2a9f279daf" />
Although the command was executed, the contents of the targeted files could not be successfully retrieved.
This indicated that command execution was possible, while access to certain files was restricted by the current execution context or file permissions.

## 9. OS Command Injection

After confirming that the application was executing user-controlled commands, I investigated whether the existing command could be manipulated to execute an additional command.

### 9.1 Reverse Shell Payload Preparation

To demonstrate the impact of the command injection vulnerability, I prepared a PHP-based reverse shell payload using a reverse-shell payload generation resource.

<img width="1766" height="929" alt="rce genarator" src="https://github.com/user-attachments/assets/38ed3e46-4416-4080-b1a8-47ea2c8584d2" />

The generated payload was prepared to establish an outbound connection to the assessment machine.

### 9.2 Command Injection and Reverse Shell

The vulnerable input was then used to append the reverse shell command to the application's existing command execution flow.

<img width="1916" height="838" alt="nc dinleyici" src="https://github.com/user-attachments/assets/e6a3c564-4373-43ec-ac1c-0c79ce02e954" />

The command was successfully processed by the target, and a reverse shell connection was established on the assessment machine.

This confirmed that the previously identified command injection vulnerability could be leveraged to obtain interactive access to the underlying server.

### 9.3 Finding Summary

**Finding:** OS Command Injection  
**Severity:** Critical  
**Impact:** Arbitrary command execution and remote shell access

The vulnerability allows attacker-controlled input to influence commands executed by the underlying operating system. In this assessment, successful exploitation resulted in a reverse shell, demonstrating that the issue could lead to full compromise of the application host depending on the privileges of the compromised process.

## 10. Post-Exploitation Enumeration

After successfully establishing a reverse shell, the next step was to perform basic enumeration of the compromised environment.

### 10.1 Current User and Privileges

The `id` command was executed to determine the user associated with the current shell session and identify the available privileges.

<img width="891" height="430" alt="id" src="https://github.com/user-attachments/assets/e91338c9-62aa-4a43-b635-7deb2cbc91d6" />

The output showed that the shell was running with limited privileges rather than with administrative or root-level access.

This confirmed that the initial shell did not provide privileged access to the system. Further enumeration was therefore required to identify relevant files, clues, or potential privilege escalation opportunities.

### 10.2 Initial CTF Ingredient

During the initial enumeration, the first required CTF ingredient was discovered: **meeseek hair**.

The discovered item was recorded as the first required ingredient for completing the challenge.

### 10.3 Reading `clue.txt`

Further investigation identified a file named `clue.txt`. The contents of the file were examined to determine the next step of the challenge.

The file contained the following instruction:

> **Look around the file system for the other ingredient.**

This indicated that another required ingredient was located somewhere within the filesystem.

Based on this clue, the next stage of the assessment focused on further filesystem enumeration to locate the remaining ingredient.

## 10.4 Filesystem Enumeration

Following the instruction found in `clue.txt`, further enumeration of the filesystem was performed to locate the remaining ingredient.

The filesystem was explored by examining accessible directories and their contents. During this process, the `/home/rick` directory was identified as containing the second required ingredient.

The contents of the files within the directory were then inspected using the `cat *` command.

<img width="360" height="265" alt="2 ci" src="https://github.com/user-attachments/assets/16e7ebba-6437-414d-9000-4024b904c02a" />

The command successfully displayed the contents of the accessible files, revealing the second required CTF ingredient.

This confirmed that the clue obtained earlier could be followed through filesystem enumeration and that the current shell privileges were sufficient to access the relevant files under `/home/rick`.

At this stage, two of the required ingredients had been identified, and the enumeration continued to locate the remaining objectives.

## 11. Privilege Escalation

After obtaining a low-privileged reverse shell, the next objective was to access the `/root` directory and retrieve the final CTF flag.

Access to the `/root` directory was initially restricted because the current shell did not have sufficient privileges. Therefore, further enumeration was performed to identify potential local privilege escalation opportunities.

### 11.1 Sudo Privilege Enumeration

The `sudo -l` command was executed to enumerate the commands that the current user was authorized to execute with elevated privileges.

<img width="947" height="249" alt="yetki yükseltme" src="https://github.com/user-attachments/assets/ac38cfc1-6054-48bd-8aaa-caac87ede584" />

The output revealed that the current user was permitted to execute `/bin/sh` through `sudo` with elevated privileges.

This configuration represented a potential privilege escalation path because the permitted shell could be executed with the privileges of the `root` user.

### 11.2 Exploiting the Sudo Misconfiguration

The identified sudo configuration was then investigated using the corresponding technique documented by GTFOBins.

<img width="1389" height="816" alt="shell" src="https://github.com/user-attachments/assets/4d90def3-82bb-473d-bc50-b2b73be19424" />

Based on the documented technique, the permitted shell was executed through `sudo` using:

```bash id="j4yq7k"
sudo /bin/sh
```

The command executed successfully and resulted in a shell with **root-level privileges**.

This confirmed that the identified sudo configuration could be abused to escalate privileges from the initial low-privileged account to `root`.

### 11.3 Accessing the Final Flag

After obtaining root-level access, the previously restricted `/root` directory became accessible.

The directory was enumerated and the final CTF flag was successfully located.

<img width="602" height="107" alt="3rd" src="https://github.com/user-attachments/assets/62183ba7-9d87-4422-908d-6d2b3c932807" />

The successful retrieval of the final flag confirmed that the privilege escalation was successful and that full administrative access to the target system had been obtained.

### 11.4 Final Result

The complete exploitation chain identified throughout the assessment was:

**Network Reconnaissance → Web Application Enumeration → Source Code Inspection → Username Discovery → Directory and File Enumeration → Authentication Endpoint Discovery → `robots.txt` Information Disclosure → Credential Discovery and Validation → Authenticated Application Access → OS Command Execution → OS Command Injection → Reverse Shell → Post-Exploitation Enumeration → CTF Ingredient Discovery → Sudo Enumeration → Sudo Misconfiguration Identified → Privilege Escalation → Root Shell → Final Flag**

The successful exploitation of the identified weaknesses demonstrated that the vulnerabilities could be chained together to progress from initial reconnaissance and unauthenticated web access to authenticated application access, arbitrary operating system command execution, interactive shell access, and ultimately root-level privileges.

The successful escalation from the initial web application compromise to `root` demonstrated that the identified vulnerabilities could be leveraged to achieve complete compromise of the target host.

All three required CTF ingredients were successfully identified, and the final flag was recovered from the `/root` directory, completing the assessment within the authorized TryHackMe CTF environment.

