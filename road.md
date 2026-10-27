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

