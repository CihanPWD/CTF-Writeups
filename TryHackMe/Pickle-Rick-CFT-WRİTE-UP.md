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

