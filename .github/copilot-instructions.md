# Home Assistant Automation Repository

## Project Overview

This repository contains Home Assistant automation blueprints written in YAML. Blueprints are reusable automation templates that users can import and configure through the Home Assistant UI.

## File Structure & Conventions

- **`automation/`** - Contains blueprint YAML files. Each file is a standalone automation blueprint.
- **Blueprint naming** - Use descriptive snake_case names that indicate trigger → action → modifiers (e.g., `motion_light_switch_day_night.yaml`)

## Blueprint Architecture

### Required Structure

Every blueprint YAML must follow this structure:

```yaml
blueprint:
  name: Human-readable name
  description: Detailed multi-line description
  domain: automation
  input:
    # Input parameters defined here

trigger:
  # Trigger definitions

action:
  # Action sequences
```

### Input Parameter Patterns

**Entity selectors** - Use specific domains to restrict entity types:

```yaml
input:
  motion_sensor:
    selector:
      entity:
        domain: binary_sensor # Single domain

  primary_entities:
    selector:
      entity:
        multiple: true
        domain:
          - light
          - switch
```

**Time-based overrides** - When allowing custom time ranges, provide both boolean toggle and time inputs with clear defaults:

```yaml
use_custom_times:
  default: false
  selector:
    boolean:

day_start_time:
  default: "07:00:00"
  selector:
    time:
```

### Jinja2 Template Patterns

**Entity filtering** - Split entity lists by domain using regex matching:

```yaml
variables:
  all_entities: !input entity_list
  lights: "{{ all_entities | select('match', '^light\\.') | list }}"
  switches: "{{ all_entities | select('match', '^switch\\.') | list }}"
```

**Time calculations** - Convert time to seconds for robust comparisons:

```yaml
now_s: "{{ now().hour * 3600 + now().minute * 60 + now().second }}"
day_dt: "{{ today_at(day_start) }}" # Handles both HH:MM and HH:MM:SS
day_s: "{{ day_dt.hour * 3600 + day_dt.minute * 60 + day_dt.second }}"
```

**Time range logic** - Handle wraparound midnight scenarios:

```yaml
is_day: >-
  {% set n = now_s | int %}
  {% set ds = day_s | int %}
  {% set ns = night_s | int %}
  {% if ds < ns %}
    {{ ds <= n and n < ns }}
  {% else %}
    {{ n >= ds or n < ns }}
  {% endif %}
```

### Action Sequences

**Conditional actions** - Use `choose` with empty list checks to avoid errors:

```yaml
- choose:
    - conditions:
        - condition: template
          value_template: "{{ my_list | length > 0 }}"
      sequence:
        - service: light.turn_on
          data:
            entity_id: "{{ my_list }}"
```

**Service calls** - Use domain-specific services (`light.turn_on`) instead of generic ones when accessing domain-specific attributes like `brightness_pct`.

**Multiple entity types** - Split service calls by entity type since lights and switches have different capabilities:

- Use `light.turn_on` with brightness parameters for lights
- Use `homeassistant.turn_on` for switches (no brightness)

## Common Automation Patterns

### Day/Night Behavior

Support both sun-based and time-based day/night determination:

- **Sun-based**: Check `sun.sun` entity state (`above_horizon`/`below_horizon`)
- **Time-based**: User-configurable start/end times with wraparound support
- Always provide a boolean toggle to choose between modes

### Motion-Triggered Actions

Standard pattern:

1. **Trigger on motion** → Turn on entities immediately
2. **Trigger on no motion for duration** → Turn off entities
3. Use `mode: single` to prevent overlapping executions
4. Use trigger IDs to distinguish between on/off actions

### Optional Extra Actions

When providing "extra" entity lists (e.g., additional things to turn off), use `default: []` and check list length before executing.

## Testing Considerations

- Ensure time calculations handle midnight wraparound (e.g., day starts at 22:00, ends at 06:00)
- Test with empty entity lists to verify conditional checks work
- Verify sun position logic works at different latitudes
- Test brightness_pct values at boundaries (1%, 100%)

## Common Gotchas

- **Empty lists in service calls** - Always check `| length > 0` before passing lists to services
- **Time format parsing** - Use `today_at()` which handles both `HH:MM` and `HH:MM:SS` formats
- **Trigger IDs** - Must be unique within the trigger list and referenced exactly in conditions
- **Variables scope** - Variables defined in one `choose` branch are not accessible in other branches
