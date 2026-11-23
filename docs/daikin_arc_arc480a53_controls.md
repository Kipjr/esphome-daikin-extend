# Daikin ARC480A53 remote-control feature specification

Target component: `daikin_arc` (ESPHome)
Target remote: Daikin ARC480A53
Status: implementation proposal; timer frame encoding is provisional and requires validation with captured IR frames.

## Requested controls

### Fan speed
Expose the following fan settings:
- Silent
- Speed 1
- Speed 2
- Speed 3
- Speed 4
- Speed 5
- Auto

The existing protocol constants in `daikin_arc.h` identify fan nibble values: silent `0xB0`, speeds 1–5 `0x30` through `0x70`, and auto `0xA0`. The current component maps fan selections to a reduced set of Home Assistant fan modes and does not expose each speed.

### ECONO
Provide an economy-mode control. Enabling or disabling ECONO should preserve the current operating mode, target temperature, fan setting, and swing settings. The control should modify only the economy flag in the transmitted state frame.

### POWERFUL
Provide a Powerful-mode control. When enabled, send a frame that requests maximum output for the current main operating mode (Cool, Heat, Dry, Auto, etc.), overriding other output-related settings as required by the remote. When disabled, return to the normal climate configuration.

### ON timer
- Press ON to enable automatic start using the last configured duration.
- Each additional ON press increments the duration by one hour.
- Limit duration to 12 hours.
- Retain the duration after cancelling the timer.

### OFF timer
- Press OFF to enable automatic shutdown using the last configured duration.
- Each additional OFF press increments the duration by one hour.
- Limit duration to 12 hours.
- Retain the duration after cancelling the timer.

### CANCEL
Cancel any active ON or OFF timer. Do not reset the remembered duration.

## Frame-field notes (provisional)
The component's existing comments identify timer enable flags in byte 5 and timer data in bytes 11–12, and Powerful-related data near byte 13. The ECONO flag location has been suggested as byte 16, bit 2. These field locations and encodings are not validated for this remote and must be checked against IR captures before treating them as authoritative.

Timer duration is expected to be represented in minutes, with 12 hours corresponding to 720 minutes. This value and the encoding/byte ordering remain to be validated.

## Validation plan
1. Capture IR frames for each fan setting and compare byte 8's upper nibble.
2. Capture otherwise identical frames with ECONO off/on to isolate the ECONO bit.
3. Capture otherwise identical frames with POWERFUL off/on to isolate its flag and related changes.
4. Capture ON timer values at 1, 2, and 12 hours; verify enable flag, byte layout, units, and byte order.
5. Capture OFF timer values at 1, 2, and 12 hours; verify enable flag, byte layout, units, and byte order.
6. Capture CANCEL after each timer is set; confirm flags clear while duration bytes remain.
7. Compile and test with ESPHome; test transmission against the physical unit.

## Scope of this change
This document records the desired functionality and the protocol areas requiring validation. It does not claim that the controls are implemented or that timer frame fields are verified.
