# Home Assistant Automations

This repository contains Home Assistant configuration files and automations for managing a smart home system using [Home Assistant](https://www.home-assistant.io/).

## 📁 Repository Structure

```
.
├── configuration.yaml           # Main Home Assistant configuration file
├── automations.yaml             # UI-managed automations
├── scripts.yaml                 # Reusable script definitions
├── scenes.yaml                  # Scene definitions
├── groups.yaml                  # Group configurations
├── customize.yaml               # Entity customizations
├── secrets.yaml.example         # Template for sensitive data (copy to secrets.yaml)
├── automations/                 # Modular automation files
│   ├── lights.yaml             # Lighting automations
│   └── notifications.yaml      # Notification automations
├── scripts/                     # Modular script files
│   └── routines.yaml           # Daily routine scripts
└── packages/                    # Feature-based configuration packages
    └── bedroom.yaml            # Bedroom-specific configuration
```

## 🚀 Getting Started

### Prerequisites

- [Home Assistant](https://www.home-assistant.io/installation/) installed and running
- Basic understanding of YAML syntax
- Access to your Home Assistant configuration directory

### Installation

1. **Clone this repository** to your Home Assistant configuration directory:
   ```bash
   cd /config
   git clone https://github.com/dmitriy-ryazantsev/ha.git
   ```

2. **Configure secrets** (copy and edit the example file):
   ```bash
   cp secrets.yaml.example secrets.yaml
   nano secrets.yaml  # Edit with your actual values
   ```

3. **Validate configuration**:
   - Go to Home Assistant → Developer Tools → "Check Configuration"
   - Or use the CLI: `ha core check`

4. **Restart Home Assistant** to load the new configuration:
   - Go to Configuration → Server Controls → Restart
   - Or use the CLI: `ha core restart`

## 📝 Configuration Files

### Main Configuration (`configuration.yaml`)

The main entry point that includes all other configuration files using `!include` directives. This modular approach keeps configurations organized and maintainable.

### Automations

- **`automations.yaml`**: Managed by the Home Assistant UI automation editor. Avoid manual edits.
- **`automations/`**: Manual automation files organized by functionality:
  - `lights.yaml`: Lighting control automations
  - `notifications.yaml`: Notification and alert automations

### Scripts

Reusable sequences that can be called from automations or manually triggered:
- **`scripts.yaml`**: UI-managed scripts
- **`scripts/`**: Manual script definitions organized by purpose

### Packages

Self-contained configuration modules that group related entities, automations, and scripts:
- **`packages/bedroom.yaml`**: Example package for bedroom-related configuration

## 🔒 Security

**Important:** Never commit `secrets.yaml` to version control!

- Keep sensitive data (passwords, API keys, coordinates) in `secrets.yaml`
- Use the `secrets.yaml.example` template as a reference
- The `.gitignore` file is configured to exclude `secrets.yaml`

### Using Secrets

Reference secrets in your configuration files:
```yaml
# In any YAML file:
api_key: !secret openweathermap_api_key
latitude: !secret ha_latitude
```

## 📚 Example Automations

### Lights at Sunset
Automatically turn on lights 30 minutes before sunset.

### Bedtime Routine
Turn off all lights at 11 PM.

### Motion-Based Lighting
Turn on bedroom lights when motion is detected in low light conditions.

## 🛠️ Customization

### Adding New Automations

1. **Via UI**: Use the Home Assistant automation editor (recommended for beginners)
2. **Manually**: Create a new YAML file in the `automations/` directory

Example:
```yaml
# automations/security.yaml
- alias: "Alert on door open"
  trigger:
    - platform: state
      entity_id: binary_sensor.front_door
      to: "on"
  action:
    - service: notify.mobile_app
      data:
        message: "Front door opened!"
```

### Creating Packages

Organize related configurations into packages:

```yaml
# packages/kitchen.yaml
sensor:
  - platform: template
    sensors:
      kitchen_temperature:
        value_template: "{{ states('sensor.kitchen_temp') }}"

automation:
  - alias: "Kitchen lights on"
    trigger:
      - platform: state
        entity_id: binary_sensor.kitchen_motion
        to: "on"
    action:
      - service: light.turn_on
        target:
          entity_id: light.kitchen
```

## 🧪 Testing

Before deploying changes to production:

1. **Check Configuration**: Validate YAML syntax
2. **Test in Development**: Use a test Home Assistant instance if possible
3. **Review Logs**: Check Home Assistant logs for errors
4. **Gradual Rollout**: Enable new automations one at a time

## 📖 Documentation

- [Home Assistant Documentation](https://www.home-assistant.io/docs/)
- [Automation Documentation](https://www.home-assistant.io/docs/automation/)
- [YAML Syntax](https://www.home-assistant.io/docs/configuration/yaml/)
- [Templating](https://www.home-assistant.io/docs/configuration/templating/)

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Test your changes thoroughly
4. Submit a pull request with a clear description

## 📄 License

This configuration is provided as-is for personal use. Customize as needed for your smart home setup.

## 🔗 Links

- [Home Assistant Website](https://www.home-assistant.io/)
- [Home Assistant Community](https://community.home-assistant.io/)
- [Home Assistant GitHub](https://github.com/home-assistant)

---

**Note:** This is a template repository. Customize all configurations to match your specific devices, sensors, and smart home setup.
