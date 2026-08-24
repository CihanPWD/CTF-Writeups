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

