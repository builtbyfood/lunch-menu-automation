# Dashboard

This folder contains the HTML dashboard for displaying lunch menus.

## 📁 Files

### `dashboard.html`
Minimal, clean dashboard that displays the lunch menu PDF.

## 🚀 Installation

### Step 1: Create Directory

```bash
mkdir -p /config/www/lunch-menu
```

### Step 2: Copy Dashboard

```bash
cp dashboard.html /config/www/lunch-menu/dashboard.html
```

### Step 3: Access Dashboard

**Direct URL:**
```
http://homeassistant:8123/local/lunch-menu/dashboard.html
```

**Via Home Assistant Panel** (if configured):
- Click "Lunch Menu" in left sidebar

## ✨ Features

- 📄 **PDF Display** - Uses PDF.js for universal browser support
- 🔄 **Floating Refresh Button** - Easy to tap on tablets
- ◀▶ **Page Navigation** - For multi-page menus
- 📱 **Mobile Friendly** - Responsive design
- 🎨 **Blue Gradient** - Clean, modern appearance
- 💾 **Fit to Width** - PDF automatically sized to screen

## 🎨 Customization

### Change Colors

Edit the `<style>` section:

```css
body {
    background: linear-gradient(135deg, #4A90E2 0%, #1A4D7E 100%);
                                        ^^^^^^^^        ^^^^^^^^
                                        Light Blue      Dark Blue
}
```

**Color scheme suggestions:**
- Green: `#2ECC71` to `#27AE60`
- Purple: `#9B59B6` to `#8E44AD`
- Red: `#E74C3C` to `#C0392B`
- Orange: `#E67E22` to `#D35400`

### Change Page Title

```html
<title>Lunch Menu</title>
       ^^^^^^^^^^^
       Change this
```

### Hide Page Navigation

If you always have single-page menus:

```css
.page-nav {
    display: none;  /* Add this line */
}
```

### Adjust Refresh Button Position

```css
.refresh-btn {
    bottom: 20px;  /* Distance from bottom */
    right: 20px;   /* Distance from right */
}
```

### Change Refresh Button Size

```css
.refresh-btn {
    width: 60px;   /* Make bigger: 80px */
    height: 60px;  /* Make bigger: 80px */
    font-size: 1.5em;  /* Icon size: 2em */
}
```

## 📱 Kiosk Mode Setup

### Android Tablet

**Using Fully Kiosk Browser:**

1. Install Fully Kiosk Browser from Play Store
2. Settings → Start URL:
   ```
   http://homeassistant:8123/local/lunch-menu/dashboard.html
   ```
3. Enable kiosk mode
4. Disable screensaver
5. Set device homepage

**Using Chrome:**

1. Add to Home Screen
2. Open from home screen (full screen mode)
3. Settings → Display → Screen timeout → Never

### iPad/iPhone

**Using Safari:**

1. Open dashboard URL
2. Share → Add to Home Screen
3. Open from home screen
4. Settings → Display → Auto-Lock → Never

**Using Guided Access:**

1. Settings → Accessibility → Guided Access
2. Enable Guided Access
3. Open dashboard
4. Triple-click home button to start

### Amazon Fire Tablet

**Using Silk Browser:**

1. Open dashboard
2. Menu → Add to Home
3. Settings → Display → Screen Timeout → Never

## 🐛 Troubleshooting

### PDF Not Loading

**Check:**
1. File exists at `/config/www/lunch-menu/menu.pdf`
2. File is actually a PDF:
   ```bash
   file /config/www/lunch-menu/menu.pdf
   ```
3. File permissions are correct
4. Clear browser cache

### Blank Screen

**Check:**
1. Browser console for errors (F12)
2. PDF.js loaded correctly (check network tab)
3. PDF file size > 0
4. PDF URL is accessible

### Navigation Buttons Don't Work

**Check:**
1. PDF has multiple pages
2. JavaScript enabled in browser
3. No JavaScript errors in console

### Refresh Button Missing

**Check:**
1. CSS loaded correctly
2. Scroll down (might be off-screen)
3. Check browser compatibility

## 🎯 Browser Compatibility

**Tested and working:**
- ✅ Chrome (desktop & mobile)
- ✅ Firefox (desktop & mobile)
- ✅ Safari (desktop & mobile)
- ✅ Edge (desktop)
- ✅ Samsung Internet
- ✅ Fully Kiosk Browser

**Known issues:**
- ⚠️ IE11: Not supported (use Edge)
- ⚠️ Very old browsers: May not support PDF.js

## 🔧 Advanced Customization

### Add Logo

```html
<div class="header">
    <img src="/local/lunch-menu/logo.png" alt="Logo" style="height: 50px;">
    <h1>Lunch Menu</h1>
</div>
```

### Add Date Display

```html
<div class="info-bar">
    <div class="info-item">
        <div class="info-label">Current Month</div>
        <div class="info-value" id="current-month"></div>
    </div>
</div>

<script>
    document.getElementById('current-month').textContent = 
        new Date().toLocaleDateString('en-US', { month: 'long', year: 'numeric' });
</script>
```

### Auto-Refresh Daily

```javascript
// Add to <script> section
setInterval(function() {
    const now = new Date();
    if (now.getHours() === 6 && now.getMinutes() === 30) {
        location.reload();
    }
}, 60000);  // Check every minute
```

### Print Button

```html
<button class="print-btn" onclick="window.print()">🖨️ Print</button>
```

```css
.print-btn {
    position: fixed;
    bottom: 20px;
    left: 20px;
    /* Same style as refresh-btn */
}
```

## 📊 File Structure

```
/config/www/lunch-menu/
├── dashboard.html     # This file
└── menu.pdf          # Downloaded PDF (created by automation)
```

## 🔗 Related Files

- **Home Assistant Config:** `../home-assistant/configuration.yaml.example`
- **Node-RED Flow:** `../node-red/lunch-menu-flow.json`
- **Documentation:** `../docs/`

## 📚 Resources

- [PDF.js Documentation](https://mozilla.github.io/pdf.js/)
- [Responsive Web Design](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Responsive_Design)
- [Web App Manifests](https://developer.mozilla.org/en-US/docs/Web/Manifest)
