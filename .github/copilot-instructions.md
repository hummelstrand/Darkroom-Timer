# Copilot / Agent instructions for Darkroom-Timer

Quick context
- This is an Arduino-based darkroom timer project. Main sketches live in the `darkroom_timer/` folder; the primary sketch is `darkroom_timer/darkroom_timer.ino` which composes behavior from several `.ino` files (e.g., `timers.ino`).
- A local Arduino library `TM1638plus` lives under `libraries/TM1638plus/` and provides the display/button interface used across sketches.

What an agent should know first (big picture)
- The repo contains multiple standalone Arduino sketches (each `.ino` is a separate build target). Work starts by editing the sketch you intend to run, typically `darkroom_timer/darkroom_timer.ino`.
- Hardware interface is through the `TM1638plus` library (display + buttons) and a simple relay/buzzer. See `libraries/TM1638plus/src/TM1638plus.*` for the API used (e.g., `displayText()`, `readButtons()`, `setLED()`).
- Platform branches: code uses conditional compilation for targets: `#if defined(__AVR__)` (Arduino), `#if defined(ESP8266)` (ESP8266). There is also a `high_freq` flag passed to `TM1638plus` for high-clock MCUs.

Typical developer workflows (how to build & run)
- Open the desired sketch folder in the Arduino IDE, or build from the CLI. There is no PlatformIO config here.
- Example `arduino-cli` build command (adjust FQBN to your board):

```
arduino-cli compile --fqbn arduino:avr:uno /path/to/repo/darkroom_timer/darkroom_timer.ino
arduino-cli upload -p /dev/ttyUSB0 --fqbn arduino:avr:uno /path/to/repo/darkroom_timer/darkroom_timer.ino
```

Key code patterns & conventions (project-specific)
- UI state machine: `uiMode` integer controls modes. Look for `uiModes()` in `darkroom_timer.ino` to understand state transitions.
- Timing: the project counts time in tenths of seconds (see `timerCountdown()` in `timers.ino`) and uses `millis()` loops rather than `delay()` (there is a custom `delayTimer()` helper).
- Persistence: settings are stored in EEPROM with specific addresses defined near the top of `darkroom_timer.ino` (e.g., `eeBrightness`, `eeIncrement`). Writes are batched after timer runs to reduce flash wear—preserve that behavior when changing persistence.
- Display/buttons: use `tm` (the `TM1638plus` instance). Common calls: `tm.displayText(...)`, `tm.readButtons()`, `tm.setLED(...)`. Button constants are defined near the top of the main sketch.
- Globals: the code uses many globals for state (FStop, timerInc, stepIdx, etc.). When refactoring, keep global usage consistent across `.ino` files (they are linked together by the Arduino build process).

Integration points & dependencies
- Local library: `libraries/TM1638plus/` — changes here affect all sketches using the display/button interface.
- Hardware expectations: 7-seg/TM1638 display and relay/buzzer; pins are mapped at the top of `darkroom_timer/darkroom_timer.ino` (STROBE_TM, CLOCK_TM, DIO_TM, RELAY_PIN, TONE_PIN).
- No external CI/tests: there are no automated unit tests; `strip_test_generator/` contains an HTML tool for strip test visuals but not test runners.

Safe change guidelines for agents
- Prefer small, local edits (fix a single function or constant) and preserve existing EEPROM write semantics.
- When modifying pin mappings or timing, update the top-of-file defines and test on hardware (or simulate carefully). Document hardware-dependent changes in the `README.md`.
- If altering the UI state machine (`uiMode`), keep mode numeric values and switch handling consistent across files; search for `uiMode` to find all affected logic.

Files to open first (examples)
- Main sketch: `darkroom_timer/darkroom_timer.ino`
- Timer logic: `darkroom_timer/timers.ino`
- Library API: `libraries/TM1638plus/src/TM1638plus.h` and `TM1638plus.cpp`
- Project README: `README.md`

If something is unclear
- Ask for board model & upload method (Arduino IDE vs `arduino-cli`) before changing build/upload instructions.
- Ask for a hardware test or a small photo/video when proposing changes that affect timings, brightness, or relay control.

End.
