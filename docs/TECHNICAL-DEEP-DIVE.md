# 🔍 Technical Deep Dive: Reverse Engineering the School Portal Login

## Introduction

This document provides a detailed, step-by-step walkthrough of how we reverse-engineered the authentication flow for a school parent portal. We'll cover using browser developer tools to inspect network requests, find form fields, extract tokens, and understand cookie handling.

**Tools Used:**
- Google Chrome DevTools (Network tab, Elements tab, Console)
- Node-RED for automation
- Home Assistant for integration

---

## Part 1: Initial Reconnaissance

### Step 1: Open Developer Tools

First, we need to see what's happening behind the scenes when logging into the portal.

**Instructions:**
1. Open the school portal login page in Chrome
2. Press `F12` or `Ctrl+Shift+I` (Windows/Linux) or `Cmd+Option+I` (Mac)
3. Click the **Network** tab
4. Check the **Preserve log** checkbox (important!)

**Why preserve log?** 
When you submit a login form, the page often redirects. Without "Preserve log" enabled, you'll lose all the network requests and won't be able to see what happened!

---

### Step 2: Inspect the Login Form

Before we submit anything, let's see what data the form expects.

**Instructions:**
1. Right-click on the **email/username field**
2. Select **Inspect** (or **Inspect Element**)
3. The Elements tab will open and highlight the input field

**What we're looking for:**

```html
<input type="email" name="session[email]" id="user_email" class="form-control">
```

**Key Information:**
- `name="session[email]"` - This is the field name we need to submit
- `type="email"` - Validates email format
- `id="user_email"` - We could use this for automation

**Repeat for password field:**

```html
<input type="password" name="session[password]" id="user_password" class="form-control">
```

**Key Information:**
- `name="session[password]"` - Field name for password
- `type="password"` - Masked input

---

### Step 3: Find the CSRF Token

Most modern web applications use CSRF (Cross-Site Request Forgery) tokens to prevent malicious login attempts.

**Instructions:**
1. In the Elements tab, press `Ctrl+F` to open the search box
2. Search for: `authenticity_token`
3. Find the hidden input field

**What we found:**

```html
<input type="hidden" name="authenticity_token" value="Ey_Gn3uE2ZBQ8EmyAFAMqP7vK9...">
```

**This is critical!** Every login attempt needs this token, and it changes on every page load.

**Why it matters:**
- The token proves we loaded the real login page
- It prevents automated attacks from external sites
- We MUST extract this value before logging in

**How to verify it changes:**
1. Refresh the page (F5)
2. Inspect the form again
3. Note the token value is different!

---

## Part 2: Watching the Login Process

Now let's see what happens when we actually log in.

### Step 4: Clear Network Log & Login

**Instructions:**
1. In the Network tab, click the **Clear** button (🚫 icon)
2. Make sure **Preserve log** is still checked
3. Enter your credentials in the login form
4. Click the **Sign In** button
5. Watch the Network tab fill up with requests!

---

### Step 5: Find the Login Request

The Network tab now shows all HTTP requests. Let's find the important one.

**Instructions:**
1. Look for a POST request (indicated by red or pink color)
2. Look for `/sessions` or `/sign_in` or `/auth` in the Name column
3. Click on that request

**What we see:**

```
Name: sessions
Type: document
Status: 302 (Found - this is a redirect!)
Method: POST
```

**The 302 status is good!** It means we're being redirected after login, which typically indicates success.

---

### Step 6: Inspect Request Headers

Click on the **Headers** tab for this request.

**Request Headers section shows:**

```
POST https://your-school-portal.com/sessions HTTP/1.1
Host: your-school-portal.com
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36...
Cookie: ps_s=eWJhc2U2NGVuY29kZWRkYXRh...
Referer: https://your-school-portal.com/signin
Origin: https://your-school-portal.com
```

**Key observations:**
- **Content-Type:** `application/x-www-form-urlencoded` - This tells us how the data is formatted
- **Cookie:** The signin page set a cookie, and we're sending it back
- **Referer:** Shows we came from the signin page
- **Origin:** Security check to prevent CSRF

---

### Step 7: Inspect Form Data

Scroll down to the **Request Payload** or **Form Data** section.

**What we see:**

```
utf8: ✓
authenticity_token: Ey_Gn3uE2ZBQ8EmyAFAMqP7vK9...
session[email]: your-email@example.com
session[password]: ●●●●●●●●●●
commit: Sign In
```

**Key observations:**
- ✓ The `authenticity_token` we found earlier is included!
- ✓ Email is in `session[email]` (matches the form field name)
- ✓ Password is sent (but shown as dots in DevTools)
- ✓ There's also a `utf8` field and `commit` button text

**Important:** This tells us exactly what data to send in our Node-RED flow!

---

### Step 8: Inspect the Response

Click on the **Response** tab for this request.

**Common outcomes:**

**Scenario 1: HTML Response (redirect page)**
```html
<!DOCTYPE html>
<html>
<head>
  <meta http-equiv="refresh" content="0; url=/users/44776746/contacts">
</head>
```

This is a redirect page. Note the URL in the meta tag!

**Scenario 2: 302 Redirect**
Look at the **Location** header in the Response Headers:

```
HTTP/1.1 302 Found
Location: https://your-school-portal.com/users/44776746/contacts
Set-Cookie: ps_r=...; ps_d=...; ps_s=...
```

**Key observations:**
- ✓ We're redirected to `/users/XXXXXX/` - this means login succeeded!
- ✓ Multiple cookies are being set - these are our authentication cookies!
- ✓ If we see `/signin` instead, login failed

---

### Step 9: Extract Authentication Cookies

This is the most important step! These cookies prove we're logged in.

**Instructions:**
1. Still in the **Headers** tab for the POST /sessions request
2. Scroll to **Response Headers**
3. Look for `Set-Cookie` entries

**What we see:**

```
Set-Cookie: ps_s=eWJhc2U2NGVuY29kZWRkYXRh...; Path=/; Expires=Mon, 15 Feb 2027 19:27:46 GMT; SameSite=Lax
Set-Cookie: ps_r=c29tZW90aGVyZGF0YQ...; Path=/; Expires=Mon, 15 Feb 2027 19:27:46 GMT; SameSite=Lax
Set-Cookie: ps_d=bW9yZWRhdGE...; Path=/; Expires=Mon, 15 Feb 2027 19:27:46 GMT; SameSite=Lax
Set-Cookie: request_method=; Path=/; Max-Age=0; Expires=Thu, 01 Jan 1970 00:00:00 GMT
```

**Key observations:**
- **Multiple cookies:** `ps_s`, `ps_r`, `ps_d` - all needed for authentication
- **Long expiration:** Valid for ~1 year
- **SameSite=Lax:** Security setting
- **request_method:** Being cleared (Max-Age=0)

**Critical: We need ALL of these cookies for subsequent requests!**

---

### Step 10: Test Authenticated Request

Let's verify these cookies actually work.

**Instructions:**
1. Look for the NEXT request after the POST /sessions
2. Click on it (should be a GET request to /users/... or similar)
3. Look at the **Request Headers**

**What we see:**

```
GET https://your-school-portal.com/users/44776746/contacts HTTP/1.1
Cookie: ps_s=...; ps_r=...; ps_d=...
```

**Perfect!** The browser automatically sent back all our authentication cookies.

---

## Part 3: Finding the PDF Links

Now that we're logged in, let's find where the lunch menu PDFs are.

### Step 11: Navigate to Menu Section

**Instructions:**
1. Stay in the Network tab with Preserve log enabled
2. Navigate to the lunch menu section (Files, Documents, etc.)
3. Watch for new requests

**Look for:**
- Requests to `/feeds/`, `/files/`, or `/documents/`
- Response Type: `document` or `html`

---

### Step 12: Inspect the Menu Page HTML

**Instructions:**
1. Find the GET request to the menu page (e.g., `/schools/35860/feeds/files`)
2. Click on it
3. Click the **Response** tab
4. You'll see the HTML of the page

---

### Step 13: Search for PDF Links

**Instructions:**
1. In the Response tab, press `Ctrl+F`
2. Search for: `.pdf`
3. Look at the matches

**What we found:**

```html
<a href="https://rails-parentsquare-prod.s3.amazonaws.com/feeds/BDM6L5GvTbKmVUSGyzzJ_PreK-February.pdf?response-content-disposition=inline%3Bfilename%3D%22PreK-February.pdf%22&amp;X-Amz-Expires=21600&amp;X-Amz-Date=20260215T192746Z&amp;X-Amz-Security-Token=IQoJb3JpZ2luX2VjEFsa...">
  PreK-February.pdf
</a>
```

**Key observations:**
- **S3 URL:** File is hosted on Amazon S3
- **Signed URL:** Has `X-Amz-*` parameters (more on this later!)
- **response-content-disposition:** Tells browser to display inline
- **Filename in URL:** We can search for specific months!

---

### Step 14: Understanding Signed URLs

Those `X-Amz-*` parameters are important. Let's understand them.

**What they are:**
- **X-Amz-Expires:** How long the URL is valid (21600 seconds = 6 hours)
- **X-Amz-Date:** When the URL was created
- **X-Amz-Security-Token:** Authentication token for S3
- **X-Amz-Signature:** Cryptographic signature

**The Problem:**
When we tried downloading with these parameters, we got blank PDFs!

**The Solution:**
We discovered that stripping everything after `.pdf` works better:

```
https://rails-parentsquare-prod.s3.amazonaws.com/feeds/BDM6L5GvTbKmVUSGyzzJ_PreK-February.pdf
```

**Why?** The signed tokens were causing authentication issues. The base URL + authentication cookies is sufficient!

---

## Part 4: Translating to Node-RED

Now let's convert what we learned into Node-RED flow configuration.

### Step 15: Building the Authentication Flow

Based on our findings, here's what our flow needs to do:

**1. GET /signin page**
```javascript
msg.url = 'https://your-school-portal.com/signin';
msg.method = 'GET';
```

**2. Extract CSRF token from HTML**
```javascript
const html = msg.payload;
const tokenMatch = html.match(/name="authenticity_token"[^>]*value="([^"]+)"/);
const token = tokenMatch[1];
```

**3. POST login with form data**
```javascript
const formData = new URLSearchParams();
formData.append('utf8', '✓');
formData.append('authenticity_token', token);
formData.append('session[email]', username);
formData.append('session[password]', password);
formData.append('commit', 'Sign In');

msg.payload = formData.toString();
msg.headers = {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Cookie': 'ps_s=' + encodeURIComponent(signinCookie)
};
```

**4. Extract auth cookies from redirect**
```javascript
const redirectCookies = msg.redirectList[0].cookies || {};
let allCookies = [];
for (const [key, val] of Object.entries(redirectCookies)) {
    allCookies.push(key + '=' + encodeURIComponent(val.value));
}
const cookieHeader = allCookies.join('; ');
```

**5. Use cookies for authenticated requests**
```javascript
msg.headers = {
    'Cookie': cookieHeader,
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)...'
};
```

---

### Step 16: Extracting PDF URLs

**Parse the HTML response:**

```javascript
const html = msg.payload;

// Find all PDF links
const pdfPattern = /href=["'](https:\/\/rails-parentsquare-prod\.s3\.amazonaws\.com\/feeds\/[^"']*\.pdf[^"']*?)["']/gi;
const pdfs = [];
let match;

while ((match = pdfPattern.exec(html)) !== null) {
    pdfs.push(match[1]);
}

// Find specific month
const searchName = "PreK-February.pdf";
let pdfUrl = null;
for (let i = 0; i < pdfs.length; i++) {
    if (pdfs[i].includes(searchName)) {
        pdfUrl = pdfs[i];
        break;
    }
}

// Strip query parameters
const pdfMatch = pdfUrl.match(/(https:\/\/[^?]+\.pdf)/);
pdfUrl = pdfMatch[1];  // Clean URL without tokens!
```

---

### Step 17: Downloading the PDF

**Why wget instead of HTTP Request node?**

We discovered that Node-RED's HTTP Request node was corrupting the PDF files. Using `wget` via exec node solved this:

```javascript
const pdfUrl = msg.pdfUrl;
const cookies = msg.authCookies;

msg.payload = `mkdir -p /share/lunch-menu && ` +
              `wget --header="Cookie: ${cookies}" ` +
              `--user-agent="Mozilla/5.0..." ` +
              `-O /share/lunch-menu/menu.pdf "${pdfUrl}"`;
```

**Why this works:**
- `wget` handles binary files correctly
- Passes authentication cookies in header
- Downloads directly to filesystem
- No buffer corruption issues

---

## Part 5: Common Pitfalls & Solutions

### Pitfall 1: "Access Denied" or Blank PDFs

**Symptom:** PDF downloads but opens blank or shows "Access Denied"

**Cause:** Not sending authentication cookies with the download request

**Solution:**
```bash
wget --header="Cookie: ps_s=...; ps_r=...; ps_d=..." \
     -O menu.pdf "https://url.pdf"
```

---

### Pitfall 2: "CSRF Token Invalid"

**Symptom:** Login fails with "Invalid authenticity token" error

**Cause:** 
- Token not extracted correctly
- Token expired (extracted from old page load)
- Token not URL-encoded properly

**Solution:**
- Extract token immediately before login
- Don't store tokens long-term
- Use `encodeURIComponent()` if needed

---

### Pitfall 3: Cookies Not Being Sent

**Symptom:** Logged in successfully, but subsequent requests fail

**Cause:** Cookies not extracted from redirect response

**Solution:**
Look in `msg.redirectList[0].cookies`, not `msg.responseCookies`:

```javascript
// ❌ Wrong
const cookies = msg.responseCookies;

// ✅ Correct
const cookies = msg.redirectList[0].cookies;
```

---

### Pitfall 4: File Corruption

**Symptom:** PDF file size correct but opens blank

**Cause:** File node or HTTP Request node converting binary to string

**Solution:** Use exec node with wget:
```bash
wget -O /path/to/file.pdf "https://url"
```

---

### Pitfall 5: Signed URLs Expiring

**Symptom:** PDF downloads work sometimes but fail other times

**Cause:** AWS signed URLs have expiration times

**Solution:** 
- Strip query parameters from URL
- Use authentication cookies instead
- Or download immediately after getting URL

---

## Part 6: Verification & Testing

### How to Verify Each Step

**1. Signin Page:**
```bash
curl -v https://your-school-portal.com/signin
# Look for: 200 OK, HTML with authenticity_token
```

**2. Token Extraction:**
```bash
curl -s https://your-school-portal.com/signin | grep authenticity_token
# Should output: <input ... name="authenticity_token" value="...">
```

**3. Login POST:**
```bash
curl -v -X POST https://your-school-portal.com/sessions \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "utf8=✓&authenticity_token=TOKEN&session[email]=EMAIL&session[password]=PASS&commit=Sign In"
# Look for: 302 redirect, Set-Cookie headers
```

**4. Authenticated Request:**
```bash
curl -v https://your-school-portal.com/schools/XXXXX/feeds/files \
  -H "Cookie: ps_s=...; ps_r=...; ps_d=..."
# Look for: 200 OK, HTML with PDF links
```

**5. PDF Download:**
```bash
wget --header="Cookie: ps_s=...; ps_r=...; ps_d=..." \
     -O test.pdf "https://s3-url.pdf"
# Check: file test.pdf (should say "PDF document")
```

---

## Part 7: Security Considerations

### What We Learned About Security

**1. CSRF Protection**
- Every form submission requires a fresh token
- Tokens are tied to your session
- This prevents automated attacks

**2. Cookie Security**
- SameSite=Lax prevents CSRF attacks
- HttpOnly cookies can't be accessed by JavaScript
- Secure cookies only sent over HTTPS

**3. Signed URLs**
- Temporary access to S3 files
- Expire after set time (6 hours in our case)
- Cryptographically signed

**4. Session Management**
- Cookies expire after ~1 year
- Multiple cookies work together
- All cookies required for authentication

---

### Best Practices

**DO:**
- ✅ Store credentials securely (secrets.yaml)
- ✅ Use HTTPS everywhere
- ✅ Rotate passwords regularly
- ✅ Enable 2FA if available
- ✅ Monitor for unauthorized access

**DON'T:**
- ❌ Hard-code passwords in flows
- ❌ Share flows with credentials included
- ❌ Expose Node-RED to internet
- ❌ Log full cookie values
- ❌ Reuse authentication tokens

---

## Conclusion

Through careful inspection of browser network traffic, we discovered:

1. **The login flow:** GET signin → Extract token → POST credentials → Extract cookies
2. **Form field names:** `session[email]`, `session[password]`, `authenticity_token`
3. **Cookie handling:** Multiple cookies from redirect response
4. **PDF location:** S3 URLs with signed parameters
5. **Download solution:** Strip tokens, use wget with cookies

**Key Tools:**
- Chrome DevTools Network tab (inspect requests)
- Elements tab (find form fields)
- Console (test JavaScript)
- Network → Preserve log (capture redirects)

**The Result:**
A fully automated system that downloads PDFs reliably without manual intervention!

---

## Appendix: Quick Reference

### Browser DevTools Shortcuts

| Action | Windows/Linux | Mac |
|--------|---------------|-----|
| Open DevTools | F12 or Ctrl+Shift+I | Cmd+Option+I |
| Network tab | Ctrl+Shift+E | Cmd+Option+E |
| Elements tab | Ctrl+Shift+C | Cmd+Option+C |
| Console | Ctrl+Shift+J | Cmd+Option+J |
| Search in page | Ctrl+F | Cmd+F |
| Clear network log | Ctrl+E | Cmd+E |

### Node-RED HTTP Request Settings

**For text responses:**
```
Return: a UTF-8 string
```

**For binary (PDFs):**
```
Return: a binary buffer
```

**For form data:**
```
Content-Type: application/x-www-form-urlencoded
Payload: formData.toString()
```

### Regular Expression Patterns

**Extract CSRF token:**
```javascript
/name="authenticity_token"[^>]*value="([^"]+)"/
```

**Find PDF links:**
```javascript
/href=["'](https:\/\/.*?\.pdf[^"']*?)["']/gi
```

**Strip query parameters:**
```javascript
/(https:\/\/[^?]+\.pdf)/
```

---

**End of Technical Deep Dive**

This document covered the complete reverse-engineering process from browser inspection to working automation. Use these techniques for any web scraping or automation project!
