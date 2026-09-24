# Access Control Vulnerabilities - PortSwigger Web Security Academy

**Week 3 Lab Report** | **Penetration Testing Portfolio** | **9 Labs Completed**

---

## Executive Summary

This report documents the completion of 9 apprentice-level access control labs from PortSwigger Web Security Academy. Access control (OWASP A01) is a critical vulnerability that allows attackers to bypass authorization checks and perform unauthorized actions or view restricted data.

**Labs Completed:** 9/9  
**Difficulty Level:** Apprentice  
**Date Completed:** September 20, 2026  
**Tools Used:** Burp Suite (Proxy, Repeater, Intruder), Browser Developer Tools  
**OWASP Category:** A01:2021 – Broken Access Control

---

## Access Control Fundamentals

Access control enforces authorization policy to ensure users cannot act outside of their intended permissions. Common vulnerabilities include:

- **Unprotected sensitive functions:** Admin panels accessible without authentication
- **Parameter-based access control:** User ID in request parameter can be modified
- **Privilege escalation:** Users can modify their role/permissions
- **Horizontal escalation:** Users can access other users' data
- **Vertical escalation:** Users can access admin functions
- **Insecure Direct Object References (IDOR):** Direct references to objects without proper authorization

---

## Lab 1: Unprotected Admin Functionality

**Objective:** Access admin panel and delete a user account without proper authentication.

**Vulnerability Type:** Unprotected sensitive function  
**OWASP Category:** A01 – Broken Access Control

### Attack Steps:

1. **Identification:**
   - Application has an admin panel at `/administrator-panel`
   - Panel is accessible without any authentication check
   - No role-based access control implemented

2. **Exploitation:**
   - Navigate directly to admin URL: `https://[target]/administrator-panel`
   - Access granted without credentials required
   - Admin panel displays user list with delete functionality

3. **Proof of Concept:**
   - Located user "wiener" in Users list
   - Clicked "Delete" button next to user
   - System message: "User deleted successfully!"
   - Lab marked as SOLVED ✅

### Key Findings:

- Admin panel has no authentication requirement
- No authorization check before allowing admin actions
- No role verification in the application
- Simple URL guessing allowed unauthorized access

### Remediation:

- Implement proper authentication (login required)
- Verify user role before granting admin panel access
- Use role-based access control (RBAC)
- Implement session token validation
- Add audit logging for admin actions

---

## Lab 2: User ID Controlled by Request Parameter with Data Leakage

**Objective:** Access other user's account and retrieve sensitive data.

**Vulnerability Type:** Horizontal privilege escalation + information disclosure  
**OWASP Category:** A01

### Attack Steps:

1. **Identification:**
   - User account accessible via `/my-account?id=[user_id]`
   - User ID is directly controlled by request parameter
   - Parameter value can be modified to access other users
   - Sensitive data leaked in HTTP response

2. **Exploitation:**
   - Used Burp Proxy to intercept request to `/my-account?id=carlos`
   - Sent request to Repeater for analysis
   - Reviewed HTTP response containing user details
   - Response included sensitive data (email, API key, etc.)

3. **Proof of Concept:**
   - Intercepted: `GET /my-account?id=carlos HTTP/2`
   - Response Status: 200 (data leakage)
   - Retrieved: User email, API Key, account details
   - Lab marked as SOLVED ✅

### Key Findings:

- Direct user ID in parameter allows horizontal escalation
- No authorization check on user parameter
- Sensitive data exposed in response headers
- HTTP response includes plaintext credentials
- No rate limiting on parameter enumeration

### Burp Suite Tools Used:

- **Proxy:** Intercepted requests
- **Repeater:** Modified user ID parameter and sent requests
- **HTTP History:** Reviewed responses containing leaked data

### Remediation:

- Never trust user input for object references
- Implement proper authorization checks
- Use indirect reference maps (users→object_ids)
- Encrypt sensitive data in transit
- Implement rate limiting on parameter enumeration
- Use HTTPS for all requests

---

## Lab 3: User ID Controlled by Request Parameter with Unpredictable User IDs

**Objective:** Access other user's account despite unpredictable user IDs.

**Vulnerability Type:** Horizontal privilege escalation  
**OWASP Category:** A01

### Attack Steps:

1. **Identification:**
   - User IDs are unpredictable (GUIDs/UUIDs)
   - User ID found in response content (HTML comments, JavaScript)
   - No authorization check on `/my-account?id=[guid]` endpoint

2. **Exploitation:**
   - Reviewed HTML response containing user GUID
   - Found comment with other user's GUID
   - Modified request parameter to target user's GUID
   - Accessed target user's account despite unpredictable ID

3. **Proof of Concept:**
   - Intercepted: `GET /my-account?id=[unpredictable_guid]`
   - Response Status: 200 OK
   - Retrieved: Target user's email, API key, account info
   - Lab marked as SOLVED ✅

### Key Findings:

- Unpredictable IDs alone don't guarantee security
- GUIDs disclosed in HTML source/comments
- No server-side authorization verification
- Application trusts any valid GUID format
- GUIDs enumerable through information disclosure

### Remediation:

- Remove sensitive GUIDs from client-side responses
- Verify user owns the resource being accessed
- Use server-side session to validate access
- Implement authorization checks on every request
- Never rely on obscurity of IDs

---

## Lab 4: User ID Controlled by Request Parameter (API Key Exposure)

**Objective:** Exploit parameter-based access control to retrieve user's API key.

**Vulnerability Type:** Horizontal privilege escalation + credential exposure  
**OWASP Category:** A01

### Attack Steps:

1. **Identification:**
   - `/my-account` endpoint accessible with user ID parameter
   - API Key displayed in "My Account" page
   - No authorization check validates user owns account

2. **Exploitation:**
   - Accessed own account: `/my-account?id=wiener`
   - Noted username and API Key structure
   - Changed ID parameter to target user: `?id=carlos`
   - Retrieved target user's API Key from response

3. **Proof of Concept:**
   - Burp Proxy intercepted: `GET /my-account?id=carlos`
   - Response contained: `Your API Key is: watNYsYKHggP9DuoyI1nxvifEnOh6wY2`
   - API Key exposed without authentication/authorization
   - Lab marked as SOLVED ✅

### Key Findings:

- Sensitive credentials (API Keys) exposed in responses
- No validation of request origin
- No verification user owns requested account
- API Keys transmitted unencrypted
- No session-based access control

### Remediation:

- Implement server-side session validation
- Use session tokens, not user IDs in URLs
- Verify current user owns requested resource
- Encrypt API Keys in storage and transit
- Implement rate limiting on API key retrieval
- Add audit logging for credential access

---

## Lab 5: User Role Can Be Modified in User Profile

**Objective:** Escalate privileges by modifying user role parameter in POST request.

**Vulnerability Type:** Vertical privilege escalation  
**OWASP Category:** A01

### Attack Steps:

1. **Identification:**
   - User profile editable via POST request to `/my-account/change-email`
   - Request includes JSON parameter: `"roleId": 2`
   - Role parameter sent by client, no server-side validation
   - Admin role is roleId = 1, user role = 2

2. **Exploitation:**
   - Accessed `/my-account` and initiated email change
   - Burp Repeater intercepted POST request
   - Modified JSON: Changed `"roleId": 2` to `"roleId": 1`
   - Submitted modified request with Send button

3. **Proof of Concept:**
   - Original request: `{"email":"test.test@com","roleId":"2"}`
   - Modified request: `{"email":"test.test@com","roleId":"1"}`
   - Response Status: 302 Found (successful modification)
   - User privileges escalated to admin
   - Lab marked as SOLVED ✅

### Key Findings:

- Client sends role information in request
- Server trusts client-supplied role values
- No server-side role validation
- No CSRF token protection
- Direct privilege escalation possible

### Burp Suite Tools Used:

- **Repeater:** Modified request parameters
- **Inspector:** Viewed JSON payload structure
- **Send button:** Submitted malicious request

### Remediation:

- Never trust client-supplied role information
- Store user role server-side in session
- Verify role from session, not request
- Implement role-based access control (RBAC)
- Add CSRF tokens to state-changing requests
- Implement audit logging for role changes

---

## Lab 6: User Role Controlled by Request Parameter (Cookie Manipulation)

**Objective:** Escalate privileges by modifying role parameter in browser cookie.

**Vulnerability Type:** Vertical privilege escalation  
**OWASP Category:** A01

### Attack Steps:

1. **Identification:**
   - User role stored in cookie: `Admin=false`
   - Cookie directly controls access to admin panel
   - No cryptographic validation of cookie value
   - Cookie trust exceeds server-side verification

2. **Exploitation:**
   - Opened Browser Developer Tools (F12)
   - Navigated to Application → Cookies
   - Located cookie: `Admin=false`
   - Modified cookie value: `Admin=true`
   - Refreshed page

3. **Proof of Concept:**
   - Modified cookie: `Admin=false` → `Admin=true`
   - Server accepted modified cookie without verification
   - Admin panel access granted
   - User deletion performed successfully
   - Lab marked as SOLVED ✅

### Browser Developer Tools Steps:

1. Press F12 to open DevTools
2. Click Application tab
3. Expand Cookies section
4. Select website cookie
5. Double-click Admin cookie value
6. Change false → true
7. Press Enter to confirm
8. Refresh page

### Key Findings:

- Sensitive authorization data in client-side cookie
- Server trusts cookie values without validation
- No cryptographic signing of cookies
- No timestamp validation
- Simple boolean controls critical access

### Remediation:

- Store user role server-side only (in session)
- Use session IDs in cookies, not sensitive data
- Implement server-side role verification
- Use signed/encrypted cookies
- Add HMAC validation for cookie integrity
- Implement HttpOnly flag for cookies
- Use Secure flag (HTTPS only)
- Add SameSite attribute for CSRF protection

---

## Lab 7: Unprotected Admin Functionality with Unpredictable URL

**Objective:** Discover and access admin panel with unpredictable URL path.

**Vulnerability Type:** Information disclosure + unprotected admin function  
**OWASP Category:** A01

### Attack Steps:

1. **Identification:**
   - Admin panel hidden at `/admin-32kjx0` (unpredictable path)
   - URL not publicly documented
   - No authentication required to access

2. **Exploitation:**
   - Navigated to unpredictable URL: `/admin-32kjx0`
   - Access granted without authentication
   - Admin panel fully functional
   - User deletion performed successfully

3. **Proof of Concept:**
   - Accessed: `https://[target]/admin-32kjx0`
   - Status: 200 OK (no auth required)
   - Deleted user "wiener"
   - System message: "User deleted successfully!"
   - Lab marked as SOLVED ✅

### Key Findings:

- Security through obscurity is ineffective
- Unpredictable URLs without authentication = false security
- Admin functionality completely unprotected
- URL discoverable through multiple methods

### Remediation:

- Implement proper authentication (login required)
- Require authorization checks for all admin functions
- Use strong access control on sensitive endpoints
- Don't rely on URL obscurity
- Implement authentication before authorization
- Add audit logging for admin access

---

## Lab 8: User ID Controlled by Request Parameter with Password Disclosure

**Objective:** Access another user's account and retrieve their password.

**Vulnerability Type:** Horizontal privilege escalation + credential exposure  
**OWASP Category:** A01

### Attack Steps:

1. **Identification:**
   - User ID parameter: `/my-account/change-password?id=[user]`
   - Password input field contains user's current password
   - HTML form has password field with value attribute
   - No authorization on password change endpoint

2. **Exploitation:**
   - Intercepted request: `POST /my-account/change-password HTTP/2`
   - Modified user ID parameter to target user
   - Reviewed response HTML source
   - Found password in form field value attribute (highlighted in Burp)

3. **Proof of Concept:**
   - Burp Repeater intercepted password change form
   - Modified user parameter
   - HTML response contained password in form value
   - Password retrieved from form value attribute
   - Lab marked as SOLVED ✅

### Key Findings:

- Password included in HTML form value attribute
- No authorization check on password change
- User ID parameter allows accessing other users
- Password visible in browser history/cache
- Credentials exposed in HTTP response body

### Security Issues:

1. **Authorization:** No check if user owns password
2. **Information Disclosure:** Password in HTML source
3. **Credential Exposure:** Password in plain HTTP response

### Remediation:

- Verify current user owns the password being changed
- Never send password in response
- Don't use value attribute for password fields
- Implement server-side session validation
- Require current password for changes
- Use HTTPS for all requests
- Add audit logging for password changes

---

## Lab 9: Insecure Direct Object References (IDOR)

**Objective:** Access live chat conversations of other users via IDOR vulnerability.

**Vulnerability Type:** Insecure Direct Object References (IDOR)  
**OWASP Category:** A01

### Attack Steps:

1. **Identification:**
   - Live chat accessible via `/download-transcript/[id]`
   - Chat ID directly referenced in URL
   - No verification user owns the transcript

2. **Exploitation:**
   - Accessed own chat transcript: `/download-transcript/1.txt`
   - Retrieved own chat history
   - Incremented ID parameter: `/download-transcript/2.txt`
   - Accessed other user's transcript

3. **Proof of Concept:**
   - Own transcript: `1.txt` - Status: 200 OK
   - Target transcript: `2.txt` - Status: 200 OK (unauthorized!)
   - Retrieved conversation with sensitive information
   - Downloaded transcript confirmed password
   - Lab marked as SOLVED ✅

### Key Findings:

- Direct object references (transcript IDs) in URL
- No authorization check on object ownership
- Direct enumeration of transcripts possible
- Sensitive data (passwords) in chat history

### IDOR Impact:

- Access to private conversations
- Credential disclosure
- Privacy violation
- Data breach risk

### Remediation:

- Verify user owns object before returning data
- Use indirect reference maps
- Implement authorization checks
- Use secure tokens instead of sequential IDs
- Implement access control lists (ACLs)
- Add audit logging for object access
- Implement rate limiting on object retrieval

---

## Tools & Techniques Summary

### Burp Suite Features Used:

| Tool | Usage | Labs |
|------|-------|------|
| **Proxy** | Intercepted requests/responses | All labs |
| **Repeater** | Modified parameters and resent | 2, 3, 4, 5, 8, 9 |
| **HTTP History** | Reviewed all requests/responses | 2, 3, 4 |
| **Inspector** | Analyzed JSON payloads | 5 |

### Browser Developer Tools:

| Feature | Usage | Lab |
|---------|-------|-----|
| **Inspector** | Viewed HTML source | 1, 7 |
| **Application** | Modified cookies | 6 |
| **Network** | Monitored requests | All labs |

---

## OWASP A01 - Broken Access Control

### 1. Vertical Access Control (Privilege Escalation)
- User gains higher privileges (user → admin)
- **Labs:** 5, 6

### 2. Horizontal Access Control (Lateral Movement)
- User accesses other users' resources
- **Labs:** 2, 3, 4, 8, 9

### 3. Context-Dependent Access Control
- Access depends on application state
- **Labs:** 1, 7

---

## Key Learning Outcomes

1. **Authorization is Critical:** Authentication ≠ Authorization
2. **Trust Nothing:** Never trust client-supplied security data
3. **Parameter Manipulation:** User input controls are vulnerable
4. **Information Disclosure:** Many paths to sensitive data
5. **Defense in Depth:** Multiple controls are necessary
6. **Testing:** Systematic parameter testing finds vulnerabilities

---

## Mitigation Strategies

### For Developers:

1. Implement server-side authorization checks
2. Verify user owns resource before access
3. Use session-based access control
4. Never trust client input for permissions
5. Implement proper RBAC
6. Add audit logging
7. Use secure tokens for object references

### For Security Testers:

1. Test all parameters for manipulation
2. Attempt vertical privilege escalation
3. Attempt horizontal privilege escalation
4. Review all responses for sensitive data
5. Test information disclosure vectors
6. Enumerate resources systematically
7. Document all findings with proof

---

## Recommendations

### Immediate Actions:

- Implement server-side authorization for all endpoints
- Remove sensitive data from responses
- Validate object ownership before access
- Implement proper authentication

### Short-term:

- Add CSRF tokens to state-changing requests
- Implement audit logging
- Conduct access control testing
- Implement rate limiting

### Long-term:

- Implement RBAC framework
- Use security frameworks
- Conduct regular security testing
- Security training for developers

---

## Conclusion

This lab series demonstrates critical access control vulnerabilities that affect real applications. The key insight is that **authorization must be enforced server-side on every request**, regardless of client-side controls or URL obscurity.

**All 9 labs completed successfully.** ✅

---

## Statistics

- **Labs Completed:** 9/9
- **Vulnerabilities Tested:** 9
- **Tools Used:** 3+
- **Hours Spent:** ~6-8 hours
- **Success Rate:** 100%

---

**Lab Report Created:** Friday, September 23, 2026  
**Penetration Tester:** John Babu Korlam  
**Portfolio:** https://github.com/JohnbabuK-create/offensive-security-portfolio
