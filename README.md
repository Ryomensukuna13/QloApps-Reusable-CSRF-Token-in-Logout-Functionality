# QloApps-Reusable-CSRF-Token-in-Logout-Functionality

**Product:** QloApps  
**Tested Version:** 1.7.0  
**CWE:** CWE-639: Authorization Bypass Through User-Controlled Key, CWE-352: Cross-Site Request Forgery (CSRF)
## Description
QloApps uses a token parameter (`token=...`) in the logout function to protect against CSRF attacks. However, this implementation is flawed:

1. The **logout URL exposes the CSRF token** in the query string.  
   - Example:  
     ```
     https://demo.qloapps.com/103-208-230-52/?mylogout=1&token=a2edbe57e755bafd90828744d4a8679d
     ```
   - Since the token is in the URL, it can be leaked via browser history, server logs, referrer headers, and third-party analytics scripts.  

2. The token is **reusable across multiple requests** and does not expire after one use.  

Together, these weaknesses allow attackers to repeatedly force a user logout and potentially bypass CSRF protections if similar token logic is used in other sensitive operations.

---

## Proof of Concept (PoC)

1. Login to the QloApps demo site.  
2. Capture the logout request:
<img width="404" height="224" alt="image" src="https://github.com/user-attachments/assets/51cf8929-7e50-4572-a57d-680c223309bc" />
3. Observe that the **response includes the token in the URL**, making it accessible to logs and third-party trackers.  
4. Replay the same request multiple times in Burp Suite.  
5. The server consistently returns **200 OK**, and the same token is accepted.  

This demonstrates that the token is both **exposed** and **reusable**.

---

## Security Impact
- Attackers can repeatedly terminate user sessions via a malicious link or hidden image:  
```html
<img src="https://demo.qloapps.com/103-208-230-52/?mylogout=1&token=a2edbe57e755bafd90828744d4a8679d" />
