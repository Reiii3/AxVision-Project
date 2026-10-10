# Change Log

Applies to Vision Daemon and Vision CLI.

## Added

- **CLI controls for features that could not be configured before:** Auto Preload, OS Performance, Display Enhancement, Temp Protect, DND, HyperOS Disable Thermal, XOS Bypass High Temp, Disable Thermal, and Smart Compact.
- **Per-game graphics driver and frame rate** can now be set from the CLI.
- **Reset Downscale:** a game's downscale can be restored to normal.
- **Help menu** (`--help`) that lists all available commands.
- **Smart Compact:** when the screen turns off, system memory is tidied up automatically.
- **Regular temperature checks during a game:** the thermal limit now follows the phone's real temperature instead of being locked to a single value for the whole session.
- **Your phone's original network mode is saved** before Network Adjustment turns on, and restored when the game ends.
- **Preload can run again after a restart,** and memory locked by a previous game is released first.

## Fixed

- **Thermal Protection:** it never activated before because the temperature was read in the wrong format. Now the phone automatically drops to Balance or Saver when it gets hot, then returns to the game profile once it has cooled down.
- **Thermal limit in the Performance, Balance, and Saver profiles:** it could previously hide the system's own heat restrictions. Now it follows the real temperature.
- **Disable Thermal:** it never turned on before. Now it works and can be switched back to normal.
- **CPU frequency settings:** all three profiles now use the correct CPU cluster. Before, they read a core that was not part of the big cluster.
- **Balance profile:** graphics now use a valid renderer.
- **Game Driver:** choosing a driver for one game used to change it for every app. Now it only affects the selected game. The ANGLE option is also saved correctly.
- **ADPF:** after a game ends, its settings return to the system default instead of being forced to another value.
- **Multitasking modes:** RAM settings now use the correct command. High mode returns memory handling to automatic.
- **Network Adjustment:**
  - Data Saver now actually turns on and off.
  - The network mode is restored to what it was, instead of being forced to a mode without 5G.
  - Fewer error messages in the log.
- **Background Reaper:** it no longer gets stuck forever if it was interrupted halfway.
- **Re-compile at 03:10:** now runs once a day. Before, it could repeat many times.
- **Preload:** it used to run only once per game, ever. Now it runs again after the phone restarts.
- **Downscale:** the scale and mode values are checked first to prevent wrong input, and the FPS you choose is actually used.
- **Battery Monitor:** no longer errors out when battery data cannot be read, and power from a dock is recognized.
- **Restoring after a game (OS Performance and XOS Bypass):** settings return to normal correctly and no longer disturb the daily profile.
- **Screen on/off detection:** no longer misreads on phones with more than one display.
- **Silent error when a game starts:** a feature that was called but did not exist no longer causes a hidden failure.

## Removed

- **Forced memory compaction on games:** it conflicted with Preload, which runs just before it.
- **Old freezer setting:** it is not recognized by Android, so it was replaced with the correct one.
- **Removal of graphics driver settings that overwrote the profile's choice.**
- **Restoring OS performance settings in a way that overwrote the daily profile.**

## Notes

- Not yet tested on a real device.
- Features that turn off temperature protection (Temp Protect, HyperOS Disable Thermal, Disable Thermal) can make the phone hotter. Use them with care.
- The loading boost value in the Downscale feature is very large and can make the phone hot.
- After a game, DNS returns to AdGuard instead of your original DNS.
- One function in the daemon (`cmd_game`) is still empty.
