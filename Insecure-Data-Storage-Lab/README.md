# Lab 2: Insecure Data Storage & Weak Cryptography

## Overview
- **Vulnerabilities:** Multiple
  - A02:2021 – Cryptographic Failures
  - A05:2021 – Security Misconfiguration
- **Severity:** CRITICAL (CVSS 9.1)
- **Platform:** OWASP Juice Shop
- **Date Completed:** October 2-3, 2026

## What This Lab Demonstrates
- Credit card data stored unencrypted
- Passwords hashed with broken MD5 algorithm
- Hash cracking via online databases
- PCI-DSS compliance violations

## Key Findings
✓ Credit card data exposed via /api/Cards endpoint  
✓ Card numbers, expiry dates, CVV accessible  
✓ Passwords hashed with MD5 (cryptographically broken)  
✓ MD5 hash cracked in <1 second  
✓ Online rainbow tables used for cracking  

## Quick Summary
Two critical vulnerabilities:
1. **Credit Card Exposure:** Sensitive payment data transmitted and stored without encryption
2. **Weak Hashing:** Passwords hashed with MD5 instead of bcrypt/Argon2

## Findings
- Credit card cracking: Trivial (just database lookup)
- Password cracking: MD5 hash `824682b7c01d5c745bb87d6b1b4308c1` → `12345` (instant)

## Tools Used
- Burp Suite (Proxy, HTTP History, Repeater)
- md5decrypt.net (Hash cracking)
- jwt.io (JWT decoder)

## CVSS Scores
- **Credit Card Exposure:** 9.1 (CRITICAL)
- **Weak Password Hashing:** 7.5 (HIGH)

## Compliance Impact
- ❌ PCI-DSS Requirement 3.4 (Card encryption)
- ❌ OWASP Cryptographic Failures
- ❌ NIST Password Guidelines

---
**Status:** ✅ COMPLETE  
**Next Lab:** Lab 3 - Broken Access Control
