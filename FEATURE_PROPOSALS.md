# INTERCEPT — New Feature Proposals

## 1. New Signal Types & Decoders

### 1.1 FT8 / FT4 / JT65 — Amateur Digital Mode Decoder
- **What**: Decode WSJT-X weak-signal digital modes using `wsjtx` or `jtdx` as a backend
- **Why**: FT8 is by far the most popular HF digital mode worldwide. Providing a web dashboard for spotting activity across bands (160m-6m) would be a major draw for the amateur radio community. Decode callsigns, grid locators, and SNR reports in real-time, with a band-activity table and a world map plotting decoded stations.
- **Feasibility**: High — WSJT-X can output UDP packets with decoded messages. INTERCEPT would need a UDP listener and an SSE broadcaster.
- **Effort**: Medium

### 1.2 DAB / DAB+ — Digital Audio Broadcasting Receiver
- **What**: Decode DAB/DAB+ radio multiplexes (174-240 MHz) via `dabtools`, `welle.io` CLI, or `dab-cmdline`
- **Why**: DAB+ is widely deployed in Europe, UK, and Australia. Listening to digital radio stations and showing station lists, bitrates, and ensemble data directly from the SDR adds broadcast-monitoring capability.
- **Feasibility**: Medium — Requires a DAB decoder CLI tool. `welle.io` has a CLI frontend (`welle-cli`). RTL-SDR can sample at the required 2.048 MS/s.
- **Effort**: Medium

### 3.1 FM Broadcast Radio
- **What**: Receive and stream FM broadcast stations (88-108 MHz) with RDS (Radio Data System) decoding
- **Why**: While "basic," an FM radio mode with a station scanner, RDS text display (station name, song title), and audio streaming via WebSocket would be a polished demo mode that introduces INTERCEPT to casual users before they explore more advanced modes.
- **Feasibility**: High — `rtl_fm` already handles FM demodulation. RDS decoding via `redsea` CLI. Audio streaming via existing WebSocket infrastructure.
- **Effort**: Low

### 1.4 NOAA Weather Radio (NWR) — Voice + SAME Alert Decoder
- **What**: Decode NOAA Weather Radio broadcasts (162.400-162.550 MHz) including SAME (Specific Area Message Encoding) alert data
- **Why**: In the US, NWR provides critical weather alerts. Decoding SAME alert tones shows county-level warning polygons. This complements the existing weather satellite mode and Space Weather mode.
- **Feasibility**: High — `rtl_fm` + `multimon-ng` already handle FM demodulation. SAME decoding via `minimodem` or `multimon-ng`. Map integration for alert polygons.
- **Effort**: Medium

### 1.5 IRIDIUM / L-Band Satellite Signal Decoding
- **What**: Decode Iridium satellite pager messages, ACARS-over-Iridium, and STDC-C broadcasts at 1.5-1.6 GHz
- **Why**: Iridium satellites relay global maritime safety (GMDSS), aircraft tracking, and short-burst data. Tools like `iridium-toolkit` and `gr-iridium` can demodulate these. A compelling addition to the "Space" category alongside satellites, ISS SSTV, and weather sats.
- **Feasibility**: Medium — Requires higher sample-rate SDR (2+ MS/s) and specific antenna (patch/helical for L-band). Software exists in the open-source ecosystem.
- **Effort**: High

### 1.6 LORAN-C / eLORAN — Precision Timing & Navigation Receiver
- **What**: Decode LORAN-C (90-110 kHz) legacy and eLORAN modernized navigation signals
- **Why**: eLORAN is being deployed as a GNSS backup. Decoding pulse groups and showing time-of-arrival data is a niche but unique feature. Complements AIS/vessel tracking for maritime use cases.
- **Feasibility**: Low-Medium — Requires an upconverter or direct-sampling capable SDR for the 100 kHz band. Very niche.
- **Effort**: High

### 1.7 DRM (Digital Radio Mondiale) — HF Digital Radio
- **What**: Decode DRM30 (HF) and DRM+ (VHF) digital radio broadcasts via `dream` decoder
- **Why**: DRM is the digital replacement for AM shortwave broadcasting. Decoding station labels, bitrates, and audio provides another broadcast-monitoring capability.
- **Feasibility**: Medium — `dream` is open source with a CLI interface. Needs an IQ stream.
- **Effort**: Medium

### 1.8 STANAG 4285 / NATO HF Modem
- **What**: Decode NATO STANAG 4285 (HF PSK modem) signals commonly used by military and diplomatic HF stations
- **Why**: Complements the existing Spy Stations mode. Many number stations and diplomatic networks use STANAG 4285 for encrypted traffic. While decryption is not possible, detecting and classifying the signal adds intelligence value.
- **Feasibility**: Medium — `sorcerer` or `sigutils` / `sig-digger` can classify the signal. Full decoding is not feasible without keys, but signal detection/classification is valuable.
- **Effort**: Medium

---

## 2. Platform & Infrastructure

### 2.1 Multi-User Authentication with Role-Based Access
- **What**: Replace the single `admin:admin` credential with proper multi-user support: Admin, Operator (can start/stop decoders), Viewer (read-only)
- **Why**: The platform is currently single-user only. In a team or lab setting, different users need different access levels. A viewer role allows sharing real-time dashboards without risking accidental decoder termination.
- **Feasibility**: High — Flask-Login is already partially used. Needs a `users` table, password hashing (werkzeug), and role decorators on routes.
- **Effort**: Medium

### 2.2 Scheduled / Cron-Based Scanning
- **What**: Schedule decoder runs (start/stop) based on time of day, satellite passes, or recurring intervals
- **Why**: Many signal types are time-dependent: ISS SSTV events are scheduled days in advance, weather satellite passes follow orbital mechanics, and HF propagation varies by time of day. Automating capture reduces operator burden.
- **Feasibility**: High — Add a `scheduled_tasks` table, a lightweight scheduler thread (APScheduler or custom), and a UI panel for managing schedules.
- **Effort**: Medium

### 2.3 Multi-SDR Orchestrator
- **What**: A visual dashboard showing all connected SDR devices, their current assignments, and the ability to drag-and-drop modes onto available devices. Conflict prevention and "swap" capabilities.
- **Why**: With multiple SDRs (e.g., one RTL-SDR for ADS-B, one HackRF for TSCM, one LimeSDR for waterfall), managing assignments manually is error-prone. The SDR registry already exists — this is a UI layer on top.
- **Feasibility**: High — The SDR device registry and claiming logic are already implemented. Needs a visual dashboard and assign/reassign API.
- **Effort**: Medium

### 2.4 Plugin / Extension System
- **What**: Allow community developers to add new signal modes as plugins without modifying core INTERCEPT code. A plugin would define: a route blueprint, a JS module, a CSS file, HTML partials, and metadata (name, icon, group). 
- **Why**: The current "19+ places to edit" for each new mode creates a high barrier to entry. A plugin system with a registration hook would let contributors add modes via a single Python package.
- **Feasibility**: Medium — Use Python entry points (`intercept.plugins`) for discovery, a registry pattern for mode metadata, and dynamic loading of blueprints/JS/CSS in the template.
- **Effort**: High

### 2.5 REST API with OpenAPI Documentation
- **What**: A documented, versioned REST API (`/api/v1/...`) for all mode start/stop/status/export operations, with Swagger UI
- **Why**: Currently there is no documented API — endpoints are ad-hoc and inconsistent. A proper REST API would enable third-party integrations, Home Assistant plugins, Node-RED flows, and custom dashboards.
- **Feasibility**: High — Flask + flasgger (Swagger) or Flask-RESTX. Add decorators to existing routes. Reuse existing logic.
- **Effort**: Medium

### 2.6 Grafana / Prometheus Metrics Export
- **What**: Export operational metrics to Prometheus: active decoders, message counts, queue depths, CPU/memory, SSE client counts, SDR device utilization
- **Why**: For long-running deployments (ADS-B feeders, AIS stations), monitoring the health of the system is critical. Prometheus metrics enable Grafana dashboards and alerting rules.
- **Feasibility**: High — `prometheus_client` Python library. Add a `/metrics` endpoint. Minimal performance impact.
- **Effort**: Low

### 2.7 IQ Recording & Playback
- **What**: Record raw IQ samples to disk with metadata (center frequency, sample rate, timestamp, gain) in SigMF format. Replay recorded IQ files through any decoder as if they were live.
- **Why**: Enables offline analysis of captures, sharing interesting signals with the community, and testing decoders against known-good recordings. The SigMF standard ensures interoperability with other tools (GNU Radio, Universal Radio Hacker).
- **Feasibility**: Medium — IQ capture via existing SDR backends already works (waterfall mode). Needs SigMF metadata writer, file management UI, and a "replay" mode that pipes IQ files to decoders.
- **Effort**: Medium

### 2.8 Spectrum Waterfall Recording & Replay
- **What**: Continuously record wideband spectrum waterfall data (not raw IQ, just FFT power spectra) for hours/days with a timeline scrubber for playback
- **Why**: Operators want to review spectrum activity over time — "was there a transmission at 433.92 MHz at 3 AM?" — without storing terabytes of raw IQ. Compressed waterfall data is compact (FFT bins × time slices) and can be queried by frequency band and time range.
- **Feasibility**: Medium — Extend the waterfall WebSocket mode to persist FFT frames to a time-series database (or HDF5/SQLite). Add a timeline playback UI with a scrubber.
- **Effort**: Medium

---

## 3. Data Management & Export

### 3.1 Database Backup & Restore UI
- **What**: One-click backup and restore of the INTERCEPT database (settings, TSCM baselines, DSC alerts, ADS-B history) from the Settings modal
- **Why**: Currently there is no easy way to back up configuration, baselines, or alert history. Users moving between machines or reinstalling lose all data.
- **Feasibility**: High — SQLite `.dump` or file copy. Add two buttons in Settings.
- **Effort**: Low

### 3.2 Configurable Data Retention Policies
- **What**: Per-mode configuration for how long data is retained: pager messages (24h), ADS-B (7 days), WiFi (30 min), TSCM sweeps (forever). Automatic cleanup via the existing CleanupManager.
- **Why**: Storage grows unbounded, especially for high-volume modes like ADS-B and pager. Users should control retention based on their storage constraints and analysis needs.
- **Feasibility**: High — Extend the existing `CleanupManager` with per-mode TTL settings from the database.
- **Effort**: Low

### 3.3 GeoJSON / KML Export for All Mapped Modes
- **What**: One-click export of visible data (aircraft, vessels, satellites, APRS stations) to GeoJSON or KML for use in GIS tools, Google Earth, or other analysis platforms
- **Why**: Current exports are CSV/JSON only. GeoJSON/KML are the standard exchange formats for geospatial data. Useful for post-mission analysis or sharing with other teams.
- **Feasibility**: High — Each DataStore already has the required fields (lat, lon, metadata). Add a format parameter to the export endpoints.
- **Effort**: Low

### 3.4 PDF Report Generation
- **What**: Generate a formatted PDF report summarizing a session: devices detected, aircraft tracked, vessels seen, threat alerts, signal statistics, and charts
- **Why**: Professional TSCM sweeps, site surveys, and reconnaissance missions require documentation. A one-click report with logo, timestamps, and a structured layout would add professional credibility.
- **Feasibility**: Medium — Use `reportlab` or `weasyprint` for PDF generation. Build a report template with sections for each active mode.
- **Effort**: Medium

---

## 4. Integration & Interoperability

### 4.1 MQTT Publisher Bridge
- **What**: Publish decoded data (sensor readings, aircraft positions, vessel positions, pager messages) to an MQTT broker in real-time
- **Why**: MQTT is the de facto standard for IoT data. Publishing sensor readings to MQTT enables integration with Home Assistant, Node-RED, InfluxDB, and other smart-home/data platforms. A single SDR could feed an entire smart home.
- **Feasibility**: High — `paho-mqtt` library. Add optional MQTT configuration in Settings. Publish from within each mode's output processing thread.
- **Effort**: Low

### 4.2 ADS-B Exchange / FlightAware / FlightRadar24 Feed
- **What**: Relay ADS-B data to community aggregators (ADSBexchange, FlightAware, FR24, OpenSky Network) while still displaying locally
- **Why**: Many users already run dump1090 feeding these aggregators. Integrating the relay into INTERCEPT simplifies setup and makes the platform a drop-in replacement for standalone feeders.
- **Feasibility**: High — These services accept Beast/AVR/TCP raw data. INTERCEPT already has access to the SBS stream on port 30003. A relay module splits the stream.
- **Effort**: Low

### 4.3 APRS-IS Internet Gateway
- **What**: Bidirectional APRS-IS integration: upload received APRS packets to the APRS-IS network and optionally display stations heard via the internet
- **Why**: Many APRS operators run igates. INTERCEPT with direwolf already decodes APRS; adding APRS-IS uplink makes it a full igate. Downlink from APRS-IS shows activity even when local reception is poor.
- **Feasibility**: Medium — The APRS-IS protocol is well-documented. direwolf already supports this internally. Needs configuration UI and passthrough.
- **Effort**: Medium

### 4.4 SatNOGS Integration
- **What**: Upload satellite observations (TLE-based passes, decoded telemetry) to the SatNOGS network and fetch scheduled observations
- **Why**: SatNOGS is the largest open-source satellite ground station network. Contributing observations increases global coverage. The satellite mode already fetches TLE data — observations could be uploaded with minimal changes.
- **Feasibility**: Medium — SatNOGS has a REST API. Needs observation metadata (waterfall, audio, decoded frames) and authentication via API key.
- **Effort**: Medium

### 4.5 SondeHub Radiosonde Integration
- **What**: Push radiosonde telemetry to SondeHub and pull nearby sonde predictions from the network
- **Why**: The radiosonde community uses SondeHub for collaborative tracking. INTERCEPT's existing radiosonde mode could become a data source and consumer.
- **Feasibility**: Medium — SondeHub has a WebSocket API. radiosonde_auto_rx already supports uploading. Configuration UI needed.
- **Effort**: Medium

### 4.6 Discord / Telegram / Slack Alert Notifications
- **What**: Send real-time alerts (ADS-B emergency, DSC distress, rogue AP detected, TSCM threat found) to Discord, Telegram, or Slack webhooks
- **Why**: Operators may not be watching the dashboard 24/7. Push notifications for critical events enable timely response. The alert system and webhook infrastructure already exist — this extends the delivery channels.
- **Feasibility**: High — Simple HTTP POST to webhook URLs. Add channel configuration in the alert rules UI.
- **Effort**: Low

### 4.7 InfluxDB / TimescaleDB Time-Series Export
- **What**: Stream decoded metrics (signal strength, message counts, sensor readings, aircraft positions) to a time-series database for long-term analytics and Grafana dashboards
- **Why**: For serious deployments, SQLite is not suitable for time-series data. Exporting to InfluxDB or TimescaleDB enables Grafana dashboards, historical analysis, and retention-based downsampling.
- **Feasibility**: Medium — `influxdb-client` Python library. Add optional configuration and a background writer thread.
- **Effort**: Medium

---

## 5. UI/UX Enhancements

### 5.1 Dark/Light/High-Contrast Theme Toggle
- **What**: A theme selector with three options: Dark (current), Light, and High-Contrast (accessibility). Theme choice persisted in localStorage.
- **Why**: The FEATURES.md already mentions a theme toggle but the current implementation only has dark mode. Light mode is essential for outdoor/field use in bright sunlight. High-contrast mode serves users with visual impairments.
- **Feasibility**: High — CSS variables already exist for theming (`--bg-primary`, `--accent-cyan`, etc.). Add a second set of light variables and a toggle in the nav bar.
- **Effort**: Low

### 5.2 Responsive Mobile Dashboard
- **What**: A mobile-first layout that works well on phones and tablets in the field, with large touch targets, simplified views, and reduced data density
- **Why**: SIGINT operations often happen in the field (vehicle-based, portable setups). A mobile-optimized view would let operators check dashboards from a phone without a laptop.
- **Feasibility**: Medium — The current HTML has responsive breakpoints but was designed desktop-first. A mobile mode with stacked panels, large buttons, and simplified maps would require significant CSS work.
- **Effort**: Medium

### 5.3 Dashboard Customization / Layout Builder
- **What**: Drag-and-drop panels to reorder and resize dashboard widgets. Save custom layouts per mode.
- **Why**: Different operators prioritize different data. An AIS operator may want the map to take 80% of the screen, while a pager operator wants the message log front and center. Custom layouts improve workflow.
- **Feasibility**: Medium — CSS Grid + a drag-and-drop library (e.g., `gridstack.js`). Layout config stored in localStorage.
- **Effort**: Medium

### 5.4 Keyboard Navigation & Shortcuts Expansion
- **What**: Expand keyboard shortcuts to include: `S` to start current mode, `X` to stop, `1-9` to switch modes, arrow keys to tune frequency, `Space` to toggle audio mute
- **Why**: Power users and field operators want to operate without a mouse. Full keyboard navigation dramatically improves speed and usability.
- **Feasibility**: High — Add a global keyboard event listener in `index.html` with a shortcuts registry.
- **Effort**: Low

### 5.5 Audio Spectrum Analyzer Overlay
- **What**: A real-time audio FFT waterfall/spectrogram in a small overlay panel, usable across multiple modes (Listening Post, WebSDR, ACARS audio, FM radio)
- **Why**: Visual feedback of audio quality helps operators tune precisely and identify modulation types. A small spectrum analyzer widget would be a "wow factor" feature.
- **Feasibility**: Medium — Web Audio API `AnalyserNode` from the existing audio streaming pipeline. Canvas-based FFT visualization already partially exists in waterfall mode.
- **Effort**: Medium

---

## 6. Community & Collaboration

### 6.1 Signal Database / Wiki Integration
- **What**: A curated local database of known signals (frequencies, modulations, sample audio, descriptions) integrated into the Listening Post and Signal Scanner. Pull from and contribute to SigIDWiki.
- **Why**: The SignalID feature already queries SigIDWiki. Expanding this to a browsable local catalog with audio samples would help users learn about the RF spectrum.
- **Feasibility**: Medium — Import SigIDWiki data (CC-BY-SA). Cache locally for offline use. Add an in-app signal browser.
- **Effort**: Medium

### 6.2 Remote Collaboration / Shared Session
- **What**: Generate a shareable link or room code that lets multiple users view the same dashboard in real-time (read-only), useful for teaching, demonstrations, or team operations
- **Why**: In a lab or classroom setting, an instructor could run INTERCEPT on one machine and share the session with students. A TSCM team could share findings in real-time.
- **Feasibility**: Low-Medium — SSE streams are already push-based. Adding a WebRTC or WebSocket bridge for multi-client sync requires significant backend work.
- **Effort**: High

### 6.3 Mode Preset Sharing / Import/Export
- **What**: Export and share mode presets (frequency lists, gain settings, filter configurations) as JSON files. Import community presets from a repository.
- **Why**: New users struggle with configuration. Pre-built presets for common signals (local ADS-B frequencies, regional pager frequencies, known spy station schedules) lower the barrier to entry.
- **Feasibility**: High — JSON export/import buttons. Optional online repository for curated presets.
- **Effort**: Low

---

## 7. Advanced Analysis & Intelligence

### 7.1 Machine Learning Signal Classifier
- **What**: Use a pre-trained ML model (e.g., TorchSig or a custom ONNX model) to automatically classify signals by modulation type and protocol from IQ samples or waterfall images
- **Why**: Signal classification is difficult for beginners. ML-based auto-classification would help users discover and identify unknown signals quickly, similar to how SignalID queries SigIDWiki but using the raw signal characteristics.
- **Feasibility**: Low-Medium — Requires integrating a Python ML runtime (ONNX Runtime is lightweight). Pre-trained models exist in the research community. Significant effort to build the integration pipeline.
- **Effort**: High

### 7.2 Geolocation / TDOA (Time Difference of Arrival)
- **What**: Use multiple INTERCEPT agents with GPS to triangulate the position of a transmitter via TDOA (time difference of arrival), displayed on a map with a confidence ellipse
- **Why**: With the distributed agent architecture already in place, adding TDOA geolocation would be a powerful intelligence capability. Three or more agents synchronized via GPS could locate a transmitter within meters.
- **Feasibility**: Low — TDOA requires microsecond-level timestamp synchronization across distributed receivers. This is a hard problem. Feasible only with hackrf_transfer or specialized hardware (KrakenSDR). Theoretical for RTL-SDR.
- **Effort**: Very High

### 7.3 Traffic Analysis & Pattern Detection
- **What**: Analyze decoded data over time to find patterns: peak ADS-B traffic hours, common WiFi probe SSIDs in an area, repeated pager capcodes, device presence schedules
- **Why**: Temporal analysis reveals operational patterns — when aircraft are most active, when a target device is typically present, or when radio traffic spikes. This turns raw data into actionable intelligence.
- **Feasibility**: Medium — The existing ADS-B history (Postgres) provides the foundation. Extend with time-bucketed aggregation queries and a charting dashboard.
- **Effort**: Medium

### 7.4 Predictive Pass Planning (Multi-Satellite)
- **What**: Given the observer location, show a timeline view of all upcoming satellite passes across all tracked satellites, with conflict detection (two passes overlapping) and automatic mode scheduling
- **Why**: Operators tracking NOAA, Meteor, ISS, and amateur satellites need to know which passes are upcoming and whether they conflict. A Gantt-chart-style timeline with pass priority and auto-scheduling would maximize data capture.
- **Feasibility**: Medium — The satellite pass prediction already works. Add a timeline UI, conflict detection, and scheduled capture triggering.
- **Effort**: Medium

---

## 8. Hardware & Device Support

### 8.1 RTL-SDR Blog V4 Native Support
- **What**: Full integration for RTL-SDR Blog V4: automatic bias-t for the integrated LNA, HF direct-sampling mode toggle, and upconverter support
- **Why**: The Blog V4 is the most popular RTL-SDR currently sold. While basic support exists (bias-t fallback was added in 2.26.9), HF direct-sampling (0-28 MHz via Q-branch) and the built-in upconverter are not exposed in the UI.
- **Feasibility**: High — Add HF-direct and upconverter options to the SDR device panel.
- **Effort**: Low

### 8.2 KrakenSDR Support
- **What**: Support the KrakenSDR (5-channel coherent RTL-SDR) for direction finding and passive radar
- **Why**: KrakenSDR is specifically designed for radio direction finding. Its 5 coherent channels enable TDOA geolocation and DF heatmaps. INTERCEPT's existing agent mode could be extended to ingest DF data.
- **Feasibility:** Low-Medium — The KrakenSDR has its own software stack (krakensdr_doa). Integration would mean consuming its output rather than controlling it directly.
- **Effort:** High

### 8.3 PlutoSDR / USRP / BladeRF Support
- **What**: Expand the SDR abstraction layer to support ADALM-Pluto, Ettus USRP, and Nuand BladeRF devices via SoapySDR or native drivers
- **Why**: These higher-end SDRs offer wider bandwidth, full-duplex, and better dynamic range than RTL-SDR dongles. Supporting them opens INTERCEPT to professional and research users.
- **Feasibility**: Medium — SoapySDR already supports these devices. The SDRFactory pattern makes adding new types straightforward. Needs CommandBuilder implementations.
- **Effort**: Medium

---

## 9. Web Serial API — Browser-Native Device Communication

INTERCEPT currently relies on a Python backend (`intercept.py`) to bridge between the browser and external hardware (SDRs, Meshtastic radios, GPS receivers). The [Web Serial API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Serial_API) (Chrome 89+, Edge 89+) allows the browser to communicate directly with serial-port devices over USB, bypassing the backend entirely for certain modes.

### Where It Would Work

#### Meshtastic LoRa Devices (Heltec, T-Beam, RAK)
- These radios enumerate as USB CDC/ACM serial ports (typically `/dev/ttyUSB0` or `/dev/ttyACM0`).
- The browser could read raw Meshtastic protobuf frames, parse node telemetry, display messages, and send channel configuration — all without the Python backend.
- This would make INTERCEPT's Meshtastic mode dramatically easier to set up: no Python driver stack, no `pyserial`, no `meshtastic` pip package. Just plug in the radio and open the page in Chrome.

#### USB GPS Receivers
- Most USB GPS receivers (GlobalSat BU-353-S4, VK-162, u-blox modules) present as serial CDC devices streaming NMEA sentences at 9600–115200 baud.
- The browser could parse `$GPGGA`, `$GPRMC`, and `$GPGSA` sentences directly, feeding position data into the GPS mode, BT Locate trails, and shared observer location.
- This eliminates the `gpsd` dependency entirely.

#### SDR Configuration Interfaces (Control Channel Only)
- Some SDRs expose a secondary serial interface for configuration (HackRF's USB DFU mode, ADF4351 synthesizer TTL interfaces, AT commands on certain LoRa/GPS combos).
- Web Serial could be used for device configuration UIs without touching the IQ data path.

### Where It Would NOT Work

#### SDR IQ Data Streaming
- Raw IQ samples from any SDR flow at **20–60 MB/s** (USB bulk transfer mode, not UART).
- Web Serial is limited to UART baud rates (theoretical max ~12 Mbps, practical ~3–8 Mbps). This is orders of magnitude too slow for IQ streaming.
- SDR processing must remain server-side where native tools (`rtl_fm`, `dump1090`, `acarsdec`) have direct USB access.

#### Native Decoder Binaries
- Tools like `airodump-ng`, `AIS-catcher`, `SatDump`, and `acarsdec` are compiled C/C++/Go/Java binaries that cannot run inside a browser sandbox.
- The Python backend is irreplaceable as the bridge between these native tools and the web UI.

### Implementation Strategy

```
┌─────────────┐     Web Serial API     ┌──────────────────┐
│  Browser JS │ ◄────────────────────► │  USB Serial Device│
│  (Chrome)   │    NMEA / Protobuf     │  (GPS / LoRa)    │
└─────────────┘                         └──────────────────┘

         │  SSE / fetch()
         ▼
┌─────────────────┐     USB Bulk (high-speed)
│  Python Backend │ ◄──────────────────────────►  SDR Dongle
│  (rtl_fm, etc.) │      20–60 MB/s IQ
└─────────────────┘
```

**Hybrid architecture:**
1. **Web Serial path** (browser → device): Meshtastic protocol data, GPS NMEA sentences, AT command configuration.
2. **Backend path** (server → SDR): Raw IQ streaming, native decoder binaries, subprocess management.
3. Both paths feed into the same SSE event bus and Vue/vanilla JS state layer — the user sees no difference.

### Browser Support
| Browser | Web Serial | Notes |
|---------|-----------|-------|
| Chrome 89+ | Full support | Desktop only (no Android) |
| Edge 89+ | Full support | |
| Opera 75+ | Full support | |
| Firefox | No support | No plans to implement |
| Safari | No support | No plans to implement |

### Feasibility Assessment
- **Impact**: High for Meshtastic and GPS modes (simpler setup, fewer dependencies). Zero impact for SDR-dependent modes.
- **Effort**: Medium — `navigator.serial` API is straightforward. The existing JS mode modules need a serial transport layer alongside the current SSE/fetch transport.
- **Chrome-only limitation**: This is acceptable because INTERCEPT is a local tool accessed via `localhost` or LAN. Firefox/Safari users would fall back to the existing backend path transparently.
- **Feature detection**: `if ('serial' in navigator)` gates the entire code path. If unsupported, the current backend-driven flow remains unchanged.

### Related Proposals
This pairs well with:
- **2.4 Plugin/Extension System** — Web Serial transport could be a plugin-provided capability.
- **4.1 MQTT Publisher Bridge** — GPS data from Web Serial could be forwarded to MQTT without touching the backend.
- **5.2 Mobile-First Layout** — Though Web Serial requires desktop Chrome, Meshtastic field ops on a laptop benefit from a mobile-friendly UI.

---

## Summary Table

| # | Feature | Category | Impact | Effort | Priority |
|---|---------|----------|--------|--------|----------|
| 1.1 | FT8 / FT4 / JT65 decoder | Signals | ★★★★★ | Medium | High |
| 1.3 | FM Broadcast Radio + RDS | Signals | ★★★★☆ | Low | High |
| 2.1 | Multi-user with RBAC | Platform | ★★★★★ | Medium | High |
| 2.6 | Prometheus metrics export | Platform | ★★★☆☆ | Low | High |
| 3.2 | Data retention policies | Data | ★★★★☆ | Low | High |
| 4.1 | MQTT publisher bridge | Integration | ★★★★★ | Low | High |
| 4.6 | Discord/Telegram/Slack alerts | Integration | ★★★★☆ | Low | High |
| 5.1 | Light theme toggle | UI | ★★★☆☆ | Low | High |
| 5.4 | Keyboard shortcuts expansion | UI | ★★★★☆ | Low | High |
| 2.2 | Scheduled scanning | Platform | ★★★★☆ | Medium | Medium |
| 2.3 | Multi-SDR orchestrator | Platform | ★★★★☆ | Medium | Medium |
| 2.5 | REST API + Swagger docs | Platform | ★★★★★ | Medium | Medium |
| 2.7 | IQ recording & playback (SigMF) | Platform | ★★★★☆ | Medium | Medium |
| 3.1 | DB backup & restore UI | Data | ★★★★☆ | Low | Medium |
| 3.3 | GeoJSON/KML export | Data | ★★★★☆ | Low | Medium |
| 4.2 | ADS-B Exchange / feeder relay | Integration | ★★★★☆ | Low | Medium |
| 4.5 | SondeHub radiosonde integration | Integration | ★★★☆☆ | Medium | Medium |
| 4.7 | InfluxDB time-series export | Integration | ★★★☆☆ | Medium | Medium |
| 5.2 | Mobile-first responsive layout | UI | ★★★☆☆ | Medium | Medium |
| 5.5 | Audio spectrum analyzer overlay | UI | ★★★★☆ | Medium | Medium |
| 7.3 | Traffic analysis & patterns | Analysis | ★★★★☆ | Medium | Medium |
| 7.4 | Predictive pass planning | Analysis | ★★★★☆ | Medium | Medium |
| 8.1 | RTL-SDR V4 native support | Hardware | ★★★★☆ | Low | Medium |
| 1.4 | NOAA Weather Radio + SAME | Signals | ★★★☆☆ | Medium | Low |
| 1.5 | IRIDIUM / L-band decoding | Signals | ★★★★☆ | High | Low |
| 2.4 | Plugin / extension system | Platform | ★★★★★ | High | Low |
| 2.8 | Waterfall recording & replay | Platform | ★★★☆☆ | Medium | Low |
| 3.4 | PDF report generation | Data | ★★★☆☆ | Medium | Low |
| 4.3 | APRS-IS internet gateway | Integration | ★★★☆☆ | Medium | Low |
| 4.4 | SatNOGS integration | Integration | ★★★☆☆ | Medium | Low |
| 5.3 | Dashboard layout builder | UI | ★★★☆☆ | Medium | Low |
| 6.1 | Signal database / wiki browser | Community | ★★★☆☆ | Medium | Low |
| 6.3 | Mode preset sharing | Community | ★★★☆☆ | Low | Low |
| 7.1 | ML signal classifier | Analysis | ★★★★☆ | High | Low |
| 7.2 | TDOA geolocation | Analysis | ★★★★☆ | Very High | Low |
| 8.2 | KrakenSDR support | Hardware | ★★★☆☆ | High | Low |
| 8.3 | PlutoSDR/USRP/BladeRF support | Hardware | ★★★☆☆ | Medium | Low |
| 9.0 | Web Serial API — browser-native device comms | Platform | ★★★★☆ | Medium | Medium |
