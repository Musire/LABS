# [Over The Wire (natas)] – [[Platform](http://natas2.natas.labs.overthewire.org/)] – [09/18/2025]

## Objective
Find the password for the next Natas level by exploiting the vulnerability on this page.  

## Environment / Platform
- Platform: OverTheWire – Natas
- Level: [0]
- Difficulty: [Easy]

## Tools Used
- Chromium Browser (Browser DevTools)

## Steps Taken
1. Logged in with credentials:
   - **Username**: `natas0`
   - **Password**: `XXXXXX`
  
2. Devtools - Elements:
   - took a quick search around the site html and found this
  
3. Observed page content:
   ```html
    <div id="content">
        You can find the password for the next level on this page.

        <!--The password for natas1 is XXXXXXXXXXXXXXXXXX -->
    </div>


---

🔑 Why this works:  
- Introduction to basic CTF style in web-based applications


