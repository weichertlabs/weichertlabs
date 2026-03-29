---
title: Hydroponic Grow System
description: Building a fully automated hydroponic grow system using Arduino, ESP32, Home Assistant, and eventually a local AI for plant monitoring and advice.
date: 2025-03-29
tags: [arduino, esp32, home-assistant, hydroponics, ai, iot]
---

This is a long-term build project with a few different goals layered on top of each other. The short version: grow plants with as little manual effort as possible, keep the budget low, and see how far automation can realistically go.

The longer version is below.

## What This Project Is About

Hydroponic growing means plants grow in water rather than soil, with nutrients delivered directly to the roots. It's efficient, space-friendly, and surprisingly approachable — but it involves a lot of variables to keep track of: water temperature, pH, nutrient levels, light cycles, pump schedules, and more.

The goal here is to automate as much of that as possible using hardware and software I'm already familiar with or want to learn:

- **Arduino & ESP32** for reading sensors and controlling hardware
- **Home Assistant** for automation logic, dashboards, and alerts
- **Local AI (Ollama + LLaVA or similar)** down the line — to analyze sensor data and camera images of the plants and give actual useful feedback

## Goals

- Learn Arduino and ESP32 hands-on, by building something real
- Build a functional automated grow system on a tight budget
- Integrate with Home Assistant for monitoring and control
- Eventually feed sensor data and plant images to a local AI model for insights and recommendations
- Document everything — including what doesn't work

## What This Project Is Not

This is not a guide on how to build the physical grow system itself — the tanks, grow beds, lighting rigs, and plumbing. That side of things may end up documented elsewhere. This project focuses on the electronics, code, and automation layer.

## Roadmap

| Part | Topic | Status |
|------|-------|--------|
| Part 1 | Arduino intro — LED, breadboard, first sketch | ✅ Published |
| Part 2 | Reading sensors — temperature & humidity with DHT22 | 🔜 Planned |
| Part 3 | ESP32 + Wi-Fi — sending sensor data over the network | 🔜 Planned |
| Part 4 | Home Assistant integration — MQTT and dashboards | 🔜 Planned |
| Part 5 | Pump and relay control — automating watering | 🔜 Planned |
| Part 6 | Local AI integration — plant monitoring with Ollama | 🔜 Planned |

The roadmap will evolve as the project does. Some parts may get split up, merged, or reordered based on what makes sense when actually building it.

## Hardware Used (So Far)

- Arduino Uno (or compatible clone)
- Breadboard + jumper wires
- Various sensors — DHT22, pH sensor, water temp (DS18B20)
- ESP32 development board
- Relay module for pump control
- Raspberry Pi 5 running Home Assistant OS
- Minisforum BD770i (local AI host — Ubuntu, RTX 4080 Super)

{{< cards >}}
  {{< card link="part-01-arduino-intro" title="Part 1 – Arduino Intro" icon="terminal" subtitle="LED, breadboard, and your first sketch" >}}
{{< /cards >}}
