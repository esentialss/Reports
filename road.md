# Penetration Testing Report

## TryHackMe - Road

**Assessment Type:** Web Application and Linux Host Penetration Test

**Platform:** TryHackMe

**Target:** Road

**Difficulty:** Medium

**Operating System:** Linux

**Assessment Status:** Completed

**Report Classification:** Technical Security Assessment - Lab Environment

## 1. Executive Summary

This report documents a penetration test conducted against the Road machine hosted on the TryHackMe platform. The assessment focused on identifying exposed services, enumerating the web application, exploiting security vulnerabilities, obtaining initial access, and escalating privileges to the root user.

The assessment identified several weaknesses that, when chained together, enabled complete compromise of the target system. The initial attack vector was an Insecure Direct Object Reference (IDOR) vulnerability in the web application's password reset functionality. This allowed an authenticated user to reset another user's password, including the administrator's account.

Following administrative account compromise, an unrestricted file upload vulnerability enabled the execution of a PHP reverse shell, providing initial access to the underlying Linux host. Further enumeration revealed a locally accessible MongoDB instance containing a backup of user data, from which credentials for the *webdeveloper* account were recovered.

Finally, a misconfiguration involving a privileged backup utility and the LD_PRELOAD environment variable allowed the *webdeveloper* account to escalate privileges to root.

The attack chain demonstrates how weaknesses in application-level authorization, file upload validation, credential storage, and privilege management can combine to compromise an entire system.

**Overall assessment:** The target was fully compromised within the authorized lab environment, with both user-level and root-level accesss obtained.

# 2. Scope and Objectives

## 2.1 Scope

The assessment was limited to the TryHackMe Road machine.

**Attribute** | **Details**

Target | Road

Target IP | TARGET-IP

Operating system | Linux

Exposed services | SSH, HTTP

Application path | /v2/

Database service | MongoDB on localhost

Assessment environment | Authorized CTF lab

## 2.2 Objectives

The primary objectives were to:

* Identify exposed network services and potential attack vectors.

* Enumerate the web application and assess its authentication and authorization controls.

* Identify and exploit vulnerabilities that could provide initial access.

* Investigate local services and recover potentially exposed credentials.

* Evaluate privilege escalation opportunities.

* Demonstrate the impact of the identified weaknesses using proof-of-compromise evidence.

# 3. Methodology 

The assessment followed a structured penetration testing workflow consisting of the following phases:

1. **Reconnaissance:** Identification of open ports and running services.

2. **Enumeration:** Discovery of web application endpoints and analysis of HTTP requests.

3. **Exploitation:** Abuse of the password reset authorization flaw and file upload functionality.

4. **Post-exploitation:** Enumeration of the compromised host and collection of the user flag.

5. **Credential discovery:** Examination of locally accessible MongoDB data.

6. **Privilege escalation:** Abuse of the privileged backup utility to obtain root access.

7. **Verification:** Confirmation of elevated privileges and retrieval of the root flag.

**Tool Used**

* Nmap

* Gobuster

* Burp Suite

* Netcat

* MongoDB client utilities

* SSH

* GCC

* Linux command-line utilities

# 4. Reconnaissance and Service Enumeration

## 4.1 Port Scanning

An initial Nmap scan was conducted to identify open ports, service versions, and common service-related information.

**Command:** nmap -sV -sC -p- TARGET-IP -oN road.txt

**Results:**

**Port** | **Service** | **Status**

22/tcp | SSH | Open

80/tcp | HTTP | Open

The scan identified two externally accessible services: SSH and HTTP.

SSH was initially deferred because valid credentials had not yet obtained. The HTTP service was selected as the primary attack surface for further investigation.

**Evidence:** Nmap scan output.

<img width="935" height="419" alt="{DFDF2551-702A-4442-8F1B-1E9704C3F22C}" src="https://github.com/user-attachments/assets/6b566123-6a32-4672-a30a-f12b88d5ad3a" />

# 5. Web Application Enumeration

## 5.1 Directory and Endpoint Discovery

The web application was initially inspected manually, including its page source, cookies, network activity, and available functionality. This did not immediately reveal an exploitable weakness.

Directory enumeration was subsequently performed using Gobuster. 

**Command:** gobuster dir -u http://TARGET-IP -w WORDLIST

**Discovered paths:**

* /assets

* /v2

The */assets* directory did not reveal any useful information. Accessing */v2* redirected the browser to the administrative login page: http://TARGET-IP/v2/admin/login.html

Attempts to authenticate using common default credentials were unsuccessful.

## 5.2 Account Registration and Password Reset Analysis

The application provided a registration feature, allowing a new user account to be created. After authentication, the *ResetUser* functionality was identified in the application's navigation menu.

Testing the password reset feature revealed that the corresponding HTTP request contained the target user's email address and the new password.

The request was captured in Burp Suite and forwarded to Repeater for controlled testing.

## 5.3 Insecure Direct Object Reference (IDOR)

To determine whether the password reset function enforced appropriate authorization, a second test account was created.

The captured request was modified by replacing the email address associated with the authenticated account with the email address of the second account.

The modified request successfully reset the second account's password, despite the operation being initiated from the first account's authenticatd session.

This demonstrated that the application relied on a client-controlled identifier without adequately verifying whether the authenticated user was authorized to modify the specified account.

**Finding:** Insecure Direct Object Reference (IDOR) in the password reset functionality.

**Security impact:**

* Unauthorized password changes for other users.

* Potential account takeover.

* Escalation of impact if privileged accounts are affected.

<img width="739" height="502" alt="{FD28883F-1B1B-446E-AD71-F54F0433C8F6}" src="https://github.com/user-attachments/assets/c23a534d-4823-4f81-8d34-205f061adedb" />

# 6. Initial Access

## 6.1 Administrator Account Compromise

Further inspection of the application revealed a profile page accessible through the username menu in the upper-right corner of the interface.

A note near the bottom of the profile page disclosed the administrator's email address: *admin@sky.thm*

