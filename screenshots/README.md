# 📸 Screenshot Guide

This folder contains placeholder images and instructions for capturing screenshots to enhance the documentation.

---

## 🎯 Required Screenshots

### 1. System Overview (`system-overview.png`)

**What to capture:**
- Complete view of the automation in action
- Could be a diagram or actual system screenshot
- Should show: Node-RED flow → Home Assistant → Dashboard

**Suggested approach:**
- Create a diagram showing the three components
- Or screenshot your actual running system
- Dimensions: 1200x600px recommended

**Placeholder:** ![System Overview](system-overview-placeholder.png)

---

### 2. Chrome DevTools - Network Tab (`devtools-network.png`)

**What to capture:**
- Chrome DevTools with Network tab open
- "Preserve log" checkbox checked
- Several requests visible in the list
- Focus on the POST /sessions request

**Steps to capture:**
1. Open school portal login page
2. Press F12 to open DevTools
3. Click Network tab
4. Check "Preserve log"
5. Take screenshot (Ctrl+Shift+P → "Capture screenshot" in Chrome)

**Dimensions:** Full browser window (1920x1080 recommended)

**Placeholder:** ![DevTools Network](devtools-network-placeholder.png)

---

### 3. Inspecting Form Fields (`inspect-form-field.png`)

**What to capture:**
- Right-click context menu on login field
- "Inspect" option highlighted
- Elements tab showing the input field HTML

**Steps to capture:**
1. Right-click email/username field
2. Screenshot showing the context menu
3. Then screenshot Elements tab with field highlighted

**Dimensions:** Portion of screen showing form + DevTools

**Placeholder:** ![Inspect Form Field](inspect-form-field-placeholder.png)

---

### 4. CSRF Token in HTML (`csrf-token.png`)

**What to capture:**
- Elements tab with search box open
- Search term: "authenticity_token"
- Hidden input field highlighted
- Token value visible (blur it if sensitive!)

**Steps to capture:**
1. Elements tab open
2. Ctrl+F to search
3. Type "authenticity_token"
4. Screenshot the highlighted result
5. **Blur the actual token value!**

**Dimensions:** DevTools panel, 800x600px minimum

**Placeholder:** ![CSRF Token](csrf-token-placeholder.png)

---

### 5. Login Request Headers (`request-headers.png`)

**What to capture:**
- Network tab with POST /sessions selected
- Headers tab visible
- Request Headers section expanded
- Show Content-Type, Cookie, User-Agent

**Steps to capture:**
1. Network tab → POST /sessions request
2. Click Headers tab
3. Scroll to Request Headers
4. Screenshot
5. **Blur any actual cookie values!**

**Dimensions:** DevTools panel

**Placeholder:** ![Request Headers](request-headers-placeholder.png)

---

### 6. Form Data Payload (`form-data.png`)

**What to capture:**
- Same POST /sessions request
- Scroll down to Form Data section
- Shows: utf8, authenticity_token, session[email], session[password], commit
- **Blur actual email and password values!**

**Steps to capture:**
1. Still in POST /sessions request
2. Scroll to Form Data or Request Payload section
3. Screenshot
4. **Blur sensitive data!**

**Dimensions:** DevTools panel

**Placeholder:** ![Form Data](form-data-placeholder.png)

---

### 7. Response Cookies (`response-cookies.png`)

**What to capture:**
- Response Headers section
- Multiple Set-Cookie entries visible
- Shows ps_s, ps_r, ps_d cookies
- **Blur actual cookie values!**

**Steps to capture:**
1. POST /sessions request → Headers tab
2. Scroll to Response Headers
3. Find Set-Cookie entries
4. Screenshot
5. **Blur cookie values!**

**Dimensions:** DevTools panel

**Placeholder:** ![Response Cookies](response-cookies-placeholder.png)

---

### 8. PDF Links in HTML (`pdf-links-html.png`)

**What to capture:**
- Network tab → GET request to menu page
- Response tab selected
- Search box showing ".pdf"
- PDF links highlighted in HTML

**Steps to capture:**
1. Find GET request to feeds/files page
2. Click Response tab
3. Ctrl+F → search ".pdf"
4. Screenshot showing matches
5. **Can blur specific filenames if desired**

**Dimensions:** DevTools Response tab

**Placeholder:** ![PDF Links](pdf-links-html-placeholder.png)

---

### 9. Node-RED Flow (`node-red-flow.png`)

**What to capture:**
- Complete Node-RED flow layout
- All nodes visible and connected
- Clean, organized appearance
- Nodes labeled clearly

**Steps to capture:**
1. Open Node-RED
2. Select the lunch menu flow tab
3. Zoom to fit all nodes
4. Clean screenshot (no overlapping debug panels)

**Dimensions:** 1600x900px recommended

**Placeholder:** ![Node-RED Flow](node-red-flow-placeholder.png)

---

### 10. Dashboard Display (`dashboard-display.png`)

**What to capture:**
- The actual dashboard showing a lunch menu
- Clean, attractive display
- Floating refresh button visible
- Mobile or tablet view

**Steps to capture:**
1. Open dashboard URL
2. Wait for PDF to load
3. Take clean screenshot
4. Optional: Use browser responsive mode for tablet view

**Dimensions:** Tablet size (768x1024) or desktop

**Placeholder:** ![Dashboard](dashboard-display-placeholder.png)

---

### 11. Home Assistant Panel (`ha-panel.png`)

**What to capture:**
- Home Assistant sidebar with "Lunch Menu" panel
- Panel clicked/opened showing dashboard
- Clean HA interface

**Steps to capture:**
1. Open Home Assistant
2. Hover or click "Lunch Menu" in sidebar
3. Screenshot showing integration

**Dimensions:** Full browser or focused on sidebar

**Placeholder:** ![HA Panel](ha-panel-placeholder.png)

---

### 12. Debug Output Success (`debug-success.png`)

**What to capture:**
- Node-RED debug panel
- Successful flow execution messages
- Shows each step completing
- Final "SUCCESS" message

**Steps to capture:**
1. Click Manual Update
2. Wait for completion
3. Screenshot debug panel on right side
4. **Blur any URLs with tokens/IDs if present**

**Dimensions:** Debug panel area

**Placeholder:** ![Debug Success](debug-success-placeholder.png)

---

## 🎨 Screenshot Best Practices

### Quality
- **Resolution:** Minimum 1280px wide
- **Format:** PNG (lossless) preferred over JPG
- **Clarity:** Ensure text is readable
- **Lighting:** Use light theme for better visibility

### Privacy
- **Blur sensitive data:**
  - Email addresses
  - Passwords (even dots)
  - Auth tokens
  - Cookie values
  - School names (if desired)
  - Student names
  - Actual URLs with IDs

- **Safe to show:**
  - Generic UI elements
  - Field names (`session[email]`)
  - HTTP method names (POST, GET)
  - Status codes (200, 302)
  - Generic structure

### Annotations
- **Add arrows or highlights** to point out key elements
- **Use red boxes** around important parts
- **Add text labels** if helpful
- **Keep annotations minimal** - don't clutter

### Tools
- **Windows:** Snipping Tool, Snip & Sketch, ShareX
- **Mac:** Command+Shift+4
- **Linux:** Flameshot, GNOME Screenshot
- **Browser:** Chrome DevTools → Ctrl+Shift+P → "Capture screenshot"
- **Editing:** GIMP, Paint.NET, Photoshop

---

## 📁 File Naming Convention

Use descriptive names:
- ✅ `devtools-network-tab.png`
- ✅ `node-red-complete-flow.png`
- ✅ `csrf-token-highlighted.png`

Not:
- ❌ `screenshot1.png`
- ❌ `img_20260216.png`
- ❌ `untitled.png`

---

## ✅ Screenshot Checklist

Before committing screenshots:

- [ ] Image is clear and readable
- [ ] Sensitive data is blurred
- [ ] File size is reasonable (<500KB per image)
- [ ] Named descriptively
- [ ] Referenced in documentation
- [ ] Shows what it's supposed to show
- [ ] Properly sized/cropped

---

## 🔄 Updating Screenshots

When system UI changes:
1. Retake affected screenshots
2. Update with same filename
3. Commit with message: "Update screenshot: [name]"
4. Check documentation still references correctly

---

## 💡 Optional Enhancements

### Animated GIFs
For showing processes:
- Login flow animation
- Dashboard loading
- Node-RED deployment

**Tools:** ScreenToGif, LICEcap, Peek

### Video Walkthrough
Create a video tutorial:
- Upload to YouTube (unlisted if preferred)
- Embed in documentation
- More engaging than static screenshots

---

## 📝 Template for Adding New Screenshots

When adding a new screenshot:

```markdown
### X. Screenshot Name (`filename.png`)

**What to capture:**
- Description of what should be shown
- Key elements to include

**Steps to capture:**
1. Step one
2. Step two
3. Take screenshot

**Privacy notes:**
- What to blur/hide
- What's safe to show

**Dimensions:** Recommended size

**Placeholder:** ![Alt Text](filename-placeholder.png)
```

---

## 🎯 Priority Screenshots

If time is limited, focus on these first:

1. **system-overview.png** - Shows complete system
2. **node-red-flow.png** - The automation flow
3. **dashboard-display.png** - End result
4. **devtools-network.png** - Key technical detail
5. **debug-success.png** - Proof it works

The others enhance understanding but aren't critical.

---

**Remember:** Screenshots should help users understand and replicate the system. Quality and clarity matter more than quantity!
