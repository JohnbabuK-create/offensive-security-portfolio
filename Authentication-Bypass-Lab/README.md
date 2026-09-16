# Authentication Vulnerabilities Assessment - Week 2 

  

**Date:** 16th September 2026  

**Platform:** PortSwigger Web Security Academy   

**Severity:** 🔴 High   

**Status:** ✅ Completed & Documented 

  

--- 

  

## Executive Summary 

  

This week I focused on three critical authentication vulnerabilities. Through hands-on labs and Burp Suite exploitation, I successfully: 

1. Identified users via response analysis 

2. Bypassed 2FA authentication   

3. Exploited broken password reset logic 

  

All three represent real-world vulnerabilities that lead to account compromise. 

  

--- 

  

## Authentication Vulnerabilities Overview 

  

Authentication is the security layer that verifies identity. When broken, attackers can: 

- ✅ Access any user account 

- ✅ Steal personal data 

- ✅ Perform actions as other users 

- ✅ Escalate to admin accounts 

  

**Common authentication flaws:** 

1. Username enumeration (revealing valid users) 

2. Multi-factor authentication bypass 

3. Broken password reset logic 

4. Weak session management 

  

--- 

  

## Lab 1: Username Enumeration via Different Responses 

  

### Vulnerability Description 

The application reveals valid usernames through different response lengths/messages when users don't exist vs. when they do. 

  

### How I Exploited It 

  

**Step 1: Reconnaissance** 

- Tried login with random username → Got "Invalid user" message 

- Tried login with real username → Got "Invalid password" message 

- Different responses = valid username detection possible 

  

**Step 2: Burp Suite Intruder Setup** 

1. Intercepted login request in Burp Proxy 

2. Sent request to Intruder tab 

3. Set username field as payload position 

4. Added payload list: [all provided usernames] 

5. Set attack type: Sniper 

6. Ran the attack 

  

**Step 3: Analysis** 

- Examined response lengths 

- Found one response with DIFFERENT length 

- That length = valid username identified 

- Other responses = invalid usernames 

  

**Step 4: Exploitation** 

- Extracted valid username from results 

- Tried common passwords 

- Successfully logged in 


### Technical Explanation 

  

The server's error messages reveal information: 

"Invalid user" → User doesn't exist (short response) 

"Invalid password" → User exists but wrong password (longer response) 

  

Attacker can: 

1. Test thousands of usernames automatically 

2. Identify valid accounts 

3. Focus brute force attacks on real accounts 

  

### Real-World Impact 

  

**Username enumeration enables:** 

- ✅ Targeted brute force attacks (only real users) 

- ✅ Account takeover 

- ✅ Phishing/social engineering (know who to target) 

- ✅ Privacy violation (reveal user database) 

  

--- 

  

## Lab 2: 2FA Simple Bypass 

  

### Vulnerability Description 

Multi-factor authentication (2FA) via email can be bypassed by accessing protected pages directly without completing 2FA verification. 

  

### How I Exploited It 

  

**Step 1: Initial Login** 

- Logged in with valid credentials 

- Redirected to 2FA email verification page 

- Requested email code 

  

**Step 2: Access Control Flaw Discovery** 

- Noticed URL pattern: `/email-verification` 

- Attempted direct navigation to: `/my-account` 

- Access granted WITHOUT entering email code! 

  

**Step 3: Successful Bypass** 

- Directly accessed protected account features 

- 2FA completely bypassed 

- Successfully authenticated 

http://0a1000ca049f7ab68386b999007d00d5.web-security-academy.net/my-account 
  

### Technical Explanation 

  

The vulnerability exists because: 

- Application checks: "Is user logged in?" (YES) 

- Application does NOT check: "Did user complete 2FA?" (MISSING) 

- Access control flaw = function-level authorization missing 

Correct: Verify login AND verify 2FA before allowing /my-account 

Broken: Only verify login, allow /my-account access 

  

### Real-World Impact 

  

**2FA bypass consequences:** 

- ✅ Multi-factor authentication defeated 

- ✅ Account takeover despite 2FA 

- ✅ Complete access to protected resources 

- ✅ All sensitive data exposed 

  

--- 

  

## Lab 3: Password Reset Broken Logic 

  

### Vulnerability Description 

Password reset tokens are not properly validated against user identity. Tokens can be used to reset ANY user's password, not just the intended user. 

  

### How I Exploited It 

  

**Step 1: Generate Reset Token** 

- Initiated password reset for target user (carlos) 

- Received password reset link with token 

- Captured request in HTTP history 

  

**Step 2: Token Analysis in Burp Repeater** 

1. Right-clicked password reset request 

2. Sent to Repeater tab 

3. Analyzed the request structure 

4. Identified token parameter 

5. Found username parameter in request 

  

**Step 3: Exploitation** 

- Modified username parameter from "my-account" to "carlos" 

- Kept the token (generated for carlos) 

- Sent request in Repeater 

- Server accepted the request 

- Successfully reset carlos's password 

  

**Step 4: Account Takeover** 

- Set new password for carlos account 

- Logged in as carlos with new password 

- Complete account compromise 

  

**Screenshot Evidence:** 




### Technical Explanation 

  

Password reset token validation should verify: 

✅ CORRECT: Token belongs to THIS user + Token is valid + Reset password 

❌ BROKEN: Token is valid + Reset password for whoever you say 

  

The vulnerability: 

- Token is validated (exists, not expired) 

- Username parameter is NOT validated against token 

- Server trusts username parameter 

- Token can be used for any user 

  

### Real-World Impact 

  

**Password reset exploitation:** 

- ✅ Reset any user's password (regardless of token ownership) 

- ✅ Account takeover without knowing original password 

- ✅ Admin account compromise 

- ✅ Complete system access 

  

**Real-world example:** Attacker generates reset token for admin, modifies username to admin, resets admin password, takes over entire system. 

  

--- 

  

## Burp Suite Techniques Demonstrated 

  

### Proxy Interception 

- Captured HTTP requests in real-time 

- Paused requests for analysis 

- Modified requests before forwarding 

  

### Intruder Tab 

- Automated payload injection (username enumeration) 

- Sniper attack mode (single variable testing) 

- Response analysis (length differences) 

- Identified valid data from responses 

  

### Repeater Tab 

- Manual request modification (password reset) 

- Analyzed token structure 

- Parameter tampering (username change) 

- Tested alternative values 

  

### HTTP History 

- Captured all requests automatically 

- Retrieved password reset tokens 

- Identified request patterns 

- Selected requests for deeper analysis 

  

--- 

  

## Common Authentication Vulnerabilities (OWASP A07) 

  

All three labs demonstrate **OWASP Top 10 A07:2021 - Authentication Failures** 

  

| Vulnerability | Impact | Lab Example | 

|----------------|--------|-------------| 

| Username enumeration | Information disclosure | Lab 1 | 

| 2FA bypass | Authentication bypass | Lab 2 | 

| Broken token logic | Authorization bypass | Lab 3 | 

  

--- 

  

## Real-World Scenarios 

  

### Scenario 1: Bank Website 

Attacker discovers admin username via enumeration → Brute forces password → Takes over admin account → Transfers funds 

  

### Scenario 2: Email Service   

Attacker bypasses 2FA → Accesses user's email → Resets passwords on other accounts → Complete identity theft 

  

### Scenario 3: Social Media Platform 

Attacker exploits password reset → Changes influential user's password → Posts malicious content → Spreads misinformation 

  

--- 

  

## Remediation & Prevention 

  

### Fix 1: Username Enumeration 

❌ BAD: 
if (user exists) → "Invalid password" 
if (user doesn't exist) → "Invalid user" 

✅ GOOD: 
Always respond: "Invalid username or password" 
(same message for both cases) 

  

### Fix 2: 2FA Bypass 

✅ CORRECT IMPLEMENTATION: 
if (logged in) AND (2FA completed) → Allow /my-account 
if (logged in) AND NOT (2FA completed) → Redirect to 2FA 

if NOT (logged in) → Redirect to login 
if (2FA needed) → Enforce 2FA, no bypasses 

  

### Fix 3: Password Reset Tokens 

❌ BAD: 
Token valid? YES → Reset password for [username parameter] 

✅ GOOD: 
Token valid? YES + Token belongs to current user? YES → Reset password 
Token doesn't match user? → Reject request 

  

--- 

  

## Tools & Technologies Used 

  

| Tool | Used For | Proficiency | 

|------|----------|-------------| 

| **Burp Suite Community** | Proxy, Intruder, Repeater | Intermediate | 

| **PortSwigger Academy** | Lab environment | Intermediate | 

| **HTTP Protocol** | Understanding requests/responses | Intermediate | 

| **Authentication Concepts** | Understanding vulnerabilities | Intermediate-Advanced | 

  

--- 

  

## Key Learnings 

  

### Technical Insights 

  

1. **Response Analysis** 

   - Different response lengths reveal information 

   - Error messages leak data 

   - Timing differences can indicate valid data 

  

2. **Access Control vs Authentication** 

   - Authentication ≠ Authorization 

   - Must verify BOTH for security 

   - Incomplete checks = bypass possible 

  

3. **Token Security** 

   - Tokens must be bound to users 

   - Cannot be used interchangeably 

   - Validation must verify token + user match 

  

4. **Burp Suite Mastery** 

   - Intruder automates testing 

   - Repeater allows manual modification 

   - Response analysis reveals vulnerabilities 

   - HTTP History tracks all traffic 

  

### Security Mindset 

  

- **Test all authentication paths:** Not just login, but 2FA, password reset, account recovery 

- **Assume defaults are broken:** Developer often forget function-level authorization 

- **Analyze responses carefully:** Different lengths/messages reveal information 

- **Test token validity:** Can tokens be reused? Modified? Used by others? 

  

### Professional Competencies 

  

✅ Vulnerability identification (3 different types)   

✅ Exploitation methodology (3 different techniques)   

✅ Burp Suite proficiency (Proxy, Intruder, Repeater)   

✅ HTTP protocol understanding   

✅ Technical communication (explaining findings)   

  

--- 

  

## Portfolio Progress 

  

| Week | Labs | Write-ups | Screenshots | Tools Used | 

|------|------|-----------|-------------|-----------| 

| **Week 1** | 1 | 1 | 4-5 | Burp Proxy/Repeater | 

| **Week 2** | 3 | 1 | 6-8 | Burp Intruder/Repeater | 

| **Total** | 4 | 2 | 10-13 | Multiple Burp modules | 

  

--- 

  

## Self-Assessment 

  

| Criterion | Rating | Evidence | 

|-----------|--------|----------| 

| Concept Understanding | 8/10 | Explained 3 auth vulnerabilities clearly | 

| Hands-On Execution | 9/10 | Exploited all 3 labs successfully | 

| Tool Proficiency | 8/10 | Used Intruder, Repeater, Proxy effectively | 

| Documentation | 7/10 | Clear explanations with evidence | 

| Real-World Application | 8/10 | Understand business impact clearly | 

| Overall Readiness | 8/10 | Strong Week 2, ready for Week 3 | 

  

--- 

  

**Report Prepared By:** John babu K 

**Date:** 16th September 2026 

**Portfolio Status:** 2 Write-ups, 4 Labs, Growing   

  

--- 

  
