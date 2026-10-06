# Lab 1: JWT Token Vulnerability

## Overview
- **Vulnerability:** Information Disclosure & Token Tampering
- **OWASP Category:** A07:2021 – Identification and Authentication Failures
- **Severity:** HIGH (CVSS 7.5)
- **Platform:** OWASP Juice Shop
- **Date Completed:** October 1-2, 2026

## What This Lab Demonstrates
- JWT token payload exposure (Base64 encoded, not encrypted)
- Sensitive user data readable in plaintext
- Token manipulation detection
- Lack of encryption on authentication tokens

## Key Findings
✓ JWT payload contains: user ID, email, password hash, role  
✓ Token decoded easily using jwt.io  
✓ Server detects tampering via signature validation  
✓ Payload exposure remains critical security risk  

## Files in This Folder
- `Lab_1_JWT_Token_Vulnerability.md` - Full write-up with screenshots
- `screenshots/` - Annotated screenshots of exploitation

## Quick Summary
The Juice Shop stores user tokens as JWT in cookies. While the JWT signature prevents tampering, the payload is only Base64-encoded (not encrypted), making sensitive user information readable to anyone with the token.

## How to Use
1. Read `Lab_1_JWT_Token_Vulnerability.md` for complete details
2. View screenshots for step-by-step exploitation
3. Reference remediation section for secure implementation

## Tools Used
- Burp Suite (Proxy, Repeater, HTTP History)
- jwt.io (JWT decoder)
- Browser Developer Tools

## CVSS Score
**7.5** (HIGH) - Information Disclosure

---
**Status:** ✅ COMPLETE  
**Next Lab:** Lab 2 - Insecure Data Storage & Weak Cryptography
