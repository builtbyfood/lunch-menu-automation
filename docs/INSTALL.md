# 🚀 Final Setup Guide - Complete System

## 📦 What You're Getting

A fully automated lunch menu system:
- ✅ Downloads from school portal monthly
- ✅ Displays on tablet in kiosk mode
- ✅ Zero manual intervention required

---

## 🎯 Quick Setup (5 Steps)

### Step 1: Configure Home Assistant

**Edit `/config/configuration.yaml`:**

```yaml
# Shell command to copy file from /share to /config/www
shell_command:
  copy_lunch_menu: "cp /share/lunch-menu/menu.pdf /config/www/lunch-menu/menu.pdf"

# Panel to display dashboard
panel_iframe:
  lunch_menu:
    title: "Lunch Menu"
    icon: mdi:food-apple
    url: "/local/lunch-menu/dashboard.html"
```

**Edit `/config/automations.yaml`:**

```yaml
automation:
  - id: copy_lunch_menu_after_download
    alias: "Copy Lunch Menu to WWW"
    description: "Triggered by Node-RED webhook after PDF download"
    trigger:
      - platform: webhook
        webhook_id: lunch_menu_downloaded
    action:
      - service: shell_command.copy_lunch_menu
      - delay:
          seconds: 2
      - service: persistent_notification.create
        data:
          title: "📄 Lunch Menu Updated"
          message: "New lunch menu is now available!"
```

**Restart Home Assistant**

---

### Step 2: Import Node-RED Flow

1. **Open Node-RED** (usually at `http://homeassistant:8123/hassio/ingress/...`)
2. **Menu (☰)** → **Import**
3. **Select:** `lunch-menu-FINAL-COMPLETE.json`
4. **Click:** Import
5. **New tab appears:** "Lunch Menu - FINAL"

---

### Step 3: Configure Node-RED Flow

**Double-click "Load Credentials" node:**

```javascript
const username = 'your@email.com';  // Your school portal email
const password = 'your_password';   // Your school portal password
```

**Update school ID (if different):**

Double-click "Validate Login" node and find this line:
```javascript
msg.url = 'https://www.your-school-portal.com/schools/35860/feeds/files';
                                                    ^^^^^
                                            Your school ID here
```

To find your school ID:
1. Log into school portal
2. Look at URL: `https://www.your-school-portal.com/schools/12345/...`
3. That number is your school ID!

**Click "Deploy"**

---

### Step 4: Upload Dashboard

**Create directory:**
```bash
mkdir -p /config/www/lunch-menu
```

**Upload** `dashboard-minimal.html` and rename to:
```
/config/www/lunch-menu/dashboard.html
```

---

### Step 5: Test Everything!

**Test Node-RED flow:**
1. Click the blue button on "Manual Update" inject node
2. Watch debug panel (right side, bug icon 🐛)
3. Should see:
   ```
   🚀 Starting download for PreK-February.pdf
   ✂️ Stripped tokens from URL
   📥 Clean URL: https://...
   🌐 Downloading to /share...
   ✅ Downloaded successfully
   📢 Triggering Home Assistant to copy file...
   ```

**Verify files exist:**
```bash
ls -lh /share/lunch-menu/menu.pdf
ls -lh /config/www/lunch-menu/menu.pdf
```

Both should show ~134KB PDF files!

**Test dashboard:**

Open: `http://homeassistant:8123/local/lunch-menu/dashboard.html`

Should see the lunch menu PDF!

**Test Home Assistant panel:**

1. Restart Home Assistant
2. Click "Lunch Menu" in left sidebar
3. Should open dashboard in panel!

---

## 📅 Automation Schedule

**Automatic monthly updates:**
- Runs: 1st of each month at 6:00 AM
- Downloads: Current month's PDF
- Triggers: Home Assistant to copy file
- Notification: "Lunch Menu Updated"

**Manual trigger:**
- Click the "Manual Update" button in Node-RED anytime!

---

## 🔧 Customization

### Change School Name

**In "Load Credentials" node:**
```javascript
msg.targetFilename = "YOUR_SCHOOL_NAME PreK-" + monthNames[new Date().getMonth()] + ".pdf";
```

### Change Schedule

**In "1st of Month 6am" inject node:**
```
crontab: "00 06 01 * *"
          mm hh DD MM DOW

Examples:
"00 06 01 * *"  = 6am on 1st of month
"00 18 01 * *"  = 6pm on 1st of month  
"00 06 * * 1"   = 6am every Monday
```

### Change Dashboard Colors

**Edit `dashboard-minimal.html`:**
```css
background: linear-gradient(135deg, #4A90E2 0%, #1A4D7E 100%);
                                    ^^^^^^^^        ^^^^^^^^
                                    Medium Blue     Dark Blue

Change to any colors you want!
```

---

## 🐛 Troubleshooting

### Problem: "No file created"

**Check Node-RED debug output:**
- Look for errors in red
- Check each step completed

**Check filesystem:**
```bash
ls /share/lunch-menu/
ls /config/www/lunch-menu/
```

**Check Home Assistant automation:**
- Settings → Automations → "Copy Lunch Menu to WWW"
- Click "Run" manually to test

---

### Problem: "Blank PDF"

**Check what you actually downloaded:**
```bash
head -c 100 /share/lunch-menu/menu.pdf
```

**Should start with:** `%PDF-1.7`

**If it says:** `<!DOCTYPE html>` → You downloaded an error page!

**Fixes:**
1. Check authentication cookies are being passed
2. Verify URL tokens are stripped
3. Check school ID is correct

---

### Problem: "Webhook not triggering"

**Test webhook manually:**
```bash
curl -X POST http://homeassistant:8123/api/webhook/lunch_menu_downloaded
```

**Check automation exists:**
- Settings → Automations
- Look for "Copy Lunch Menu to WWW"

**Check webhook ID matches:**
- Node-RED: `lunch_menu_downloaded`
- Home Assistant: `webhook_id: lunch_menu_downloaded`

---

## 📱 Kiosk Mode Setup

**For Android tablet:**

1. Install "Fully Kiosk Browser"
2. Set start URL: `http://homeassistant:8123/local/lunch-menu/dashboard.html`
3. Enable kiosk mode
4. Disable screensaver
5. Done!

**For iPad:**

1. Safari → Open dashboard
2. Share → Add to Home Screen
3. Settings → Accessibility → Guided Access
4. Enable Guided Access
5. Triple-click home to start

---

## 🎉 You're Done!

Your automated lunch menu system is now:
- ✅ Downloading PDFs automatically
- ✅ Copying to web-accessible location
- ✅ Displaying on tablet
- ✅ Updating monthly
- ✅ Sending notifications

**Enjoy never having to manually download lunch menus again!** 🎊

---

## 📊 System Architecture

```
Monthly Schedule
      ↓
Node-RED (6:00 AM on 1st)
      ↓
1. Login to school portal
2. Find PDF URL
3. Strip tokens
4. Download with wget to /share
5. Trigger webhook
      ↓
Home Assistant
      ↓
1. Receive webhook
2. Copy file to /config/www
3. Send notification
      ↓
Dashboard
      ↓
Display PDF on tablet!
```

---

## 🆘 Support

**If you run into issues:**

1. Check the complete blog post (COMPLETE-BLOG-POST.md)
2. Review troubleshooting section above
3. Check Node-RED debug output
4. Verify all configurations match this guide

**Common mistakes:**
- ❌ Forgot to update password in Node-RED
- ❌ Wrong school ID
- ❌ Webhook ID mismatch
- ❌ Dashboard not uploaded to correct location
- ❌ Didn't restart Home Assistant after config changes

---

**Happy automating!** 🚀
