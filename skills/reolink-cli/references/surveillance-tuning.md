# Surveillance Tuning & Sensitivity Calibration

This reference guides AI agents in autonomously tuning camera detection settings (AI and motion) to eliminate false positives (noise) and fix missed detections.

## 1. Initial Interaction & Assessment
Tuning should be performed for **one camera at a time**.
Before changing any settings, ask the user to clarify the issue:
1. **Target Camera:** Which camera needs tuning?
2. **Issue Type:** Are we fixing false alarms (noise) or missed detections?
3. **Condition specifics:** What is causing the issue (e.g., IR flare from cobwebs/bugs at night, moving foliage/shadows in daylight, heavy rain, or missed vehicles)? 

*Note: If the user explicitly asks to `/goal` their way to the right settings, proceed directly to the diagnostic loop.*

## 2. Diagnostics (Read State)
Run these commands to baseline the current configuration:
```bash
# Check basic capability and stream info
reolink-cli --camera <name> info
reolink-cli --camera <name> device inventory --capabilities

# Read current sensitivities
reolink-cli --camera <name> detect ai get
reolink-cli --camera <name> detect motion get

# Check recent event history to quantify the noise
reolink-cli --camera <name> events query --since 24h
```

## 3. Tuning Strategies

### Strategy A: False Alarms (Daylight Shadows, Foliage, Rain, Traffic)
- **AI Detection:** Lower the sensitivity for the specific AI type causing the false alarms (e.g., `person` or `vehicle`), but try to preserve high thresholds where possible (e.g., keep Person > 70, Vehicle > 60) unless specifically requested.
- **Motion Detection (PIR/Pixel):** Often the root cause of generic "any motion" false alarms.
  - Lower the overall motion sensitivity: `reolink-cli --camera <name> detect motion apply --sensitivity 15`
  - Or disable raw motion entirely if AI is reliable: `reolink-cli --camera <name> detect motion apply --disable`

### Strategy B: False Alarms (Nighttime IR Flare, Cobwebs, Bugs)
- **Symptoms:** High frequency of "any motion" events at night, glowing orbs, or constant AI triggers due to reflected IR light.
- **Fix:** 
  - Substantially reduce `detect motion` sensitivity (e.g., to 10-20).
  - Apply AI object size limits (if supported) or carefully lower the AI `person`/`vehicle` sensitivity by 5-10 points.
  - Test disabling raw motion if only AI events are desired.

### Strategy C: Missed Detections
- **Symptoms:** Actual events (people/vehicles) are not triggering recordings.
- **Fix:**
  - Increase the relevant AI sensitivity: `reolink-cli --camera <name> detect ai apply --type person --sensitivity 85`
  - Increase global motion sensitivity.

## 4. Applying Changes Iteratively (The Goal Loop)
Use idempotent `apply` commands to make adjustments:
```bash
# Adjust AI sensitivity
reolink-cli --camera <name> detect ai apply --type person --sensitivity 75
reolink-cli --camera <name> detect ai apply --type vehicle --sensitivity 65

# Adjust raw motion sensitivity
reolink-cli --camera <name> detect motion apply --sensitivity 20
```

### Firmware Constraints (Schedules)
If you need to adjust recording schedules/timetables and the camera firmware rejects API-level recording schedule changes (e.g., battery models returning `400` or `405` for `record schedule set`), do **not** brute-force it. Instead:
1. Provide concise steps for the user to make the change in the official Reolink App UI (e.g., "Open app -> Device Settings -> Surveillance/Recording -> Schedule").
2. Wait for the user to confirm.
3. Verify the change by monitoring `events query` for new desired behaviors.

## 5. Verification
After applying a change, monitor the camera to confirm the issue is resolved.
- For historical verification: `reolink-cli --camera <name> events query --since 1h`
- For live monitoring during an active `/goal` loop: Use `events stream` or periodic `events query`.

Continue iterating (lowering or raising sensitivity by small increments of 5) until the optimal balance is found: zero false alarms and 100% capture of true events.
