---
excerpt: LoRa comms notes and gotchas
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - learning
  - lora
  - meshtastic
  - telecom
toc: false
toc_sticky: false
---

Hi, I am `friendlyantz`. I run `ANT*`-series nodes, like `ANTZ`, `ANTG`, `ANTS`, `ANTA` and `ANTB` in Melbourne
# power usage

| name                   | watage                                                                                                                                                                                                                                                                                                                               |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| picoWs (serie 1 and 2) | pico2-series should be more energy efficient, but perhaps this is negligible I think as most power goes to TX via LoRa module???<br><br>wattage ranges: can be 0.2W at rest. but sometimes sits around 0.6-09W. 1.5W in active zones when sending, relaying messages(normal quiet channel)<br>with wifi consuming probably 20% more. |
| Lilygo TDeck (esp32s)  | `ESP32` Should be more power hungry than `pico`, but in practice i observed same values., or even lower at 0.3-0.6W. 0.3W sleep, 0.45W screen on, whenTX wattage consumes extra to 0.5W - 0.8W for a fraction of a sec.<br><br>142mAh consumed over 2hrs - screen off                                                                |
|                        |                                                                                                                                                                                                                                                                                                                                      |
|                        |                                                                                                                                                                                                                                                                                                                                      |
