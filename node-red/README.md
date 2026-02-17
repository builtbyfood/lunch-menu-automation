# Node-RED Flow

This folder contains the Node-RED flow for automating lunch menu downloads.

## 📁 Files

### `lunch-menu-flow.json`
Complete Node-RED flow with all nodes configured.

## 🚀 Quick Import

### Step 1: Open Node-RED

Access Node-RED (usually at one of these):
- `http://homeassistant:8123/hassio/ingress/...` (Home Assistant addon)
- `http://your-server:1880` (standalone)

### Step 2: Import Flow

1. Click the hamburger menu (☰) in top-right
2. Select **Import**
3. Click **select a file to import**
4. Choose `lunch-menu-flow.json`
5. Click **Import**

### Step 3: Configure Credentials

**Double-click the "Load Credentials" node** and update:

```javascript
const username = 'your-email@example.com';  // Your school portal email
const password = 'your-password-here';      // Your school portal password
```

**IMPORTANT:** Click **Done** to save changes!

### Step 4: Configure School ID

**Double-click the "Validate Login" node** and find this line:

```javascript
msg.url = 'https://your-school-portal.com/schools/YOUR_SCHOOL_ID/feeds/files';
```

Replace `YOUR_SCHOOL_ID` with your actual school ID.

**How to find your school ID:**
1. Log into your school portal
2. Look at the URL bar
3. Find the number after `/schools/`
   - Example: `https://portal.com/schools/35860/...`
   - School ID = `35860`

### Step 5: Deploy

Click the red **Deploy** button in top-right.

### Step 6: Test

Click the blue button on the **Manual Update** inject node.

Watch the debug panel (right side) for progress messages:
- 🚀 Starting download
- ✂️ Stripped tokens from URL
- 📥 Clean URL
- ✅ Downloaded successfully

## 🔧 Flow Overview

```
┌────────────────┐
│ Manual Update  │  ← Click to test
│ 1st of Month   │  ← Automatic monthly
└────────┬───────┘
         │
         ▼
┌────────────────┐
│ Load Creds     │  ← Your email/password
└────────┬───────┘
         │
         ▼
┌────────────────┐
│ Prepare Signin │  ← GET /signin page
└────────┬───────┘
         │
         ▼
┌────────────────┐
│ HTTP: Signin   │  ← Fetch login page
└────────┬───────┘
         │
         ▼
┌────────────────┐
│ Extract Token  │  ← Parse CSRF token
└────────┬───────┘
         │
         ▼
┌────────────────┐
│ POST Login     │  ← Submit credentials
└────────┬───────┘
         │
         ▼
┌────────────────┐
│ Extract Cookies│  ← Get auth cookies
└────────┬───────┘
         │
         ▼
┌────────────────┐
│ Get Menu Page  │  ← Access files page
└────────┬───────┘
         │
         ▼
┌────────────────┐
│ Parse Menu     │  ← Find PDF URLs
└────────┬───────┘
         │
         ▼
┌────────────────┐
│ Download (wget)│  ← Download PDF
└────────┬───────┘
         │
         ▼
┌────────────────┐
│ Trigger HA     │  ← Call webhook
└────────────────┘
```

## ⚙️ Customization

### Change Target Filename

In **Load Credentials** node, modify:

```javascript
msg.targetFilename = "Grade -" + monthNames[new Date().getMonth()] + ".pdf";
                     ^^^^^^
                     Change to your grade/class
```

### Change Schedule

Double-click **1st of Month 6am** inject node:

```
Repeat: none
Using cron expression: 00 06 01 * *
                       mm hh DD MM DOW
```

Examples:
- `00 06 01 * *` = 6am on 1st of month
- `00 18 01 * *` = 6pm on 1st of month
- `00 06 * * 1` = 6am every Monday

### Change Portal URL

Update these nodes with your portal's URL:
- **Prepare Signin**: `msg.url = 'https://your-portal.com/signin'`
- **Extract & Login**: `msg.url = 'https://your-portal.com/sessions'`
- **Validate Login**: `msg.url = 'https://your-portal.com/schools/ID/feeds/files'`

## 🐛 Troubleshooting

### "No file created"

**Check:**
1. Debug panel for error messages
2. Node-RED logs
3. Credentials are correct
4. School ID matches

### "Authentication failed"

**Check:**
1. Username/password correct
2. CSRF token extracted (check debug)
3. Cookies being saved (check debug)
4. Portal URL is correct

### "PDF is blank/corrupt"

**Check:**
1. Using wget (not HTTP Request node)
2. URL tokens are stripped
3. Auth cookies being passed
4. File size is correct (~130KB+)

**Verify PDF:**
```bash
head -c 8 /share/lunch-menu/menu.pdf
# Should show: %PDF-1.7
```

### "Webhook not triggering"

**Check:**
1. Webhook URL in flow matches automation
2. Home Assistant is accessible
3. Test webhook manually:
   ```bash
   curl -X POST http://homeassistant:8123/api/webhook/lunch_menu_downloaded
   ```

## 📊 Debug Messages

**Expected output when working:**

```
🚀 Starting download for February.pdf
✂️ Stripped tokens from URL
📥 Clean URL: https://s3.amazonaws.com/.../February.pdf
🌐 Downloading to /share...
✅ Downloaded successfully
📢 Triggering Home Assistant to copy file...
```

## 🔒 Security Notes

**Credentials:**
- Store in Node-RED credentials (encrypted)
- Or use environment variables
- Never commit flow with real passwords!

**Before sharing flow:**
```bash
# Export flow
# Replace credentials with placeholders
# Verify no sensitive data
```

## 📚 Node-RED Resources

- [Node-RED Documentation](https://nodered.org/docs/)
- [Node-RED Cookbook](https://cookbook.nodered.org/)
- [Function Node Guide](https://nodered.org/docs/user-guide/writing-functions)
- [HTTP Request Node](https://cookbook.nodered.org/http/)

## ⬆️ Updating the Flow

To update to a new version:

1. Export your current flow (backup!)
2. Note your credentials and school ID
3. Import new flow
4. Re-enter credentials
5. Re-enter school ID
6. Deploy and test
