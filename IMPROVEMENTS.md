# INTERCEPT — Code Review: Improvement Proposals

## Security — Critical

1. **Default `admin:admin` credentials** — `config.py:504-505`. Force password change on first login or generate a random one at first boot.

2. **CSRF disabled globally for all blueprints** — `routes/__init__.py:96-98`. Every POST endpoint (`/killall`, exports, mode start/stop) is vulnerable to CSRF. Only SSE endpoints should be exempted.

3. **Missing Content-Security-Policy header** — No CSP headers anywhere. With 192+ `innerHTML` usages and inconsistent escaping across the frontend, any XSS bug becomes trivially exploitable.

4. **XSS vulnerabilities in `wefax.js`** — Multiple lines (617-627, 688-700, 769-771) use `innerHTML` with unescaped user/decoder data (`station`, `content`, `filename`, `img.url`). The worst: raw string interpolation inside `onclick` handlers.

5. **SSRF in `controller.py:127`** — Agent URL validation accepts `localhost`, `127.0.0.1`, `[::1]`, and encoded internal IPs (e.g., `0x7f000001`). Must block private/internal IP ranges.

6. **Subprocess parameter injection vector** — `acars.py:316`, `vdl2.py:280`, `rtlamr.py:184`. User-supplied frequency values and tool parameters are appended directly to subprocess command lists without validating they are proper numeric values.

7. **Missing HSTS header** — When HTTPS is enabled, `Strict-Transport-Security` is never set. Users can be downgraded to HTTP via MITM, leaking session cookies.

8. **Self-signed certificate with no user verification** — `_ensure_self_signed_cert()` generates a cert with no SANs and no mechanism for the user to verify the fingerprint on first connect. Trains users to click through browser warnings.

9. **Information disclosure at `/devices/debug`** — `app.py:564`. Exposes loaded kernel modules, USB device list, rtl_test output, SoapySDRUtil output. Only gated by `require_login()`.

10. **`kill_all` endpoint: POST without additional auth confirmation** — If `INTERCEPT_DISABLE_AUTH=true` is set, anyone can POST to `/killall` and terminate all receiver processes. No rate limiting or confirmation token.

11. **Default bind to all interfaces** — `config.py:407`: `HOST = _get_env('HOST', '0.0.0.0')`. Default is to listen on all network interfaces, potentially exposing the dashboard to the local network without explicit consent.

---

## Architecture & Maintainability — High

12. **`app.py` (1,391 lines) is a God Module** — Mixes app factory, global state, initialization, health checks, SSL, exports, WebSockets, and cleanup. Should be decomposed into separate modules.

13. **25+ copies of identical `x_process/queue/lock` boilerplate** — `app.py:223-318`. Each mode repeats `_process = None; _queue = queue.Queue(); _lock = threading.Lock()`. Could be abstracted into a `SignalPipeline` dataclass or similar.

14. **19+ places to edit when adding a new mode** — `index.html` requires changes in 19+ spots: `modeCatalog`, `moduleDestroyMap`, HTML partials, CSS, JS, visuals containers, nav buttons, agent modes, hide arrays, and the massive `switchMode()` if/else chain. There is no data-driven mode registration system.

15. **`escapeHtml` defined 16 separate times** — 16 JS files redefine the same function. A global version already exists in `core/utils.js` but is not used consistently.

16. **Side effects at module import time** — `app.py:1297` calls `_init_app()` at module level. Importing `app` (even for testing) triggers database initialization, blueprint registration, WebSocket setup, and a background `threading.Thread`. `utils/process.py:143-149` registers signal handlers at import time.

17. **Conflicting configuration sources** — Some values appear in three places: `config.py`, `utils/constants.py`, and `gunicorn.conf.py`. For example, `ADSB_SBS_PORT = 30003` in both `config.py` and `constants.py`. Prone to drift.

---

## Error Handling & Stability — High

18. **Widespread silent exception swallowing** — 15+ locations with `except Exception: pass` or `contextlib.suppress(Exception)` and no logging. Real failures are invisible: WebSocket init at `app.py:1190-1223`, `claim_sdr_device` at `app.py:403`, health checks, etc.

19. **No global 500 error handler** — Only an `@app.errorhandler(429)` exists. Any unhandled exception returns a generic 500 with no centralized logging.

20. **Silent data loss on queue overflow** — All queues use `put_nowait()`. When they hit the max (1000), data is silently dropped with no metrics or alerts.

21. **Race condition in singleton factories** — `tscm/correlation.py:1162` and `tscm/device_identity.py:1139`: `if _instance is None: _instance = Class()` without a lock. Multiple instances can be created under concurrency.

22. **`_device_history` dict mutated without any lock** — `tscm/detector.py:53`. Module-level dict modified by multiple threads simultaneously. Guaranteed data corruption on concurrent TSCM sweeps.

23. **Subprocess leaks** — `wifi/scanner.py:798`: airodump-ng not registered for atexit cleanup, becomes orphaned on crash. `bluetooth/fallback_scanner.py:273`: hcidump subprocess lacks a `finally` block, may leak on exception.

24. **`cleanup_stale_processes()` uses `pkill -9`** — `process.py:152`. Kills **all** system instances of rtl_433, rtl_fm, etc. Should only kill PIDs registered in `_spawned_processes`.

25. **`sse_stream()` vs `sse_stream_fanout()` inconsistency** — `weather_sat.py:589` and `subghz.py:425` use `sse_stream()` instead of fanout. Potentially incorrect behavior with multiple SSE clients.

26. **`audio_websocket.py:111`** — Parameters (`freq`, `mod`, `squelch`, `gain`) received over WebSocket are used in subprocess commands with **no validation at all**.

27. **Database connections per-greenlet are never closed** — `database.py:42`. Each greenlet holds an open SQLite connection. Without `teardown_appcontext`, risk of file descriptor exhaustion under load.

28. **Deferred init runs on daemon thread** — `app.py:1293`. Under Flask dev server, `KeyboardInterrupt` kills daemon threads mid-operation without cleanup.

29. **TOCTOU concerns in process health reading** — `app.py:933-955`. Between the `is not None` check and `.poll()`, a different thread could set the process to `None` or terminate it, producing transient false negatives.

---

## Duplicated Code — Medium

30. **SSE response boilerplate duplicated 20+ times** — The same `Response(sse_stream_fanout(...)), mimetype='text/event-stream', headers...` pattern in 20+ route files. A `build_sse_response()` utility would eliminate hundreds of lines.

31. **Start/stop lifecycle duplicated 10+ times** — `acars.py:212-389` and `vdl2.py:198-353` are ~90% identical. Other modes (ais, pager, sensor, ook) share ~70% structure. A `DecoderServiceBase` class would eliminate ~500 lines.

32. **Inline `clear_queue()` loop duplicated 20+ times** — Already exists as `utils/sse.py:clear_queue()` but is never used in route files.

33. **Duplicate table creation in SQL** — `database.py:205-249`. Tables `alert_rules`, `alert_events`, `recording_sessions` are created twice.

34. **Colliding function names between `utils/validation.py` and `utils/sdr/validation.py`** — `validate_frequency`, `validate_gain`, `validate_ppm`, `validate_device_index` have completely different signatures. Confusing import shadowing.

35. **RSSI statistics implemented 3 times** — `bluetooth/aggregator.py`, `wifi/scanner.py`, `tscm/correlation.py`.

36. **Baseline tracking implemented 3 times** — Bluetooth, WiFi, and TSCM each have their own baseline systems with different data models and APIs.

37. **SDR CommandBuilders share ~80% boilerplate** — HackRF, Airspy, LimeSDR, SDRPlay share identical patterns for `build_adsb_command`. A `SoapySDRCommandBuilder` base class would reduce duplication.

38. **MAC vendor lookup duplicated across modules** — `wifi/constants.py`, `tscm/correlation.py`, and `wifi/deauth_detector.py` each implement their own OUI-to-vendor resolution.

39. **Risk scoring logic duplicated** — `tscm/correlation.py`. Identical threshold logic (`>= 6` = HIGH, `>= 3` = NEEDS_REVIEW) appears in both `_recalculate_score` and `apply_score_modifier`.

---

## Testing & CI/CD — Medium

40. **`continue-on-error: true` in CI** — `ci.yml:26`. Test failures do **not** block PRs.

41. **14 route modules with no dedicated test file** — alerts, audio_websocket, bluetooth_v2, wifi_v2, ground_station, meteor_websocket, recordings, rtlamr, settings, space_weather, spy_stations, sstv_general, updater, vdl2.

42. **14 utility modules with no test coverage** — aircraft_db, event_pipeline, logging, process_monitor, recording, responses, rotator, satellite_predict, satellite_telemetry, satnogs, sigmf, temporal_patterns, updater, airline_codes.

43. **No coverage threshold** — CI never runs `pytest --cov` or enforces a minimum coverage.

44. **No matrix testing** — Only Python 3.11 is tested, despite `pyproject.toml` claiming support for 3.9, 3.10, and 3.12.

45. **No mypy type checking** in CI or pre-commit — Only ruff lint runs.

46. **`smoke_test_bluetooth.py` not integrated** with pytest or CI. Requires a running server to work.

47. **Thin stub tests** — 13 test files under 50 lines with only 1-2 assertions. Marginal value.

48. **SSE streaming endpoints not tested beyond content-type checks** — No test verifies events are actually emitted.

---

## Docker & Dependencies — Medium

49. **Container runs as root** — No `USER` directive, no `COPY --chown`. No non-root user created in the runtime stage.

50. **`privileged: true` in docker-compose** — `docker-compose.yml:24`. Use `--device=/dev/bus/usb` or `--cap-add=SYS_RAWIO` instead.

51. **Hardcoded Postgres password** — `docker-compose.yml:136`: `POSTGRES_PASSWORD: intercept`. Should use Docker secrets or an env file.

52. **`pyproject.toml` out of sync with `requirements.txt`** — Missing: flask-compress, gevent, gunicorn, numpy, scipy, Pillow, meshtastic, psycopg2-binary, scapy, and more. `pip install .` from pyproject.toml would fail at runtime.

53. **No lockfile** — No `requirements.lock`, `poetry.lock`, or `uv.lock`. Reproducible builds are impossible with `>=` version pins.

54. **Broken `pyproject.toml` entrypoint** — Defines `intercept = "intercept:main"` but `intercept.py` has no `main()` function.

55. **docker-compose.yml service duplication** — `intercept` and `intercept-history` are ~80% identical. YAML anchors would eliminate 70+ duplicate lines.

56. **No logging rotation on docker-compose** — Missing `max-size`/`max-file`. Container logs grow unbounded.

57. **No resource limits** on docker-compose services — Missing `mem_limit`, `cpus`, etc.

58. **Shallow clone missing on some git pulls in Dockerfile** — AIS-catcher, rx_tools, and SatDump are cloned without `--depth 1`.

59. **No image vulnerability scanning** — No Trivy, Snyk, or Docker Scout in CI or build pipeline.

60. **`build-multiarch.sh` lacks cache flags** — No `--cache-from`/`--cache-to` passed to `docker buildx build`. Every build compiles all tools from source.

---

## Frontend — Medium-Low

61. **Inconsistent error handling in JS** — 5 different patterns: `reportActionableError()`, `showError()`, `showNotification()`, `alert()`, `console.error()`. Many `fetch()` calls have no `.catch()` at all.

62. **Monolithic 16,639-line `index.html`** — All 29 mode partials are included inline. The browser must parse DOM for all modes even when unused.

63. **8 modes have HTML partials but no dedicated CSS** — bluetooth, wifi, websdr, pager, sensor, ais, rtlamr, satellite. Styles are missing or inlined.

64. **No CSS/JS bundling** — 20+ separate `<link>` tags for CSS alone. No minification pipeline.

65. **Hardcoded colors in JS** — `weather-satellite.js` uses `#0d1117` instead of CSS variable `var(--bg-primary)`.

66. **Modes with HTML partials but no dedicated JS module** — pager, sensor, rtlamr, acars, ais, satellite, tscm, vdl2. Their logic is embedded in the monolithic `index.html`.

67. **External URLs hardcoded** — `gps.js` pins `globe.gl@2.33.1`. `index.html` has preconnect hints to unpkg.com, cdn.jsdelivr.net, fonts.googleapis.com.

---

## Code Quality — Low

68. **`__import__("datetime")` anti-pattern** — `app.py:715`. Use proper top-level imports.

69. **CDN settings overwritten after loading** — `app.py:197-205`. Reads `offline.assets_source` from the database, then unconditionally overwrites both `assets_source` and `fonts_source` to `"local"`.

70. **Inconsistent lock and process naming** — Pager uses `process_lock` (generic) instead of `pager_lock`. Pager process is `current_process` which is dangerously ambiguous.

71. **Overly permissive hostname validation regex** — `validation.py:77`. Allows consecutive dots, trailing dots, and single-character hostnames.

72. **`sanitize_callsign` missing `escape_html`** — `validation.py:181`. Unlike `sanitize_ssid` and `sanitize_device_name`, callsigns are not HTML-escaped.

73. **SSE fanout busy-wait infinite loop** — `sse.py:34`. When `channel.source_queue is None`, the fanout thread spins in a `time.sleep(0.5)` loop forever with no shutdown flag.

74. **`init_db()` is 675 lines** — `database.py:109-783`. Should be split into smaller functions: `_init_core_tables()`, `_init_tscm_tables()`, `_seed_admin_user()`, `_run_migrations()`, etc.

75. **Unused `clear_queue()` utility** — 20+ route files implement their own queue draining loop instead of calling `utils.sse.clear_queue()`.

76. **Inconsistent URL/route naming** — Some blueprints use `url_prefix` (e.g., `/acars/start`), others hardcode paths (e.g., `/start_pager`). No convention.

77. **Inconsistent keepalive interval** — `bluetooth.py:570-571` hardcodes `timeout=1.0, keepalive_interval=30.0` directly, bypassing the shared constants.

78. **Jinja `| safe` on device data** — `index.html:3872`: `{{ devices | tojson | safe }}`. Disables all auto-escaping. A device name containing `</script>` would break the entire page.
