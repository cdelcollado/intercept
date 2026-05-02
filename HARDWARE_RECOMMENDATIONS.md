# INTERCEPT — Hardware Recommendations

Beyond a basic RTL-SDR receiver, this guide covers hardware to unlock every mode and maximize performance. Recommendations are based on real-world compatibility with INTERCEPT's tooling (Linux drivers, SoapySDR, subprocess integration).

---

## 1. Additional SDR Receivers

Running multiple modes simultaneously requires multiple SDRs — one per active decoder.

| Product | Bandwidth | Frequency Range | Price | Best For |
|---------|-----------|-----------------|-------|----------|
| **RTL-SDR Blog V4** | 2.4 MS/s | 500 kHz – 1.7 GHz (HF via direct sampling) | ~$40 | Primary workhorse: ADS-B, AIS, pager, sensors, APRS, ACARS |
| **Nooelec NESDR SMArt v5** | 2.4 MS/s | 25 MHz – 1.7 GHz | ~$35 | Best alternative to RTL-SDR V4; excellent thermal stability for long-running feeders |
| **Airspy R2** | 10 MS/s | 24 MHz – 1.8 GHz | ~$170 | Wideband waterfall, Listening Post scanning, HF via SpyVerter |
| **Airspy Mini** | 6 MS/s | 24 MHz – 1.8 GHz | ~$100 | Compact wideband: weather sats, radiosondes, wideband recordings |
| **SDRplay RSP1B** | 10 MS/s | 1 kHz – 2 GHz | ~$140 | Best all-rounder: HF through L-band without upconverters; excellent dynamic range |
| **HackRF One** | 20 MS/s | 1 MHz – 6 GHz | ~$330 | Sub-GHz analyzer, TSCM sweeps, wideband IQ capture, signal replay |
| **LimeSDR Mini 2.0** | 40 MHz BW | 10 MHz – 3.5 GHz | ~$400 | Full-duplex (transmit+receive), wideband, experimental |

**Recommendation**: Start with 2× RTL-SDR Blog V4 ($80 total) — one dedicated to a continuous feeder (ADS-B or AIS), the other for exploratory scanning. Add an SDRplay RSP1B or Airspy R2 as a third device for wideband/high-dynamic-range work.

---

## 2. WiFi Adapters (Monitor Mode + Packet Injection)

WiFi scanning, deauth detection, PMKID capture, rogue AP detection, and WiFi Locate all require a monitor-mode capable adapter with packet injection.

| Product | Chipset | Bands | Price | Notes |
|---------|---------|-------|-------|-------|
| **Alfa AWUS036ACH** | RTL8812AU | 2.4 + 5 GHz (AC1200) | ~$55 | Gold standard for Kali Linux; dual-band with excellent range and sensitivity |
| **Alfa AWUS036AXM** | MT7921AU | 2.4 + 5 + 6 GHz (WiFi 6E) | ~$65 | Future-proof; supports WiFi 6/6E scanning and 6 GHz band |
| **Alfa AWUS036ACS** | RTL8811AU | 2.4 + 5 GHz (AC600) | ~$25 | Budget dual-band option; good range, lower throughput |
| **Panda PAU09** | Ralink RT5572 | 2.4 + 5 GHz (N600) | ~$35 | Reliable Linux driver support out of the box; no injection issues |
| **TP-Link TL-WN722N v1** | Atheros AR9271 | 2.4 GHz only (N150) | ~$20 (used) | Legendary for reliability; v1 only — later versions swapped chipsets |

**Recommendation**: **Alfa AWUS036ACH** (~$55) as primary. Excellent Linux support via `aircrack-ng` suite, dual-band with a removable high-gain antenna, and strong packet injection reliability. Add the **AWUS036AXM** if 6 GHz scanning matters to you.

**Important**: Always verify chipset before buying — many adapters change chipsets between hardware revisions without changing model numbers. The adapters listed above have consistent chipset histories.

---

## 3. Bluetooth Adapters

While most laptops have built-in Bluetooth, these are usually limited to a handful of simultaneous connections and have poor antenna placement. For serious BLE scanning (BT Locate, TSCM sweeps, tracker detection), a dedicated external adapter is transformative.

| Product | Chipset | BLE | Price | Notes |
|---------|---------|-----|-------|-------|
| **Sena UD100-G03** | CSR8510 | BLE 4.0 | ~$15 | Standard recommendation; good range, reliable Linux support |
| **ASUS USB-BT500** | RTL8761B | BLE 5.0 | ~$20 | Compact, BLE 5.0 long-range mode, works with BlueZ |
| **TP-Link UB500** | RTL8761B | BLE 5.0 | ~$15 | Same chipset as ASUS, slightly smaller |
| **nRF52840 Dongle (Nordic)** | nRF52840 | BLE 5.3 | ~$30 | Advanced: can run custom firmware (Sniffer, Ubertooth-style captures) |

**Recommendation**: **ASUS USB-BT500** (~$20) for general use. The **nRF52840 Dongle** (~$30) if you want to flash custom BLE sniffer firmware for passive 40-channel scanning (similar to Ubertooth capabilities but at lower cost).

---

## 4. Ubertooth One — Advanced BLE Sniffing

INTERCEPT has native Ubertooth integration for passive BLE capture across all 40 advertising channels simultaneously. This goes far beyond what a standard Bluetooth adapter can do.

| Product | Price | Notes |
|---------|-------|-------|
| **Great Scott Gadgets Ubertooth One** | ~$120 | Passive BLE capture, 40-channel scanning, raw advertising payload access |

**Recommendation**: **Ubertooth One** (~$120) if you do serious TSCM sweeps or Bluetooth security research. This is the only way to passively capture BLE advertisements across all channels without missing packets. Standard adapters only poll a few channels at a time and miss bursty advertisements.

---

## 5. GPS Receivers

Required for: GPS mode, BT Locate, WiFi Locate (situational awareness), APRS, and AIS with position-aware features. Multiple modes benefit from precise observer location.

| Product | Chipset | Price | Notes |
|---------|---------|-------|-------|
| **GlobalSat BU-353-S4** | SiRF Star IV | ~$35 | Industry standard USB GPS; excellent sensitivity, magnetic base, IPX7 waterproof |
| **VK-162 G-Mouse** | u-blox 7 | ~$15 | Budget USB GPS; small, decent performance, good enough for most use cases |
| **u-blox NEO-M9N module (USB)** | u-blox M9 | ~$55 | Multi-constellation (GPS + GLONASS + Galileo + BeiDou), 25 Hz updates, survey-grade |

**Recommendation**: **GlobalSat BU-353-S4** (~$35) for most users — reliable, weatherproof, and works with `gpsd` out of the box. The **u-blox NEO-M9N** (~$55) if you need fast update rates for BT Locate signal trail mapping.

---

## 6. Meshtastic LoRa Devices

Required for the Meshtastic mesh networking mode. These devices create a decentralized LoRa mesh for off-grid text communication and telemetry sharing.

| Product | Chipset | Price | Notes |
|---------|---------|-------|-------|
| **LILYGO T-Beam Supreme** | ESP32-S3 + SX1262 + GPS | ~$45 | Best all-rounder: WiFi/BLE for config, LoRa for mesh, built-in GPS, 18650 battery holder |
| **LILYGO T-Echo** | nRF52840 + SX1262 + GPS | ~$55 | E-ink display, excellent battery life, built-in GPS |
| **Heltec WiFi LoRa 32 v3** | ESP32-S3 + SX1262 | ~$22 | Compact, no GPS, no battery — good as a stationary mesh router |
| **RAKwireless WisBlock** | nRF52840 + SX1262 | ~$50 (kit) | Modular, industrial-grade, solar-ready with WisBlock power modules |

**Recommendation**: **LILYGO T-Beam Supreme** (~$45) as primary node — GPS, battery, OLED display, excellent range with the SX1262 radio, and direct USB connection to INTERCEPT. Get a second **Heltec WiFi LoRa 32 v3** (~$22) as a stationary relay node.

---

## 7. Antennas

The stock telescopic antenna included with RTL-SDR dongles is adequate for strong local signals but severely limits range and band coverage. Matching the antenna to the frequency band is the single most impactful upgrade.

### 7.1 Broadband / General Purpose

| Antenna | Frequency | Price | Notes |
|---------|-----------|-------|-------|
| **Tram 1410 Discone** | 25 MHz – 1.3 GHz | ~$70 | Outdoor wideband, excellent for general scanning, VHF/UHF |
| **Diamond D-130J Discone** | 25 MHz – 1.3 GHz | ~$110 | Higher-quality Tram alternative, better build quality |
| **Comet DS-150S Discone** | 25 MHz – 1.5 GHz | ~$120 | 1.5 GHz coverage for ADS-B + L-band |

### 7.2 ADS-B (1090 MHz)

| Antenna | Frequency | Price | Notes |
|---------|-----------|-------|-------|
| **RTL-SDR Blog 1090 MHz ADS-B Antenna** | 1090 MHz | ~$45 | Purpose-built: 5 dBi gain, weatherproof, includes cable and suction-cup mount |
| **FlightAware 1090 MHz Antenna** | 1090 MHz | ~$45 | 5.5 dBi, outdoor-rated, N-connector, 26" tall |
| **Vinnant 1090 MHz Collinear** | 1090 MHz | ~$60 | 9 dBi, fiberglass radome, best range for ADS-B feeders |

### 7.3 AIS / VHF Marine (162 MHz)

| Antenna | Frequency | Price | Notes |
|---------|-----------|-------|-------|
| **Shakespeare 5206-C** | 156-162 MHz | ~$35 | 3 dB marine VHF whip, stainless steel, N-connector |
| **Diamond D-190** | 50/145/435 MHz | ~$100 | Tri-band, covers VHF marine + 2m/70cm amateur bands |

### 7.4 Weather Satellites (137 MHz)

| Antenna | Frequency | Price | Notes |
|---------|-----------|-------|-------|
| **RTL-SDR Blog V-Dipole** | 120-150 MHz (tunable) | ~$25 | Extensible dipole kit; set elements to ~53 cm each for 137 MHz. Proven for NOAA APT/Meteor LRPT |
| **Winkler Turnstile** | 137 MHz | ~$60 | Crossed dipole, circular polarization, no rotor needed — ideal for satellite passes |
| **QFH (Quadrifilar Helix)** | 137 MHz | ~$70-150 | Best for weather satellites; circularly polarized, needs construction or pre-built |

### 7.5 HF (0-30 MHz)

| Antenna | Frequency | Price | Notes |
|---------|-----------|-------|-------|
| **MLA-30+ Active Loop** | 100 kHz – 30 MHz | ~$40 | Compact amplified loop, excellent for apartment/indoor HF, WeFax, CW, SSTV |
| **YouLoop** | 10 kHz – 30 MHz | ~$40 | Passive loop (no amplifier needed), very quiet, good with SDRplay/Airspy |
| **Nooelec 9:1 Balun + Wire** | 100 kHz – 30 MHz | ~$25 (balun) + wire | Simple end-fed; best bang-for-buck for outdoor HF with space for a long wire |

### 7.6 L-Band (1.5-1.7 GHz) — Iridium, Inmarsat

| Antenna | Frequency | Price | Notes |
|---------|-----------|-------|-------|
| **RTL-SDR Blog Active L-Band Patch** | 1525-1660 MHz | ~$50 | Patch antenna with built-in LNA and SAW filter for Iridium/Inmarsat |
| **Modified GPS patch + SAWbird+ LNA** | 1.5-1.7 GHz | ~$60 (bundle) | DIY alternative; Nooelec SAWbird+ Iridium LNA + a ceramic GPS patch tuned to L-band |

**Recommendation**: Start with the **Tram 1410 Discone** (~$70, outdoor) as your always-connected general-purpose antenna. Add the **RTL-SDR Blog ADS-B antenna** (~$45) for aircraft and the **MLA-30+ loop** (~$40) for HF. These three cover 90% of INTERCEPT modes.

---

## 8. LNAs (Low Noise Amplifiers)

Critical for weak signals: weather satellites, Iridium, distant ADS-B aircraft, and HF with small antennas. An LNA placed at the antenna compensates for cable loss before the signal reaches the SDR.

| Product | Frequency | Gain | NF | Price | Best For |
|---------|-----------|------|----|-------|----------|
| **Nooelec SAWbird+ ADS-B** | 1090 MHz | ~20 dB | <1 dB | ~$35 | ADS-B range extension; built-in SAW filter blocks cell tower interference |
| **Nooelec SAWbird+ Iridium** | 1.5-1.7 GHz | ~20 dB | <1 dB | ~$40 | Iridium/Inmarsat L-band signals |
| **Nooelec SAWbird+ NOAA** | 137 MHz | ~20 dB | <1 dB | ~$35 | NOAA APT + Meteor LRPT weather satellites |
| **Nooelec LaNA** | 50 kHz – 150 MHz | ~20 dB | ~2.5 dB | ~$35 | General purpose VLF-HF LNA; good with the MLA-30+ for WeFax and CW |
| **LNA4ALL** | 50 kHz – 4 GHz | ~20 dB | <1 dB | ~$50 | Ultra-wideband, exceptionally low noise figure |
| **RTL-SDR Blog Wideband LNA** | 50 MHz – 4 GHz | ~20 dB | <1.5 dB | ~$25 | Budget wideband option with bias-t power |

**Recommendation**: **Nooelec SAWbird+ ADS-B** (~$35) first — the built-in SAW filter is essential near cell towers. Add the **SAWbird+ NOAA** (~$35) if you do weather satellite reception. For general use, the **LNA4ALL** (~$50) covers everything with excellent specs.

**Important**: LNAs require power — usually via bias-t (built into RTL-SDR V4 and Nooelec Smart dongles) or a separate USB power injector. Verify your SDR supports bias-t before buying.

---

## 9. Filters

Band-pass filters block out-of-band interference (FM broadcast, cell towers, pagers) that can desensitize the SDR frontend.

| Product | Passband | Price | Best For |
|---------|----------|-------|----------|
| **RTL-SDR Blog FM Bandstop** | Blocks 88-108 MHz | ~$15 | Essential — FM broadcast stations saturate RTL-SDR frontends. Put this before any LNA. |
| **Nooelec Flamingo AM Bandstop** | Blocks 0-30 MHz | ~$15 | Blocks AM broadcast interference for VHF/UHF scanning |
| **RTL-SDR Blog 1090 MHz SAW Filter** | 1090 MHz ± 2 MHz | ~$15 | Tight ADS-B filter; blocks adjacent cell/pager interference |
| **Nooelec SAWbird+ series** | Various (see LNAs) | ~$35 each | Combined LNA + filter; simpler than separate components |

**Recommendation**: **RTL-SDR Blog FM Bandstop** (~$15) is non-negotiable for any SDR setup. FM stations are so powerful they clip the ADC on RTL-SDRs, producing phantom signals everywhere. Add the **1090 MHz SAW filter** (~$15) if you feed ADS-B data and live near cell towers.

---

## 10. Raspberry Pi — Deployment Platform

For permanent/headless deployments (24/7 ADS-B feeder, AIS station, weather satellite auto-scheduler), a Raspberry Pi is the ideal host.

| Model | RAM | Price | Notes |
|-------|-----|-------|-------|
| **Raspberry Pi 5** | 8 GB | ~$80 | Best performance; handles multiple SDRs + INTERCEPT with gunicorn |
| **Raspberry Pi 4 Model B** | 4 GB | ~$55 | Proven, widely available, sufficient for 1-2 simultaneous decoders |
| **Raspberry Pi 4 Model B** | 2 GB | ~$35 | Minimal; only if running a single feeder (ADS-B or AIS) |

**Accessories needed**:
- **SD card**: Samsung Pro Endurance 64 GB (~$12) — high-write-endurance for log files
- **Power supply**: Official Raspberry Pi 5 PSU (~$12)
- **Case**: Official Pi 5 case with fan (~$10) — thermal throttling is real with INTERCEPT's subprocess workload
- **Optional: NVMe HAT + SSD**: Pimoroni NVMe Base (~$15) + WD SN580 250 GB NVMe (~$30) — dramatically faster than SD, especially for Postgres ADS-B history

**Recommendation**: **Raspberry Pi 5 8 GB** (~$80) with **Samsung Pro Endurance 64 GB** (~$12) for a reliable always-on rig. Add an NVMe SSD if running the ADS-B history profile with Postgres.

---

## 11. Portable Field Kit

For mobile SIGINT / TSCM sweeps / vehicle-based operations.

| Item | Product | Price |
|------|---------|-------|
| **Laptop / SBC** | Raspberry Pi 5 8 GB + 7" touchscreen | ~$160 |
| **Power** | Anker PowerCore 26800 mAh USB-C PD | ~$60 |
| **Power (advanced)** | TalentCell 12V LiFePO4 battery pack | ~$50 |
| **Carrying case** | Harbor Freight Apache 3800 (Pelican-style) | ~$40 |
| **SDRs** | RTL-SDR V4 (×2) + HackRF One | ~$410 |
| **WiFi** | Alfa AWUS036ACH | ~$55 |
| **Bluetooth** | ASUS USB-BT500 + Ubertooth One | ~$140 |
| **GPS** | GlobalSat BU-353-S4 | ~$35 |
| **Antenna** | Tram 1410 (on mag mount) + MLA-30+ (on window) | ~$110 |
| **Filters** | FM bandstop + ADS-B SAW | ~$30 |
| **LNA** | LNA4ALL | ~$50 |
| **Cables** | SMA adapters, USB extensions, coax jumpers | ~$30 |

**Total field kit**: ~$1,170 — covers every INTERCEPT mode in a portable package.

---

## 12. USB Hub — Essential for Multi-Device Setups

With multiple SDRs, WiFi/Bluetooth adapters, and GPS, USB ports disappear quickly. A powered USB 3.0 hub is essential.

| Product | Ports | Price | Notes |
|---------|-------|-------|-------|
| **Anker 10-Port 60W USB 3.0 Hub** | 10× USB-A 3.0 | ~$55 | Reliable, 60W total power, per-port switches, proven with SDRs |
| **Plugable 7-Port USB 3.0 Hub** | 7× USB-A 3.0 | ~$35 | 25W, good enough for 3-4 SDRs |
| **Sabrent 4-Port USB 3.0 Hub (powered)** | 4× USB-A 3.0 | ~$15 | Compact, individual switches, 5V/2.5A |

**Recommendation**: **Anker 10-Port** (~$55) if you're serious. SDRs draw more current than typical USB devices, and bus-powered hubs will cause undervoltage issues. Always get a powered hub.

---

## Summary: Recommended Build Path

### Minimal starter kit (~$135 beyond the SDR you already have)

| Item | Product | Price |
|------|---------|-------|
| WiFi adapter | **Alfa AWUS036ACS** | ~$25 |
| Bluetooth | **ASUS USB-BT500** | ~$20 |
| GPS | **VK-162 G-Mouse** | ~$15 |
| Antenna | **Tram 1410 Discone** | ~$70 |
| Filter | **RTL-SDR FM Bandstop** | ~$15 |
| **Total** | | **~$145** |

### Enthusiast kit (~$600 for a fully-featured setup)

| Item | Product | Price |
|------|---------|-------|
| Second SDR | **RTL-SDR Blog V4** | ~$40 |
| WiFi adapter | **Alfa AWUS036ACH** | ~$55 |
| Bluetooth | **ASUS USB-BT500** | ~$20 |
| GPS | **GlobalSat BU-353-S4** | ~$35 |
| Outdoor antenna | **Tram 1410 Discone** | ~$70 |
| ADS-B antenna | **RTL-SDR Blog 1090 MHz** | ~$45 |
| HF antenna | **MLA-30+ Active Loop** | ~$40 |
| LNA ×2 | **SAWbird+ ADS-B** + **SAWbird+ NOAA** | ~$70 |
| Filters | **FM bandstop** + **ADS-B SAW** | ~$30 |
| Meshtastic node | **LILYGO T-Beam Supreme** | ~$45 |
| USB hub | **Plugable 7-Port powered** | ~$35 |
| Deployment | **Raspberry Pi 5 8 GB** + SD + case | ~$105 |
| **Total** | | **~$590** |

### Professional kit (~$1,300 for serious SIGINT work)

Everything in the enthusiast kit plus:

| Item | Product | Price |
|------|---------|-------|
| Third SDR (wideband) | **SDRplay RSP1B** or **Airspy R2** | ~$150 |
| HackRF | **HackRF One** | ~$330 |
| Ubertooth | **Ubertooth One** | ~$120 |
| Weather sat antenna | **Winkler Turnstile** | ~$60 |
| LNA (wideband) | **LNA4ALL** | ~$50 |
| **Total** | | **~$710** |
