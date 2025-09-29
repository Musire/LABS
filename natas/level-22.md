# [Over The Wire (natas)] – [[Platform](http://natas22.natas.labs.overthewire.org/)] – [09/29/2025]

## Objective
Find the password for the next Natas level by exploiting the vulnerability on this page.  

## Environment / Platform
- Platform: OverTheWire – Natas
- Level: [22]
- Difficulty: [Easy]

## Tools Used
- Chromium Browser
- Burp Suite (proxy + repeater)

## Login
1. Logged in with credentials:
   - **Username**: `natas22
   - **Password**: `XXXXXX`
  
2. Screenshot:
   ![alt text](image-32.png)
   
   - this prompted that I should check the sourcecode by clicking the link `http://natas22.natas.labs.overthewire.org/index-source.html`

3. Observed Page Content

```php
      session_start();

      if(array_key_exists("revelio", $_GET)) {
         // only admins can reveal the password
         if(!($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1)) {
         header("Location: /");
         }
      }
```
   - This starts a session and checks if there is a "revelio" key in the GET request array. If the admin key exists in the session array and if it equals 1. If it doesn't then redirects to the root directory.


```php
      if(array_key_exists("revelio", $_GET)) {
         print "You are an admin. The credentials for the next level are:<br>";
         print "<pre>Username: natas23\n";
         print "Password: <censored></pre>";
      } 
```
   - If the key "revelio" is present in the GET request, then will reveal flag.

---
  

     
4. Steps taken (Broken Access Control):

   1. First thing I tried was sending the "revelio" key through the params in the url, however when the page was loaded it would redirect to the `/` route becuase of the `302` status response.

   2. The chromium browser automatically follows the redirect. Attempting the same request including the endpoint `/?revelio=true`.
   
   2. This produced the flag.

   ```html
      <div id="content">

         You are an admin. The credentials for the next level are:<br><pre>Username: natas23
         Password: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX</pre>
         <div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
      </div>
   ```
   

---

🔑 **Why this works**: 

   - The code checks for $_GET["revelio"] and, if present, attempts to block non‑admins by sending header("Location: /"); — but it does not stop execution after sending that header.

   - header() only sends an HTTP header; PHP continues executing the script unless you call exit()/die(). As a result, the later block that prints the credentials still runs even for non‑admins.

   - The redirect relies on client navigation to hide the response, but a client can (a) follow a different URL, (b) inspect raw responses (curl/proxy), or (c craft requests that reach the code path that prints the flag — therefore the server still reveals protected data.

  - Because the authorization decision is not enforced as a hard server-side guard (i.e., stop execution or deny output), the protection is effectively bypassed.

---

💥 **Impact**

   - **Unauthorized information disclosure** — an attacker can obtain sensitive data (the next-level password) without legitimate admin privileges.

   - **Low complexity exploit** — trivial to reproduce using a browser, curl, or an intercepting proxy; no advanced skills required.

   - **Automatable and scalable** — can be scripted to harvest data if similar flaws exist elsewhere.

   - **Erodes trust in access controls** — exposes systemic weaknesses that could be leveraged for further attacks.

---
  
🛠️ **Remediation**

   - Check authorization before any sensitive output is produced. Don’t rely on redirects to hide data — prevent generating/printing sensitive data unless the caller is authorized.

   - Avoid control-flow mistakes: never rely on client behavior (automatic redirects) as a security control — enforce logic server‑side.

   - Minimize information returned: do not include sensitive credentials in source or HTML; require authenticated admin context to access such resources (and log/monitor accesses).

   - Harden session/auth logic: ensure $_SESSION["admin"] is set by server-side logic only, regenerate session IDs on privilege changes, and validate session integrity.

   - Code review & testing: add automated tests to assert that non‑admin users cannot reach privileged outputs (unit/integration tests and security-focused checks).

   - Remove debug/verbose behavior in production that could reveal internal state.