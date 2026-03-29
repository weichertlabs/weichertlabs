---
title: Installing Home Assistant OS on Raspberry Pi 5
description: How to install Home Assistant OS on a Raspberry Pi 5 using either an SD card or a USB SSD — from flashing the image to the first login.
date: 2025-03-29
tags: [home-assistant, raspberry-pi, haos, iot, smart-home]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Home Assistant OS (HAOS) is the recommended way to run Home Assistant — it's a minimal Linux-based operating system built specifically for it. You get automatic updates, add-on support, and a clean supervisor layer that handles everything behind the scenes. You don't need to touch the OS at all; Home Assistant just runs.

This guide walks through installing HAOS on a Raspberry Pi 5, covering both SD card and USB SSD as storage options.

## What You'll Need

**Hardware:**
- Raspberry Pi 5 (4GB or 8GB)
- Power supply (the official 27W USB-C supply is recommended for Pi 5)
- A storage option — either:
  - **SD card** — 32GB or larger, Class 10 / A2 rated
  - **USB SSD** — any USB 3.0 SSD enclosure with a 2.5" or M.2 SSD inside
- Ethernet cable (strongly recommended for the initial setup)
- A computer to flash the image from

**Software:**
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/) — free, available for Windows, macOS, and Linux

## SD Card vs USB SSD — Which Should You Use?

Both work, but they have real differences worth knowing about.

An **SD card** is the simplest option — plug it in and you're done. The downside is that SD cards wear out from the constant read/write activity that Home Assistant generates. A cheap card might fail within a year or two. If you go this route, use a quality card (Samsung Pro Endurance or SanDisk Endurance series are commonly recommended for this use case).

A **USB SSD** is more reliable long-term and noticeably faster, especially as your Home Assistant setup grows with more devices, automations, and history. The Pi 5 has USB 3.0 ports, so a decent SSD will make a real difference. The trade-off is a bit more setup — the Pi 5 may need its bootloader updated to prioritize USB boot.

For a long-running home automation setup, the SSD is the better choice. For a quick start or a test setup, the SD card is perfectly fine.

## Step 1 – Flash the Image

Open Raspberry Pi Imager on your computer.

1. Click **Choose Device** and select **Raspberry Pi 5**
2. Click **Choose OS**, scroll down and select **Other specific-purpose OS → Home assistants and home automation → Home Assistant**
3. Select **Home Assistant OS** (it will show the latest version)
4. Click **Choose Storage** and select your SD card or USB SSD
5. Click **Next**

When asked about applying OS customisation settings, click **No** — HAOS handles its own first-run configuration and doesn't use the Pi Imager customisation options.

Click **Yes** to confirm writing. The imager will download the image, flash it, and verify the write. This takes a few minutes depending on your internet speed and storage speed.

{{< callout type="info" >}}
**Tip:** If you're using a USB SSD, connect it to your computer via a USB-to-SATA or USB-to-M.2 adapter to flash it, then move it to the Pi afterwards.
{{< /callout >}}

## Step 2 – Enable USB Boot (SSD only)

If you're using an SD card, skip this step.

The Raspberry Pi 5 may not boot from USB by default — it depends on when your Pi was manufactured and what bootloader version it shipped with. The easiest way to check and update is to use the Raspberry Pi Imager.

On your computer, open Raspberry Pi Imager again:

1. Click **Choose Device → Raspberry Pi 5**
2. Click **Choose OS → Misc utility images → Bootloader → USB Boot**
3. Flash this to a separate SD card
4. Insert that SD card into the Pi, power it on, and wait about 10 seconds — the green LED will blink steadily when done
5. Power off, remove the SD card, connect your SSD, and power back on

Your Pi will now boot from USB. You only need to do this once — the bootloader setting is stored on the Pi itself.

## Step 3 – First Boot

Insert your flashed SD card or connect your SSD to the Pi. Connect an Ethernet cable, then power on.

The first boot takes a few minutes — Home Assistant needs to expand the filesystem and pull down the latest version of the Home Assistant application. Don't interrupt it.

You can follow the progress by navigating to:

```
http://homeassistant.local:8123
```

If that address doesn't resolve on your network, find your Pi's IP address from your router's device list and use that instead:

```
http://192.168.x.x:8123
```

When the loading screen appears, it means the OS is running and Home Assistant is initialising. Wait until you see the onboarding screen — this usually takes 5–10 minutes on the very first boot.

## Step 4 – Create Your Account

The onboarding screen walks you through:

1. **Create a user account** — this is your local admin account. Choose a strong password and write it down somewhere safe.
2. **Name your Home Assistant installation** — this is just a display name, call it whatever makes sense.
3. **Set your location** — used for sunrise/sunset automations and weather integrations. You can be approximate if you prefer.
4. **Choose what data to share** — analytics are anonymous and optional.

After completing onboarding, you're in. You'll see the default dashboard with nothing on it yet — which is expected.

## What's Next

Home Assistant is running, but it's empty. The next steps are typically:

- **Add integrations** — connect your smart home devices (Zigbee, Matter, Wi-Fi bulbs, etc.)
- **Set up Zigbee** — if you have a Zigbee USB adapter, the next guide covers pairing it with Home Assistant and adding Zigbee devices
- **Customise your dashboard** — add cards, views, and automations

This installation is also the foundation for connecting the [Hydroponic Grow System project](/projects/hydroponics/) to Home Assistant further down the line — sensor data from the ESP32 will eventually land here via MQTT.

## Related Links

- [Home Assistant OS documentation](https://www.home-assistant.io/installation/raspberrypi)
- [Raspberry Pi Imager download](https://www.raspberrypi.com/software/)
- [Home Assistant community forums](https://community.home-assistant.io/)
- [Hydroponic Grow System project](/projects/hydroponics/)
