# 🔒 Privacy & Security Notes

## Overview

This documentation has been sanitized to protect privacy and security. All specific identifiers have been generalized.

---

## ⚠️ What Was Changed

### Personal Information Removed:
- ❌ Specific school names → ✅ "School" / "PreK"
- ❌ School portal platform name → ✅ "school portal"
- ❌ Specific email addresses → ✅ "your-email@example.com"
- ❌ School IDs → ✅ "YOUR_SCHOOL_ID"
- ❌ Domain names → ✅ "your-school-portal.com"

### Why This Matters:
- Protects student/school privacy
- Prevents targeted attacks
- Makes guide universally applicable
- Reduces security exposure

---

## 🔐 Security Best Practices

### 1. **Credential Management**

**❌ DON'T:**
```javascript
const password = 'myActualPassword123';  // Hard-coded!
```

**✅ DO:**
- Store credentials in Home Assistant secrets
- Use environment variables
- Enable 2FA on your school portal account
- Rotate passwords regularly

**Implementation:**
```yaml
# /config/secrets.yaml
school_email: "your-email@example.com"
school_password: "your-secure-password"
```

```javascript
// In Node-RED, use environment variables
const username = env.get('SCHOOL_EMAIL');
const password = env.get('SCHOOL_PASSWORD');
```

---

### 2. **Network Security**

**Recommendations:**
- ✅ Run Home Assistant on internal network only
- ✅ Use VPN for remote access
- ✅ Enable HTTPS/SSL
- ✅ Restrict Node-RED access to local network
- ✅ Use strong WiFi passwords

**Avoid:**
- ❌ Exposing Node-RED to the internet
- ❌ Using default passwords
- ❌ Sharing flows with credentials included

---

### 3. **File Permissions**

**Check your file permissions:**
```bash
# These should NOT be world-readable
chmod 600 /config/secrets.yaml
chmod 700 /config/www/lunch-menu/

# Home Assistant should own these
chown homeassistant:homeassistant /config/www/lunch-menu/
```

---

### 4. **Webhook Security**

**Your webhook is currently unauthenticated:**
```
http://homeassistant:8123/api/webhook/lunch_menu_downloaded
```

**To secure it:**

**Option 1: Use a secret webhook ID**
```yaml
trigger:
  - platform: webhook
    webhook_id: lunch_menu_xyz123_random_secret
```

**Option 2: Validate payload**
```yaml
condition:
  - condition: template
    value_template: "{{ trigger.json.secret == 'YOUR_SECRET_KEY' }}"
```

**Option 3: IP restrictions**
```yaml
condition:
  - condition: template
    value_template: "{{ trigger.json.from == 'node-red-container' }}"
```

---

### 5. **Logging Considerations**

**What gets logged:**
- Node-RED debug messages (may contain URLs)
- Home Assistant automation traces
- System logs

**Sanitize sensitive data:**
```javascript
// ❌ Don't log full cookies
node.warn("Cookies: " + authCookies);

// ✅ Do log safely
node.warn("Cookies: " + authCookies.substring(0, 20) + "...");
```

**Disable debug nodes in production:**
- Remove or disable debug nodes after testing
- Use status messages instead for monitoring

---

### 6. **Access Control**

**Limit who can access:**

**Home Assistant Users:**
```yaml
# Restrict panel access by user
panel_iframe:
  lunch_menu:
    title: "Lunch Menu"
    icon: mdi:food-apple
    url: "/local/lunch-menu/dashboard.html"
    require_admin: false  # Change to true for admin-only
```

**Node-RED Access:**
- Enable authentication in Node-RED settings
- Use different credentials than Home Assistant
- Consider OAuth/SSO if available

---

### 7. **Third-Party Risk**

**Your flow connects to:**
- School portal (credentials transmitted)
- AWS S3 (PDF downloads)
- CDN (PDF.js library)

**Mitigations:**
- ✅ Use HTTPS everywhere (already built-in)
- ✅ Validate SSL certificates
- ✅ Host PDF.js locally instead of CDN (optional)
- ✅ Monitor for unusual activity

---

### 8. **Data Retention**

**What's stored:**
- PDF files (contain school information)
- Home Assistant logs (may contain metadata)
- Node-RED flows (contain credentials if not careful!)

**Recommendations:**
```bash
# Automatically delete old PDFs
find /config/www/lunch-menu/ -name "*.pdf" -mtime +90 -delete

# Rotate logs
logrotate /config/home-assistant.log
```

---

### 9. **Backup Security**

**If you back up Home Assistant:**

**⚠️ Your backups contain:**
- Secrets file
- Automation configurations
- Node-RED flows (possibly with credentials)

**Secure your backups:**
```bash
# Encrypt backups
gpg -c home-assistant-backup.tar.gz

# Store off-site securely
# Use encrypted cloud storage
# Or encrypted USB drive in safe location
```

---

### 10. **Disclosure Policy**

**If sharing your configuration:**

**Always remove:**
- ❌ Usernames/emails
- ❌ Passwords (obviously!)
- ❌ API keys
- ❌ Webhook URLs
- ❌ School names
- ❌ Student names
- ❌ Internal IPs/domains

**Use placeholders:**
- ✅ `YOUR_EMAIL`
- ✅ `YOUR_PASSWORD`
- ✅ `YOUR_SCHOOL_ID`
- ✅ `your-domain.com`

---

## 🎯 Quick Security Checklist

Before deploying:
- [ ] Changed default passwords
- [ ] Stored credentials in secrets.yaml
- [ ] Enabled Home Assistant authentication
- [ ] Restricted network access
- [ ] Reviewed file permissions
- [ ] Disabled debug logging in production
- [ ] Secured webhook with secret ID
- [ ] Encrypted backups
- [ ] Tested on local network only
- [ ] Reviewed logs for sensitive data

---

## 📋 Template secrets.yaml

```yaml
# /config/secrets.yaml
# Keep this file secure! Never commit to git!

school_portal_email: "your-email@example.com"
school_portal_password: "your-secure-password-here"
school_portal_url: "https://your-school-portal.com"
school_id: "12345"
webhook_secret: "random-secret-key-here"
```

**Generate secure webhook secret:**
```bash
openssl rand -hex 32
```

---

## 🚨 If Credentials Are Compromised

**Immediate steps:**
1. Change password on school portal immediately
2. Revoke any active sessions
3. Review recent login activity
4. Update credentials in Home Assistant
5. Regenerate webhook secrets
6. Check for unauthorized access to your system

---

## 📖 Additional Resources

- [Home Assistant Security Checklist](https://www.home-assistant.io/docs/configuration/securing/)
- [Node-RED Security Guide](https://nodered.org/docs/user-guide/runtime/securing-node-red)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)

---

## ✅ Final Notes

**This system is reasonably secure for home use when:**
- Run on internal network
- Protected by firewall
- Uses strong passwords
- Credentials stored properly
- Regular updates applied

**This system is NOT suitable for:**
- Public/production deployment
- Processing sensitive student data
- Compliance-regulated environments
- Multi-tenant use

**Use responsibly and at your own risk!** 🔒

---

**Remember:** Security is an ongoing process, not a one-time setup. Review regularly!
