# Daikin ARC480A53 remote-control feature specification

Target component: `daikin_arc` (ESPHome)
Target remote: Daikin ARC480A53
Status: capture-backed specification; feature implementation is not yet included.

## Requested controls

### Fan speed
Expose all seven settings:
- Silent
- Speed 1
- Speed 2
- Speed 3
- Speed 4
- Speed 5
- Auto

### ECONO and POWERFUL
Expose both as Home Assistant controls. The remote does not allow ECONO and POWERFUL to be active at the same time. Enabling one must disable the other. Preserve the current climate mode, target temperature, fan setting, and swing settings when changing either option.

### ON and OFF timers
Expose separate ON and OFF timer controls, plus CANCEL. The captured examples cover 1–4 hours only. A 12-hour maximum is not confirmed by captures and must not be treated as verified. Keep duration values as explicit lookup data until the timer encoding is understood.

CANCEL clears the active timer state. The captured CANCEL frame differs from the baseline frame in the option byte; duration retention needs a physical test.

## Captured AEHA frames

All values below are complete 17-byte AEHA sequences supplied for the ARC480A53 remote. Offsets refer only to these 17-byte sequences. They are not offsets into the component's existing `remote_state[]` buffer.

### Fan setting (Heat, 21 °C)

| Setting | AEHA frame |
|---|---|
| Silent | `E400008254000D000000000000A300105F` |
| 1 | `E400008254000C000000000000A300105E` |
| 2 | `E4000082540002000000000000A3001051` |
| 3 | `E400008254000A000000000000A3001059` |
| 4 | `E4000082540006000000000000A3001055` |
| 5 | `E400008254000E000000000000A300105D` |
| Auto | `E4000082540005000000000000A3001057` |

The fan setting changes byte 6 in these captures. The observed values are non-linear; use explicit values rather than deriving them from a numeric speed.

### ECONO and POWERFUL (Heat, 21 °C, Auto)

| State | AEHA frame |
|---|---|
| Baseline | `E4000082540005000000000000A3001057` |
| POWERFUL on | `E4000082540005000000008000A30010D7` |
| POWERFUL off | `E4000082540005000000000000A3001057` |
| ECONO on | `E4000082540005000000000000A3201077` |
| ECONO off | `E4000082540005000000000000A3001057` |

In these captures POWERFUL changes byte 11 from `00` to `80`; ECONO changes byte 14 from `00` to `20`. These flags must be mapped to the actual transmit buffer before implementation. The captures show that the two options are mutually exclusive.

### Timer frames (Heat, 21 °C, Auto)

Baseline:
`E4000082540005000000000000A3001057`

| Timer | Duration | AEHA frame |
|---|---:|---|
| ON | 1 h | `E40000C2340005003C00000000A3001054` |
| ON | 2 h | `E40000C2340005001E00000000A3001066` |
| ON | 3 h | `E40000C2740005002D00000000A3001025` |
| ON | 4 h | `E40000C2340005000F00000000A300107B` |
| OFF | 1 h | `E40000A2340005000003C00000A30010CD` |
| OFF | 2 h | `E40000A2340005000001E00000A30010EE` |
| OFF | 3 h | `E40000A2740005000002D00000A30010BC` |
| OFF | 4 h | `E40000A2340005000000F00000A30010FF` |
| CANCEL | — | `E4000082340005000000000000A308103F` |

Timer ON captures use byte 3 value `C2`; timer OFF captures use `A2`; CANCEL uses `82`. Timer data appears in different byte positions for ON and OFF. The duration values do not support a verified simple linear formula. The 3-hour captures also differ in byte 4 from the 1-, 2-, and 4-hour captures, so further same-state captures are required before assigning independent fields.

Do not copy these byte offsets directly into `remote_state[]`. The supplied sequences are 17-byte AEHA frames, while the current component constructs a separate header and 19-byte state payload.

## Implementation constraints

- Preserve the existing working transmit path and checksum logic.
- Map captured AEHA fields to the component's existing frame layout before changing C++ encoding.
- Do not infer timer formulas from the current 1–4 hour samples.
- Use explicit fan and timer lookup tables where the captured values are known.
- Keep ECONO and POWERFUL mutually exclusive in both the API and transmitted state.
- Do not claim protocol support for unverified timer durations.

## Validation still required

1. Establish the exact relationship between the captured 17-byte AEHA sequence and the component's 20-byte header plus 19-byte state payload.
2. Capture more timer durations with identical climate settings, including the maximum duration supported by the remote.
3. Verify CANCEL behavior and whether it retains the previously selected duration.
4. Compile with ESPHome and test each exposed control against the physical air conditioner.
