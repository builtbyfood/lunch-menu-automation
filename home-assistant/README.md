# Home Assistant Configuration

This folder contains example configuration files for Home Assistant.

## 📁 Files

### `configuration.yaml.example`
Main configuration additions for the lunch menu system.

**What it does:**
- Defines shell command to copy PDF files
- Creates panel iframe for dashboard access

**How to use:**
1. Copy contents to your `/config/configuration.yaml`
2. Restart Home Assistant
3. "Lunch Menu" should appear in left sidebar

### `automations.yaml.example`
Automation that copies the PDF after download.

**What it does:**
- Listens for webhook from Node-RED
- Copies PDF from `/share` to `/config/www`
- Sends notification when complete

**How to use:**
1. Copy contents to your `/config/automations.yaml`
2. Restart Home Assistant
3. Automation will trigger when Node-RED calls webhook

### `secrets.yaml.example`
Template for storing sensitive information.

**What it does:**
- Stores credentials securely
- Keeps sensitive data out of main config

**How to use:**
1. Copy to `/config/secrets.yaml`
2. Fill in your actual values
3. **NEVER commit to git!**

## 🚀 Quick Setup

### Step 1: Copy Configuration Files

```bash
# On your Home Assistant system
cd /config

# Copy example files (rename to remove .example)
cp automations.yaml.example automations.yaml
# Or append to existing automations.yaml

# Create secrets file
cp secrets.yaml.example secrets.yaml
# Edit secrets.yaml with your values
```

### Step 2: Edit configuration.yaml

Add these lines to your existing `/config/configuration.yaml`:

```yaml
shell_command:
  copy_lunch_menu: "cp /share/lunch-menu/menu.pdf /config/www/lunch-menu/menu.pdf"

panel_iframe:
  lunch_menu:
    title: "Lunch Menu"
    icon: mdi:food-apple
    url: "/local/lunch-menu/dashboard.html"
```

### Step 3: Restart Home Assistant

```bash
# Via UI: Settings → System → Restart
# Or via CLI:
ha core restart
```

### Step 4: Verify

- Check sidebar for "Lunch Menu" item
- Check Developer Tools → Services for `shell_command.copy_lunch_menu`
- Check Developer Tools → Events for webhook registration

## 🔧 Customization

### Change Panel Title/Icon

```yaml
panel_iframe:
  lunch_menu:
    title: "School Menu"  # Change this
    icon: mdi:silverware-fork-knife  # Or this
    url: "/local/lunch-menu/dashboard.html"
```

### Require Admin Access

```yaml
panel_iframe:
  lunch_menu:
    title: "Lunch Menu"
    icon: mdi:food-apple
    url: "/local/lunch-menu/dashboard.html"
    require_admin: true  # Add this line
```

### Add Mobile Notification

Edit the automation action:

```yaml
action:
  - service: shell_command.copy_lunch_menu
  - service: notify.mobile_app_your_phone
    data:
      title: "Lunch Menu Updated"
      message: "{{ now().strftime('%B') }} menu is ready!"
```

## 📋 Folder Structure

After setup, your Home Assistant should look like:

```
/config/
├── configuration.yaml       # Main config (edited)
├── automations.yaml         # Automations (edited or created)
├── secrets.yaml            # Secrets (created, not committed!)
└── www/
    └── lunch-menu/
        ├── dashboard.html  # Dashboard file
        └── menu.pdf        # Downloaded PDF
```

## 🐛 Troubleshooting

### Panel Doesn't Appear

- Check configuration.yaml syntax (use Check Configuration in UI)
- Restart Home Assistant
- Clear browser cache

### Automation Not Triggering

- Check webhook ID matches Node-RED
- Test webhook manually:
  ```bash
  curl -X POST http://homeassistant:8123/api/webhook/lunch_menu_downloaded
  ```
- Check automation logs in UI

### PDF Not Copying

- Verify shell command syntax
- Check file permissions
- Test command manually:
  ```bash
  cp /share/lunch-menu/menu.pdf /config/www/lunch-menu/menu.pdf
  ```

## 🔒 Security Notes

- **Never** commit `secrets.yaml` to git
- Use strong passwords in secrets
- Consider enabling 2FA on your school portal
- Restrict panel access if needed (`require_admin: true`)

## ℹ️ More Information

- [Home Assistant Documentation](https://www.home-assistant.io/docs/)
- [Shell Command](https://www.home-assistant.io/integrations/shell_command/)
- [Panel iFrame](https://www.home-assistant.io/integrations/panel_iframe/)
- [Automations](https://www.home-assistant.io/docs/automation/)
- [Webhooks](https://www.home-assistant.io/docs/automation/trigger/#webhook-trigger)
