---
title: Setting Up Zigbee with SONOFF ZBDongle-P and ZHA
description: How to connect the SONOFF Zigbee 3.0 USB Dongle Plus (ZBDongle-P) to Home Assistant using the built-in ZHA integration and start pairing Zigbee devices.
date: 2025-03-29
tags: [home-assistant, zigbee, sonoff, zha, iot, raspberry-pi]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

The SONOFF Zigbee 3.0 USB Dongle Plus (ZBDongle-P) is one of the most widely used Zigbee adapters for Home Assistant. It's based on the Texas Instruments CC2652P chip, comes pre-flashed with coordinator firmware, and is recognized by Home Assistant automatically — no drivers or manual configuration needed in most cases.

This guide uses **ZHA (Zigbee Home Automation)**, the Zigbee integration built directly into Home Assistant. It's the simplest path to get Zigbee running, and it works great with the ZBDongle-P.

## What You'll Need

- SONOFF Zigbee 3.0 USB Dongle Plus (ZBDongle-P)
- Home Assistant OS running on Raspberry Pi 5 (see [Part 1](/guides/iot/home-assistant/install-haos-raspberry-pi))
- A USB extension cable (strongly recommended — more on this below)
- At least one Zigbee device to pair

## Before You Plug It In — Two Important Tips

**Use a USB extension cable.** The Pi 5 generates enough electromagnetic interference that a dongle plugged directly into its USB port can have reduced range and reliability. A cheap 1–2 metre USB extension cable moves the antenna away from the board and makes a noticeable difference. This is recommended by SONOFF themselves and widely echoed in the Home Assistant community.

**Keep it away from Wi-Fi.** Zigbee operates on the 2.4 GHz band, the same as Wi-Fi. If your Pi is sitting right next to your router or a Wi-Fi access point, try to put some distance between the dongle antenna and the router. A metre or two is usually enough.

## Step 1 – Connect the Dongle

Screw the antenna onto the SMA connector on the dongle — finger tight is fine. Then plug it into your Raspberry Pi via the USB extension cable.

Home Assistant should detect it within a few seconds.

## Step 2 – Add the ZHA Integration

In Home Assistant, go to **Settings → Devices & Services**.

If the dongle was detected automatically, you'll see a notification at the top of the page offering to set up **Zigbee Home Automation**. Click **Configure** and follow the prompts — the serial port will already be filled in correctly.

If there's no automatic notification:

1. Click **+ Add Integration** in the bottom right
2. Search for **Zigbee Home Automation** and select it
3. On the next screen, select your serial port — it will appear as something like `/dev/ttyUSB0` or `/dev/serial/by-id/usb-ITead_Sonoff_Zigbee_3.0_USB_Dongle_Plus-...`
4. Leave the rest of the settings as defaults and click **Submit**

ZHA will initialize the Zigbee network. This takes about 30 seconds.

{{< callout type="info" >}}
**Tip:** If you don't see the dongle in the serial port list, try unplugging and replugging it, then refresh the page. If it still doesn't appear, check that the USB extension cable is working by trying a direct connection temporarily.
{{< /callout >}}

## Step 3 – Pair Your First Zigbee Device

Once ZHA is running, go to **Settings → Devices & Services → Zigbee Home Automation** and click on the integration. Then click **Add device**.

Home Assistant will open the Zigbee network for pairing for 60 seconds. During this window, put your Zigbee device into pairing mode — the exact method varies by device, but it's usually a long press on a button or a reset sequence. Check your device's manual.

When the device is found, it will appear in the list. Give it a name and assign it to a room if you want. That's it — it's now in Home Assistant.

{{< callout type="info" >}}
**Tip:** When a device joins, it will blink or beep to confirm pairing (this is called the identify procedure). If you're adding several devices in bulk and find this annoying, you can disable it under **Settings → Devices & Services → Zigbee Home Automation → Configure → Options**.
{{< /callout >}}

## Step 4 – Find Your Device

After pairing, go to **Settings → Devices & Services → Devices** and search for your device by name. Click on it to see its entities — sensors, switches, or whatever the device exposes.

You can also go to **Settings → Areas & Zones** to assign devices to rooms, which makes the dashboard and automations much easier to manage as your setup grows.

## ZHA vs Zigbee2MQTT — Which Should You Use?

You may have seen references to **Zigbee2MQTT** (Z2M) as an alternative to ZHA. Both work with the ZBDongle-P, and both are good options. Here's the short version:

**ZHA** is built into Home Assistant, simpler to set up, and requires no extra add-ons. It's a great choice for most home setups and is what this guide uses.

**Zigbee2MQTT** is a separate add-on that runs alongside Home Assistant via MQTT. It supports a wider range of devices, gives you more low-level control, and has very active community development. It's more complex to set up but worth it if you need devices that ZHA doesn't support, or if you want deeper control over your network.

For getting started, ZHA is the right call. You can always migrate to Z2M later if you need to.

## Understanding Zigbee Mesh Networking

One thing worth knowing early: Zigbee is a mesh network, not a star. Mains-powered devices (smart plugs, bulbs, wall switches) act as **routers** — they extend the network by relaying signals from other devices. Battery-powered devices (sensors, remotes) are **end devices** — they connect to the nearest router but don't relay for others.

This means the more mains-powered Zigbee devices you have, the stronger and more reliable your network becomes. A single coordinator with only battery-powered sensors at the edge of range will be less stable than a setup with a few smart plugs scattered around acting as relays.

## What's Next

With Zigbee running, the next logical steps are:

- Add more devices and organise them into rooms and areas
- Build automations — for example, turning lights on based on a motion sensor
- Eventually connect sensor data from the [Hydroponic Grow System project](/projects/hydroponics/) into Home Assistant via MQTT

## Related Links

- [ZHA integration documentation](https://www.home-assistant.io/integrations/zha/)
- [SONOFF ZBDongle-P product page](https://itead.cc/product/sonoff-zigbee-3-0-usb-dongle-plus/)
- [Zigbee2MQTT documentation](https://www.zigbee2mqtt.io/)
- [Part 1 – Installing HAOS on Raspberry Pi 5](/guides/iot/home-assistant/install-haos-raspberry-pi)
- [Hydroponic Grow System project](/projects/hydroponics/)
