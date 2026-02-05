---
title: ESPHome Dashboard
description: Instructions on how to integrate an ESPHome Dashboard with Home Assistant for centralized firmware update management.
ha_category:
  - Update
ha_release: "2026.1"
ha_iot_class: Local Polling
ha_config_flow: true
ha_codeowners:
  - '@pstrawder'
ha_domain: esphome_dashboard
ha_platforms:
  - update
ha_integration_type: service
ha_quality_scale: bronze
related:
  - docs: /integrations/esphome/
    title: ESPHome integration
  - url: https://esphome.io/
    title: ESPHome documentation
---

The **ESPHome Dashboard** {% term integration %} connects Home Assistant to an [ESPHome Dashboard](https://esphome.io/guides/getting_started_command_line.html#bonus-esphome-dashboard) instance, enabling you to monitor and update the firmware of all your ESPHome devices directly from Home Assistant. This integration is designed to complement the [ESPHome](/integrations/esphome/) integration by providing centralized firmware management capabilities.

When you have multiple ESPHome devices, keeping their firmware up to date can become tedious. The ESPHome Dashboard integration solves this by providing update entities for each device configured in your ESPHome Dashboard, allowing you to see at a glance which devices have pending updates and install them with a single click.

## How it works

The ESPHome Dashboard is a web-based interface for managing ESPHome device configurations. It keeps track of all your device YAML files and knows the current firmware version running on each device. When you modify a device's configuration, the dashboard detects that the deployed firmware is outdated compared to the configuration.

This integration polls the ESPHome Dashboard API to retrieve information about all configured devices, including:

- Device names and addresses
- Currently deployed firmware versions
- Available firmware versions based on the YAML configuration

For each device in your dashboard, Home Assistant creates an update entity that shows whether an update is available and allows you to trigger the update process.

## Prerequisites

Before setting up this integration, you need an ESPHome Dashboard running and accessible from your Home Assistant instance.

You can run the ESPHome Dashboard in several ways:

- **ESPHome Device Builder Add-on**: If you're using Home Assistant OS or Supervised, install the [ESPHome Device Builder Add-on](https://my.home-assistant.io/redirect/supervisor_addon/?addon=5c53de3b_esphome&repository_url=https%3A%2F%2Fgithub.com%2Fesphome%2Fhome-assistant-addon) from the add-on store. This is the recommended approach for most users.
- **Docker container**: Run the ESPHome Dashboard as a standalone Docker container.
- **Command line**: Install ESPHome via pip and run `esphome dashboard /path/to/configs`.

Make sure your dashboard is accessible over the network and note the URL (for example, `http://192.168.1.100:6052`).

{% include integrations/config_flow.md %}

{% configuration_basic %}
Dashboard URL:
  description: "The full URL to your ESPHome Dashboard, including the protocol and port. For example: `http://192.168.1.100:6052` or `http://esphome.local:6052`."
Username:
  description: "Username for dashboard authentication. Leave blank if your dashboard does not require authentication."
Password:
  description: "Password for dashboard authentication. Leave blank if your dashboard does not require authentication."
{% endconfiguration_basic %}

## Supported functionality

### Entities

The ESPHome Dashboard integration creates the following entities for each device configured in your ESPHome Dashboard.

#### Update

- **Firmware**
  - Shows whether a firmware update is available for the device
  - Displays the currently installed firmware version
  - Displays the latest available firmware version (based on the YAML configuration)
  - Allows you to install updates directly from Home Assistant

When a device in your ESPHome Dashboard is also connected to Home Assistant via the [ESPHome](/integrations/esphome/) integration, the firmware update entity automatically links to the existing device. This means you'll see the update entity alongside all other entities from that device, keeping everything organized in one place.

### Installing updates

When you install an update through this integration, the following process occurs:

1. The integration sends a compile request to the ESPHome Dashboard
2. The dashboard compiles the device's YAML configuration into firmware
3. Once compilation succeeds, the firmware is uploaded to the device via <abbr title="Over-The-Air">OTA</abbr>
4. The device restarts with the new firmware

{% note %}
The device must be online and reachable from the ESPHome Dashboard for OTA updates to work. If the device address is not available, the install feature will be disabled for that device.
{% endnote %}

## Data updates

The integration polls the ESPHome Dashboard every 5 minutes to check for device status changes and available updates. This polling interval balances staying up to date with minimizing unnecessary network traffic.

## Known limitations

- The integration requires network connectivity between Home Assistant and the ESPHome Dashboard
- OTA updates require the target device to be online and reachable from the dashboard
- The integration does not support discovering ESPHome Dashboards automatically; you must manually enter the URL
- If you rename a device in the ESPHome Dashboard, a new update entity will be created

## Troubleshooting

### Cannot connect to the dashboard

If you see a "Failed to connect to the dashboard" error during setup:

1. Verify the dashboard URL is correct and includes the protocol (`http://` or `https://`)
2. Ensure the ESPHome Dashboard is running and accessible
3. Check that there are no firewalls blocking the connection
4. Try accessing the dashboard URL directly in a web browser to confirm it's working

### Authentication failed

If you see an "Authentication failed" error:

1. Verify your username and password are correct
2. Check if your dashboard requires authentication; some setups may not require credentials
3. Try logging into the dashboard directly in a web browser with the same credentials

### Update entity shows no address

If the update entity cannot install updates because no address is available:

1. Ensure the device is online and connected to your network
2. Verify the device appears as online in the ESPHome Dashboard
3. Check that the device's YAML configuration includes the correct network settings

### Updates fail to install

If an update fails during installation:

1. Check the Home Assistant logs for detailed error messages
2. Verify the device is online and reachable from the ESPHome Dashboard
3. Try compiling and uploading manually through the ESPHome Dashboard web interface to see if there are configuration errors

## Removing the integration

This integration follows standard integration removal. No additional steps are required.

{% include integrations/remove_device_service.md %}
