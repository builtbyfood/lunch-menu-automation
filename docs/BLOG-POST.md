# Automating School Lunch Menus with Home Assistant & Node-RED: An Epic Debugging Journey

**Or: How I learned to stop worrying and love `wget`**

## 🎯 The Goal

Automate downloading my child's monthly lunch menu from our school's parent portal and display it on a tablet in kiosk mode. Sounds simple, right?

*Narrator: It was not simple.*

## 📋 Table of Contents

1. [The Setup](#the-setup)
2. [Phase 1: Authentication Success](#phase-1-authentication-success)
3. [Phase 2: The File Save Mystery](#phase-2-the-file-save-mystery)
4. [Phase 3: The Great PDF Corruption](#phase-3-the-great-pdf-corruption)
5. [Phase 4: Filesystem Revelation](#phase-4-filesystem-revelation)
6. [Phase 5: The Blank PDF Problem](#phase-5-the-blank-pdf-problem)
7. [The Final Solution](#the-final-solution)
8. [Lessons Learned](#lessons-learned)
9. [Complete Implementation Guide](#complete-implementation-guide)

---

## The Setup

**Environment:**
- Home Assistant OS
- Node-RED addon
- School parent portal (web-based platform)
- Goal: Display PDF lunch menu on tablet in kiosk mode

**Initial Approach:**
"I'll just download a PDF from a website. How hard can it be?"

*Spoiler: Very hard.*

---

## Phase 1: Authentication Success ✅

### The Challenge
school portal requires authentication. No direct PDF links.

### The Solution
Built a complete authentication flow in Node-RED:

1. **GET /signin** - Fetch signin page
2. **Extract CSRF token** - Parse HTML for `authenticity_token`
3. **POST /sessions** - Submit login with credentials
4. **Extract auth cookies** - Get session cookies from redirect
5. **Access protected pages** - Use cookies for authenticated requests

### Key Learnings

**Critical Detail #1: Cookie Encoding**
```javascript
// ❌ Wrong
msg.headers['Cookie'] = cookieValue;

// ✅ Right
msg.headers['Cookie'] = 'ps_s=' + encodeURIComponent(cookieValue);
```

**Critical Detail #2: Complete User-Agent**
```javascript
'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36'
```

Short or missing user agents = rejected requests!

**Critical Detail #3: Cookie Extraction**
```javascript
// Cookies are in redirectList, not responseCookies!
const redirectCookies = msg.redirectList[0].cookies || {};
```

---

## Phase 2: The File Save Mystery 🤔

### The Problem
Flow showed "SUCCESS! File saved" but... no file existed anywhere.

**Debug output:**
```
Step 10: Downloading PDF...
SUCCESS! Menu saved: PreK-February.pdf
```

**File system:**
```bash
$ ls /config/www/lunch-menu/
# Nothing here
```

### Attempted Solutions

**Attempt 1: File Node**
```
HTTP Download → File Node (encoding: binary)
Result: No file created ❌
```

**Attempt 2: Function Node with `fs.writeFile`**
```javascript
const fs = require('fs');
fs.writeFileSync(path, data);
```
```
Result: ReferenceError: require is not defined ❌
```

**Key Learning:** Node-RED function nodes run in a sandboxed environment. No `require()` allowed!

---

## Phase 3: The Great PDF Corruption 😱

### The Breakthrough
Files WERE being saved! But they were corrupt!

**Downloaded:**
```
buffer[137484]  // 137KB - looks good!
```

**Saved file:**
```bash
-rw-r--r-- 1 root root 131075 Feb 15 13:40 current.pdf
# Only 131KB - 6KB missing!
```

**Opening the PDF:** Blank white pages 😱

### Investigation

**Theory 1: Encoding Issues**
- Tried `encoding: none`
- Tried `encoding: binary`
- Tried `encoding: ""`
- Still corrupted!

**Theory 2: File Node Bug**
Discovered the file node was converting binary buffer to STRING!

**Debug evidence:**
```javascript
// Before file node
msg.payload: buffer[137484] ✅

// After file node
msg.payload: string ❌
```

The file node was converting our perfect binary PDF into a string, losing data!

---

## Phase 4: Filesystem Revelation 🤯

### The Discovery
```bash
$ ls /config/www/lunch-menu/
# File from yesterday

$ ls /share/lunch-menu/
# File from today!
```

**Mind. Blown.**

Node-RED addon has its OWN filesystem! `/config` in Node-RED ≠ `/config` in Home Assistant!

**Filesystem Map:**
```
Node-RED Container:
  /config → Node-RED's config
  /share → Shared with Home Assistant ✅

Home Assistant:
  /config → Home Assistant's config
  /share → Shared with Node-RED ✅
```

**Solution:** Save to `/share`, then copy to Home Assistant's `/config/www`

---

## Phase 5: The Blank PDF Problem 😤

### The Mystery Deepens

Now files were saving to `/share`, but PDFs were STILL blank!

**Size:** 134KB (correct!)  
**Content:** White pages (incorrect!)

### The Investigation

Checking what we actually downloaded:

```bash
head -c 200 /share/lunch-menu/menu.pdf
```

Expected (real PDF):
```
%PDF-1.7
%����
1 0 obj
```

Got (HTML error page):
```html
<!DOCTYPE html>
<html>
<head><title>Access Denied</title>
```

**We were downloading HTML, not PDF!**

### Root Cause: Signed S3 URLs

The PDF URL looked like:
```
https://rails-parentsquare-prod.s3.amazonaws.com/feeds/BDM6L5GvTbKmVUSGyzzJ_School%20PreK-February.pdf?response-content-disposition=inline%3Bfilename%3D%22School%20PreK-February.pdf%22&X-Amz-Expires=21600&X-Amz-Date=20260215T192746Z&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEFsaCXVzLWVhc3QtMSJIMEY...
```

**The Problem:**
- Those `X-Amz-*` parameters are AWS signature tokens
- They were causing authentication issues
- Node-RED was downloading "Access Denied" pages instead of PDFs!

**Manual Test:**
Removing everything after `.pdf` in the URL → PDF loaded perfectly in browser!

### The Fix: Strip Query Parameters

```javascript
// Strip everything after .pdf
const pdfMatch = pdfUrl.match(/(https:\/\/[^?]+\.pdf)/);
if (pdfMatch) {
    pdfUrl = pdfMatch[1];
}
```

**Before:**
```
...School%20PreK-February.pdf?response-content-disposition=...
```

**After:**
```
...School%20PreK-February.pdf
```

**Result:** Still blank! 😭

---

## The Final Solution 💡

### The Ultimate Problem

Even with stripped URLs, PDFs were blank. Why?

**The cookies!** 

When manually accessing the URL in a browser, my browser sent authentication cookies. `wget` without cookies = Access Denied!

### The Winning Combination

**1. Strip signed URL tokens**
```javascript
const pdfMatch = pdfUrl.match(/(https:\/\/[^?]+\.pdf)/);
pdfUrl = pdfMatch[1];  // Base URL only!
```

**2. Use `wget` with authentication cookies**
```bash
wget --header="Cookie: ps_s=...; ps_r=...; ps_d=..." \
     --user-agent="Mozilla/5.0..." \
     -O /share/lunch-menu/menu.pdf \
     "https://...pdf"
```

**3. Trigger Home Assistant to copy file**
```javascript
// Node-RED calls webhook
POST http://homeassistant:8123/api/webhook/lunch_menu_downloaded

// Home Assistant automation copies file
cp /share/lunch-menu/menu.pdf /config/www/lunch-menu/menu.pdf
```

---

## Lessons Learned 🎓

### 1. **Signed URLs Can Be Deceptive**
Just because a URL works in your browser doesn't mean it works programmatically. Browsers send cookies automatically!

### 2. **Node-RED File Node Has Limitations**
Binary data + file node = potential corruption. For critical binary files, use shell commands.

### 3. **Container Filesystems Are Isolated**
`/config` in one container ≠ `/config` in another. Use shared volumes (`/share`)!

### 4. **Debug What You're Actually Downloading**
```bash
head -c 200 file.pdf
```
Is it really a PDF? Or an error page that saved as PDF?

### 5. **`require()` Doesn't Work in Node-RED Functions**
Node-RED functions are sandboxed. Use built-in nodes or exec for external tools.

### 6. **`wget` > HTTP Request Node for Complex Downloads**
When dealing with:
- Signed URLs
- Complex authentication
- Binary files
- Weird edge cases

Just use `wget`. It's battle-tested and reliable.

### 7. **Separate Concerns**
Let Node-RED do what it's good at (workflow). Let Home Assistant do what it's good at (file operations, automations).

---

## Complete Implementation Guide 📚

### Part 1: Home Assistant Configuration

**1. Add Shell Command**

Edit `/config/configuration.yaml`:
```yaml
shell_command:
  copy_lunch_menu: "cp /share/lunch-menu/menu.pdf /config/www/lunch-menu/menu.pdf"
```

**2. Create Automation**

Edit `/config/automations.yaml`:
```yaml
automation:
  - id: copy_lunch_menu_after_download
    alias: "Copy Lunch Menu to WWW"
    trigger:
      - platform: webhook
        webhook_id: lunch_menu_downloaded
    action:
      - service: shell_command.copy_lunch_menu
      - service: persistent_notification.create
        data:
          title: "Lunch Menu Updated"
          message: "New lunch menu is now available!"
```

**3. Add Panel (Optional)**

```yaml
panel_iframe:
  lunch_menu:
    title: "Lunch Menu"
    icon: mdi:food-apple
    url: "/local/lunch-menu/dashboard.html"
```

**4. Restart Home Assistant**

---

### Part 2: Node-RED Flow

**Import:** `lunch-menu-FINAL-COMPLETE.json`

**Key Nodes:**

1. **Load Credentials** - Update with your school portal credentials
2. **Prepare Signin** - GET /signin
3. **Extract & Login** - POST /sessions with credentials
4. **Validate Login** - Extract auth cookies
5. **Get Menu** - Access protected pages
6. **Parse Menu** - Find PDF URL and strip tokens
7. **Download with wget** - Download to `/share` with cookies
8. **Trigger HA Webhook** - Tell Home Assistant to copy file

**Update Your Password:**
```javascript
const username = 'your@email.com';
const password = 'your_password';  // Change this!
```

**Update School ID (if needed):**
```javascript
msg.url = 'https://www.parentsquare.com/schools/YOUR_SCHOOL_ID/feeds/files';
```

---

### Part 3: Dashboard

**Upload Dashboard**

Save to: `/config/www/lunch-menu/dashboard.html`

**Features:**
- Displays PDF using PDF.js (works in all browsers!)
- Floating refresh button
- Page navigation for multi-page menus
- Full-width display
- Blue gradient background

**Access:**
```
http://homeassistant:8123/local/lunch-menu/dashboard.html
```

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        Node-RED Flow                         │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  1. Login to school portal                                    │
│     ├─ GET /signin → Extract CSRF token                     │
│     ├─ POST /sessions → Get auth cookies                    │
│     └─ Validate redirect                                     │
│                                                               │
│  2. Access Menu Page (with auth cookies)                     │
│     └─ GET /schools/XXXXX/feeds/files                       │
│                                                               │
│  3. Parse HTML → Find PDF URLs                               │
│     └─ Extract S3 URLs                                       │
│                                                               │
│  4. Clean URL                                                │
│     └─ Strip query parameters (X-Amz-* tokens)              │
│                                                               │
│  5. Download PDF (wget with auth cookies)                    │
│     └─ Save to: /share/lunch-menu/menu.pdf                  │
│                                                               │
│  6. Trigger Home Assistant                                   │
│     └─ POST /api/webhook/lunch_menu_downloaded              │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Home Assistant                            │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  1. Receive Webhook                                          │
│                                                               │
│  2. Execute Shell Command                                    │
│     └─ cp /share/lunch-menu/menu.pdf \                      │
│           /config/www/lunch-menu/menu.pdf                   │
│                                                               │
│  3. Send Notification                                        │
│     └─ "New lunch menu available!"                          │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Dashboard                               │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  • Displays: /local/lunch-menu/menu.pdf                     │
│  • Uses: PDF.js for rendering                               │
│  • Refresh button reloads page                              │
│  • Perfect for kiosk mode tablet                            │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Guide

### Problem: "No file created"

**Check:**
```bash
ls -la /share/lunch-menu/
ls -la /config/www/lunch-menu/
```

**Fix:** File might be in `/share` but not copied to `/config/www`. Check Home Assistant automation logs.

---

### Problem: "File exists but is blank"

**Check first 200 bytes:**
```bash
head -c 200 /share/lunch-menu/menu.pdf
```

**If you see HTML:** You're downloading an error page!
- Check authentication cookies are being passed to wget
- Check URL is stripped of query parameters
- Verify login is successful

**If you see `%PDF`:** It's a real PDF!
- Problem is elsewhere (viewer, permissions, etc.)

---

### Problem: "Authentication fails"

**Check:**
1. Username/password correct?
2. CSRF token being extracted?
3. Cookies being URL-encoded?
4. User-Agent header complete?

**Debug:**
```javascript
node.warn("Token: " + token);
node.warn("Cookies: " + JSON.stringify(responseCookies));
node.warn("Redirect URL: " + responseUrl);
```

---

### Problem: "Webhook not triggering"

**Check webhook URL:**
```
http://homeassistant:8123/api/webhook/lunch_menu_downloaded
```

**Test manually:**
```bash
curl -X POST http://homeassistant:8123/api/webhook/lunch_menu_downloaded
```

**Check automation:** Home Assistant → Settings → Automations

---

## Performance & Reliability

**Monthly Schedule:**
```
Runs: 1st of each month at 6:00 AM
Duration: ~5-10 seconds
Resources: Minimal
```

**Failure Modes:**
- school portal website changes → Update selectors
- Authentication changes → Update login flow
- PDF URL structure changes → Update regex
- Filesystem issues → Check permissions

**Monitoring:**
- Home Assistant notification on success
- Node-RED debug panel shows all steps
- Check file timestamp: `ls -la /config/www/lunch-menu/menu.pdf`

---

## Future Enhancements

**Possible improvements:**

1. **Multi-school support** - Download menus for multiple schools
2. **Email notifications** - Send PDF via email when updated
3. **Calendar integration** - Parse menu and add to calendar
4. **Dietary filtering** - Highlight allergen-free options
5. **Historical archive** - Keep all past menus
6. **Meal planning** - Extract ingredients, create shopping lists

---

## Conclusion

What started as "just download a PDF" turned into a deep dive through:
- Web scraping and authentication
- Binary file handling
- Container filesystem isolation
- AWS S3 signed URLs
- Node-RED limitations
- Home Assistant integration

**Final Stats:**
- **Time invested:** Many hours of debugging
- **Lines of code:** ~200 (Node-RED + Home Assistant)
- **Coffee consumed:** Too much ☕
- **"Why isn't this working?!" moments:** Countless
- **Satisfaction when it finally worked:** Priceless! 🎉

**The Result:** 
A fully automated system that downloads and displays the school lunch menu every month, requiring zero manual intervention.

**Was it worth it?**

Could I have just manually downloaded the PDF once a month? Sure.

But where's the fun in that? 😄

---

## Resources

**Files:**
- `lunch-menu-FINAL-COMPLETE.json` - Complete Node-RED flow
- `dashboard-minimal.html` - Ultra-minimal dashboard
- `configuration.yaml` - Home Assistant config snippets

**Technologies Used:**
- Home Assistant OS
- Node-RED
- PDF.js
- wget
- Bash scripting
- JavaScript

**Helpful Links:**
- [Node-RED Documentation](https://nodered.org/docs/)
- [Home Assistant Webhooks](https://www.home-assistant.io/docs/automation/trigger/#webhook-trigger)
- [PDF.js](https://mozilla.github.io/pdf.js/)

---

## Acknowledgments

Thanks to:
- The Home Assistant community
- Node-RED developers
- Claude (for endless patience during debugging)
- My child's school (for having predictable lunch menus)
- Coffee ☕

---

**Written by:** Travis  
**Date:** February 2026  
**Status:** ✅ Working perfectly!  
**Next Challenge:** Probably something equally "simple" 😅

---

*If you found this helpful (or entertaining), feel free to share! If you're attempting something similar, may your debugging journey be shorter than mine!* 🚀
