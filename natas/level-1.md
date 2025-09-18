# [Over The Wire (natas)] – [[Platform](http://natas1.natas.labs.overthewire.org/)] – [09/18/2025]

## Objective
Find the password for the next Natas level by exploiting the vulnerability on this page.  

## Environment / Platform
- Platform: OverTheWire – Natas
- Level: [1]
- Difficulty: [Easy]

## Tools Used
- Chromium Browser (Browser DevTools)

## Steps Taken
1. Logged in with credentials:
   - **Username**: `natas1`
   - **Password**: `XXXXXX`
  
2. Devtools - Elements:
   - right click disabled on the browser, ctrl + shift + I or f12 is enabled
   - took a quick search around the site html and found this
  
3. Observed page content:
   ```html
    <div id="content">
      You can find the password for the
      next level on this page, but rightclicking has been blocked!

      <!--The password for natas2 is xxxxxxxxxxxxxxxxxxxxxxxxx -->
    </div>


---

🔑 Why this works:  
- reminder that even though primary methods disabled there are always secondary methods to acheive the objective.


