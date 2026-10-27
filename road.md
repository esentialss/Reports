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

