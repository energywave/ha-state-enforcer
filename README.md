# State Enforcer for Home Assistant

## Overview

State Enforcer is a Home Assistant package that ensures an entity (light, switch, or other) successfully changes its state when turned on or off. If the state change fails within a defined timeout, the script retries multiple times. If it still fails, it sends a notification to the user with actionable options: **Retry** or **Cancel**.

## Features

- Ensures that an entity correctly changes state.
- Retries multiple times within a user-defined timeout.
- Sends an interactive notification if the action fails.
- Works asynchronously without blocking automations.
- Uses persistent notifications to inform the user during execution.

## Requirements

State Enforcer depends on:

- **Set State script** (for dynamic entity states): [GitHub Link](https://github.com/xannor/hass_py_set_state)
- **Multinotify** (for notifications): [GitHub Link](https://github.com/energywave/multinotify)

## Installation

1. Copy the `state_enforcer.yaml` and `state_enforcer_notifications.yaml` files to your Home Assistant packages directory.
2. Include the package in your `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

3. Customize state_enforcer_notifications.yaml with your desired devices for notifications
4. Restart Home Assistant to apply the changes.

## Usage

You can use the script in automations to ensure a light, switch, or any entity successfully turns on or off. It will retry for the specified timeout in seconds. If it fails, you will receive a notification that let you retry or cancel.

### Basic Example

```yaml
action:
  - service: script.turn_on
    target:
      entity_id: script.state_enforcer
    data:
      variables:
        entity_id: light.living_room
        action: off
        timeout: 30
```

- `entity_id`: The entity to turn on/off.
- `action`: Either `on` or `off`.
- `timeout`: (Optional) Maximum time (in seconds) to verify the state change (default: 30s).

### Example with Automation

```yaml
automation:
  - alias: "Turn on my garden lights"
    trigger:
      - platform: time
        at: "23:00:00"
    action:
      - service: script.turn_on
        target:
          entity_id: script.state_enforcer
        data:
          variables:
            entity_id: light.garden_light
            action: on
            timeout: 60
      - service: script.turn_on
        target:
          entity_id: script.state_enforcer
        data:
          variables:
            entity_id: light.porch_light
            action: on
            timeout: 60
```

This will run two separate instances of State Enforcer, one for garden light and one for porch light.
If you put other actions they'll executed in the meanwhile State Enforcer is doing it's job, it will not wait the completion of the operation (as it can take 60 seconds, if thing go wrong!)

It's something like doing it in parallel, take that in mind.

## Notifications

If the state change fails within the timeout, you will receive a notification with two options:

- **Retry**: Attempts to set the desired state again, with the same timeout.
- **Cancel**: Stops further attempts and acknowledges the failure.

Example notification:

> ⚠️ Could not turn off **Living Room Light** within 30 seconds. Current state: "on".
>
> [🔄 Retry] [❌ Cancel]

## How It Works

### 1️⃣ Execution and Initial Notification

- The script starts and creates a persistent notification indicating that the state enforcement is in progress.

### 2️⃣ Checking Existing Instances

- If another instance of the script is already running for the same entity, it is stopped before proceeding.

### 3️⃣ Attempting to Change the State

- The script sends the `homeassistant.turn_on` or `homeassistant.turn_off` command.
- It then waits **5 seconds** and checks if the state has changed.
- If not, it retries until the **timeout** is reached.

### 4️⃣ Failure Handling

- If the entity does not reach the expected state, `state_enforcer_notifications` is triggered to notify the user.
- The user can then retry the action or cancel it.

## License

This project is licensed under the MIT License.